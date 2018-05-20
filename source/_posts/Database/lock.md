---

title: 数据库 - 并发一致性问题

---

### 丢失修改

数据的修改会存在覆盖，例如，T1 和 T2 两个事务对数据 t 先后进行修改：

![ock (2](/Users/H/Downloads/Lock (2).png)

可以看出事务2的修改并没有起作用，而是被事务2的修改覆盖，也就是事务1丢失了修改

解决方案：**读未提交**

### 脏读

读取即将回滚的事务的数据，与预期的结果不一样，例如，事务1修改数据 t，但未提交，而事务2读取数据 t，此时事务1进行回滚操作，事务1读取数据 t

![ock (3](/Users/H/Downloads/Lock (3).png)

两个事务最后的读取结果不同，因为事务1在进行提交或回滚操作前，事务2提前读取数据 t，导致事务2读取的就是脏数据，也就是脏读

解决方案：**读已提交**

### 不可重复读

在另一个事务的修改操作前后读取数据，结果不一样，例如，事务2查询数据 t，然后事务1修改数据 t后，事务2再次读取数据 t，可以发现事务1两次读取的结果不一样

![ock (4](/Users/H/Downloads/Lock (4).png)

解决方案：**重复读**

### 幻影读

在一定范围内，一个事务的插入数据操作前后，另一个事务读取该范围的数据，结果不一样，例如：事务1查询范围内的数据，然后事务2在此范围内插入一条数据，事务1再次查询该范围内的数据，可以发现事务1两次查询的数据不一样

![ock (5](/Users/H/Downloads/Lock (5).png)

解决方案：**可串行化**

###四种隔离级别对比

| 隔离级别 | 脏读 | 不可重复读 | 幻影读 |
| :------: | :--: | :--------: | :----: |
| 未提交读 |  Y   |     Y      |   Y    |
|  提交读  |  N   |     Y      |   Y    |
| 可重复读 |  N   |     N      |   Y    |
| 可串行化 |  N   |     N      |   N    |

## 锁

### 锁粒度

![ock2 (1](/Users/H/Downloads/Lock2 (1).png)

对需要进行修改的数据上锁，而不是全部的数据，锁定的数据量越少，发生锁争用的可能就越小，系统的并发程度就越高

但是锁的各种操作会消耗资源，包括获取锁，检查锁是否已经解除，释放锁，都会增加系统开销。因此锁粒度越小，系统开销就越大

MySQL 中提供了两种锁粒度：**行级锁**以及**表级锁**

### 锁类型

####排它锁（Exclusive）

简称 X 锁，也称写锁

####共享锁（Shared）

简称 S 锁，也称读锁

规定：

- 若一个事务对数据 t 加了 X 锁，就可以对数据 t 进行读取和修改，在解锁前，其他事务不能对 数据 t 加任何锁
- 若一个事务对数据 t 加了 S 锁，就可以对数据 t 进行读取，但是不能修改，在解锁前，其他事务只能对数据 t 加 S 锁读取数据，不能加 X 锁

两个锁的兼容性：

| Lock Type |  X   |  S   |
| :-------: | :--: | :--: |
|     X     |  N   |  N   |
|     S     |  N   |  Y   |

####意向锁（Intention）

支持多粒度锁，本身就是一个表锁，通过在原来的 X / S 锁之上引入了 IX / IS（SIX），表示一个事务想要在某个数据行上加 X 锁或 S 锁

在锁定该表前，不需要检查表级锁或行级锁，只需要检查表上的意向锁

**IS锁**

如果对一个数据对象加 IS 锁，表示它的后裔结点拟（意向）加 S 锁。例如，要对某个数据对象加 S 锁，则需要先加 IS 锁

**IX锁**

如果对一个数据对象加 IX 锁，表示它的后裔结点拟（意向）加 X 锁。例如，要对某个数据对象加 X 锁，则需要先加 IX 锁

各种锁的兼容性：

| Lock Type |  X   |  IX  |  S   |  IS  |
| :-------: | :--: | :--: | :--: | :--: |
|     X     |  N   |  N   |  N   |  N   |
|    IX     |  N   |  Y   |  N   |  Y   |
|     S     |  N   |  N   |  Y   |  Y   |
|    IS     |  N   |  Y   |  Y   |  Y   |

### 锁协议

####三级锁协议

**一级锁协议**

事务 T1 要修改数据 T 时必须加 X 锁，直到 T1 结束才释放锁。

可以解决丢失修改问题，因为不能同时有两个事务对同一个数据进行修改，那么一个事务的修改就不会被覆盖

![ock3 (1](/Users/H/Downloads/Lock3 (1).png)

**二级锁协议**

在一级的基础上，要求读取数据 T 时必须加 S 锁，读取完马上释放 S 锁。

可以解决读脏数据问题，因为如果一个事务在对数据 T 进行修改，根据一级锁协议，会加 X 锁，那么就不能再加 S 锁了，也就是不会读入数据

![ock3 (2](/Users/H/Downloads/Lock3 (2).png)

**三级锁协议**

在二级的基础上，要求读取数据 T 时必须加 S 锁，直到事务结束了才能释放 S 锁。

可以解决不可重复读的问题，因为一个事务读取 T 时，其它事务不能对 T 加 X 锁，从而避免了在读的期间数据发生改变

![ock3 (3](/Users/H/Downloads/Lock3 (3).png)

####两段锁协议