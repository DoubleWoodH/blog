---

title: GoPackages - database/sql 源码解析(1)

---

###简单例子

```go
package main

import (
    "database/sql"
    "fmt"
    "log"

    _ "github.com/go-sql-driver/mysql"
)

func main() {
    db, err := sql.Open("mysql", "user:password@/dbName")
    if err != nil {
        panic(err)
    }

    defer db.Close()

    if err = db.Ping(); err != nil {
        log.Fatal(err)
    }

    rows, err := db.Query("SELECT name FROM tableName")
    if err != nil {
        log.Fatal("Query error: ", err)
    }

    for rows.Next() {
        var name string
        if err := rows.Scan(&name); err != nil {
            log.Fatal("Scan error: ", err)
        }
        fmt.Println(name)
    }
    if err := rows.Err(); err != nil {
        log.Fatal(err)
    }
}
```

####注册 mysql 驱动

```go
var (
    driversMu sync.RWMutex
    drivers   = make(map[string]driver.Driver)
)

func Register(name string, driver driver.Driver) {
    // 驱动锁
    driversMu.Lock()
    defer driversMu.Unlock()

    if driver == nil {
        panic("sql: Register driver is nil")
    }

    // 判断是否注册过
    if _, dup := drivers[name]; dup {
        panic("sql: Register called twice for driver " + name)
    }

    drivers[name] = driver
}

// github.com/go-sql-driver/mysql/driver
func init() {
    sql.Register("mysql", &MySQLDriver{})
}
```

####DB

```go
type DB struct {
    driver       driver.Driver
    dsn          string                       // dataSourceName
    numClosed    uint64                       // 原子计数器，已关闭连接的数量
    mu           sync.Mutex
    freeConn     []*driverConn                // 当前连接池中空余的连接
    connRequests map[uint64]chan connRequest  // 连接请求
    nextRequest  uint64
    numOpen      int                          // 打开或将要打开的连接的数量
    openerCh     chan struct{}
    closed       bool
    dep          map[finalCloser]depSet       // 记录 db 与 conn 之间的依赖关系，维持连接池以及关闭时使用
    lastPut      map[*driverConn]string       // 只用于调试
    maxIdle      int                          // = 0 则为 defaultMaxIdleConns
    maxOpen      int                          // <= 0 表示无限
    maxLifetime  time.Duration                // 一个连接可能被重用的最长时间
    cleanerCh    chan struct{}
}
```

####Open：创建数据库实例

```go
func Open(driverName, dataSourceName string) (*DB, error) {
    driversMu.RLock()
    // 通过 driverName 获取驱动程序实例
    driveri, ok := drivers[driverName]
    driversMu.RUnlock()

    if !ok {
        return nil, fmt.Errorf("sql: unknown driver %q (forgotten import?)", driverName)
    }

    // 封装成一个db实例返回，但是此时并没有一个连接存在
    db := &DB{
        driver:       driveri,
        dsn:          dataSourceName,
        openerCh:     make(chan struct{}, connectionRequestQueueSize),
        lastPut:      make(map[*driverConn]string),
        connRequests: make(map[uint64]chan connRequest),
    }

    // 而是开启了一个 goroutine 来生产连接
    go db.connectionOpener()

    return db, nil
}
```

**生产连接：**

```go
func (db *DB) connectionOpener() {
    // channel 的缓冲区大小为 1000000
    // goroutine 会一直阻塞，直至需要连接时，往 openerCh 发送数据即可
    for range db.openerCh {
        db.openNewConnection()
    }
}

func (db *DB) openNewConnection() {
    // database/sql 第一次调用驱动的接口，获取一个真实的连接
    ci, err := db.driver.Open(db.dsn)

    db.mu.Lock()
    defer db.mu.Unlock()


    if db.closed {
        if err == nil {
            // 关闭连接
            ci.Close()
        }
        db.numOpen--
        return
    }

    if err != nil {
        db.numOpen--
        db.putConnDBLocked(nil, err)
        db.maybeOpenNewConnections()    // 产生新连接
        return
    }

    // 将从驱动获取的连接封装成 driverConn
    dc := &driverConn{
        db:        db,
        createdAt: nowFunc(),
        ci:        ci,
    }

    // 将新生成的 driverConn 放入空闲数组中
    if db.putConnDBLocked(dc, err) {
        db.addDepLocked(dc, dc)
    } else {
        db.numOpen--
        ci.Close()
    }
}
```

**DB.maybeOpenNewConnections：产生新连接**

```go
func (db *DB) maybeOpenNewConnections() {
    // 获取请求数量
    numRequests := len(db.connRequests)

    if db.maxOpen > 0 {
        numCanOpen := db.maxOpen - db.numOpen
        if numRequests > numCanOpen {
            numRequests = numCanOpen
        }
    }

    // 当请求数量大于 0 时，打开的连接数加一，
    for numRequests > 0 {
        db.numOpen++
        numRequests--
        if db.closed {
            return
        }

        // 上面的 goroutine 将会执行
        db.openerCh <- struct{}{}
    }
}
```

因此，在获取一个 db 实例的时候，同时也生成了一个生产连接的 goroutine，这个 goroutine 阻塞在 channel 中，当需要连接的时候，直接向这个 channel 发送数据即可，产生连接有以下两种情况：

1. 第一次调用 **Ping** 函数的时候，会产生一个连接
2. 当调用 **DB.Exec** 或者 **DB.Query** 等方法时，如果空闲数组中有连接，则直接获取，如果空闲数组中没有可用的连接，则会产生一个新的连接

####DB.Ping：判断数据库实例是否可用

```go
func (db *DB) Ping() error {
	return db.PingContext(context.Background())
}

func (db *DB) PingContext(ctx context.Context) error {
    var dc *driverConn
    var err error

    for i := 0; i < maxBadConnRetries; i++ {
        // 从空闲数组获取或重新生成一个连接
        dc, err = db.conn(ctx, cachedOrNewConn)
        if err != driver.ErrBadConn {
            break
        }
    }

    if err == driver.ErrBadConn {
        // 总是重新生成一个连接
        dc, err = db.conn(ctx, alwaysNewConn)
    }
    if err != nil {
        return err
    }

    return db.pingDC(ctx, dc, dc.releaseConn)
}

func (dc *driverConn) releaseConn(err error) {
	dc.db.putConn(dc, err)
}

func (db *DB) putConn(dc *driverConn, err error) {
    db.mu.Lock()
    if !dc.inUse {
        if debugGetPut {
            fmt.Printf("putConn(%v) DUPLICATE was: %s\n\nPREVIOUS was: %s", dc, stack(), db.lastPut[dc])
        }
        panic("sql: connection returned that was never out")
    }
    if debugGetPut {
        db.lastPut[dc] = stack()
    }
    dc.inUse = false

    for _, fn := range dc.onPut {
        fn()
    }
    dc.onPut = nil

    if err == driver.ErrBadConn {
        db.maybeOpenNewConnections()
        db.mu.Unlock()
        dc.Close()
        return
    }
    if putConnHook != nil {
        putConnHook(db, dc)
    }
    added := db.putConnDBLocked(dc, nil)
    db.mu.Unlock()

    if !added {
        dc.Close()
    }
}
```

####DB.Close：关闭数据库

```go
func (db *DB) Close() error {
    db.mu.Lock()

    if db.closed {
        db.mu.Unlock()
        return nil
    }

    close(db.openerCh)

    if db.cleanerCh != nil {
        close(db.cleanerCh)
    }

    var err error
    fns := make([]func() error, 0, len(db.freeConn))
    for _, dc := range db.freeConn {
        fns = append(fns, dc.closeDBLocked())
    }

    db.freeConn = nil
    db.closed = true
    for _, req := range db.connRequests {
        close(req)
    }

    db.mu.Unlock()
    for _, fn := range fns {
        err1 := fn()
        if err1 != nil {
            err = err1
        }
    }

    return err
}
```

