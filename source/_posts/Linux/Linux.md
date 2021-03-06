---
title: Linux探索之旅
---

> 鸟哥的《Linux私房菜-基础学习篇》学习笔记（从第六章起）

# 第六章  Linux的文件权限与目录配置

## 6.1用户与用户组

#### 1.文件所有者

#### 2.用户组

#### 3.其他人

---

## 6.2文件权限概念

### 6.2.1 文件属性

- `ls -a` 查看所有文件（包括隐藏文件）


- `ls -al` 查看文件名及其属性
  - 第一列第一个字符代表文件的**类型**
    - `d`：目录
    - `-`：文件
    - 其余9个字符按次序分别代表 **owner**、**group**、**others **各自的 **read**( **`r`** )、**write**( **`w`** )、**execute**( **`x`** ) 权限

  - 第二列表示有多少个文件连接到**同一个节点（i-node）**

  - 第三列表示这个文件（或目录）的 **“所有者账号”**

  - 第四列表示这个文件的**所属用户组**

  - 第五列代表文件的容量，单位为 **`B`**

  - 第六列为**创建文件**日期或是**修改**时间
    - 若是修改时间距离现在太久了，就只会月/日/年
    - `ls -l --full-time` ：显示**完整**的时间

  - 第七列是文件名
    - 文件名之前多一个点，代表这个文件为**`隐藏文件`**

    - 例1：

      ```sh
      drwxr-xr--    1    test1    testgroup    4096    Jun 19 10:25    groups/

      others 是否可以进入此目录？
      ```

    - 例2：

      ```shell
      -rwxrwxr--    1     root        wheel      13     Feb 17 11:01     XH.sh

      ./XH.sh 能否执行？

      sh XH.sh 呢？（可以不要执行权限）
      ```


---

### 6.2.2 改变文件属性与权限

- **`chgrp`：即`change group`，改变文件所属用户组**
  - **`chgrp  [-R]  组名  文件或目录`**
    - 组名必须在 `/etc/group` 内存在

    - **`-R`**：连同子目录下的所有文件、目录

    - 例：

      ```shell
      chgrp wheel XH.sh
      ```



- **`chown`：即 `change owner`，改变文件所有者**

  - **`chown [-R] 账户名称 文件或目录`**
    - 改变文件所有者

    - **`-R`**：同上

    - 例：

      ```shell
      chown root XH.sh
      ```
  - **`chown [-R] 账户名称:组名 文件或目录`**
    - 改变文件所有者以及用户组

    - **`-R`**：同上

    - 例：

      ```shell
      chown root:wheel XH.sh
      ```
  - **`cp  源文件  目标文件`**
    - 若目标文件不存在，则将会把源文件的属性和权限复制给目标文件
    - 若目标文件存在，则只会覆盖目标文件的内容



- **`chmod`：即 `change mode`，改变文件权限**

  - 使用数字进行修改

    - **`r`**：4，**`w`**：2，**`x`**：1

    - 每种身份的3个权限都是累加的，比如：

      - 假设： 欲将  test  的权限改为 `- r w x   r - x   - - -`

        owner = `r w x` = 4+2+1 = 7

        group = `r - x` = 4+0+1 = 5

        others = `- - -` = 0+0+0 = 0

        则该文件的`权限数字`为750

        **`chmod  [-R]  权限数字  文件或目录`**

        即：`chmod 750 test`

  - 使用符号进行修改

    - **`u`**：user，**`g`**：group，**`o`**：others，**`a`**：all

    - **`+`**：加入，**`-`**：除去，**`=`**：设置

    - 例：

      ```shell
      chmod  u=rwx,g=r,o=rx  test

      chmod  u=rwx,go=rx  test

      chmod  a+x  test

      chmod  a-r  test
      ```



- 例如你从网络上下载一个可执行文件，但偏偏在你的 **`Linux`** 系统中就是无法执行，那么就是可能文件的属性和权限被修改了

---

### 6.2.3目录与文件的权限意义

#### 权限对文件的重要性

- **`r`（read）：可读取文件的实际内容**
- **`w`（write）：可修改该文件的内容（不含删除文件）**
- **`e`（execute）：该文件可被系统执行**
- **权限主要是针对文件的内容，与文件名没有关系**

#### 权限对目录的重要性

- **`r`（read）：可读取目录结构列表**
- **`w`（write）：可更改该目录结构列表**
  - 新建新的文件与目录
  - 删除已经存在的文件与目录（不论该文件与目录的权限为何）
  - 将已存在的文件或目录进行重命名
  - 转移该目录内的文件、目录位置


- **`e`（execute）：能否进入该目录成为工作目录**

- 例：

  ```shell
  dr-xr-xr-x    29   smartestee   staff    986   Feb 17 11:56    scripts

  -r--r--r--    1       root      wheel     24   Feb 17 11:35     XH.sh
  ```

  (1).查看 `scripts` 文件列表

  (2).修改 `XH.sh` 的文件名

  (3).进入 `scripts` 目录

  (4).查看 `XH.sh` 的文件内容

  (5).修改 `XH.sh` 的文件内容

  (6).执行 `XH.sh`

---

### 6.2.4文件种类与扩展名

#### 文件种类

- **普通文件（regular  file）`-`**

  - 纯文本文件（**ASCII**）

    - 内容为我们可以直接读取的数据，几乎我们用来作为设置的文件都属于 **ASCII file**
    - 可用 **`cat`** 读取数据
  - 二进制文件（**binary**）
    - 可执行文件
    - 例：**`cat`**
  - 数据格式文件（**data**）
    - 有些程序在运行的过程中会读取某些特定格式的文件，这种文件就是 **data file**
    - 用户登录时，登录的数据将会记录在 **/var/log/wtmp** 这个文件内，该文件就是一个 **data file**，使用 **`last`** 就可以读取，若是使用 **`cat`** 读取，就会出现乱码


- **目录（directory）`d`**

- **连接文件（link）`l`**

  - 类似于 **Windows** 的快捷方式

- **设备与设备文件（device）**

  - 与系统外设及储存等相关的一些文件，通常放在 **/dev**


- **块（block）设备文件 `b`**
    - 一些存储数据，以提供系统随机访问的接口设备，例如硬盘、软盘等


- **字符（character）设备文件 `c`**
    - 一些串行端口的接口设备，例如键盘、鼠标等

- **套接字（sockets）`s`**

    - 数据接口文件，通常用于网络上的数据连接，在 **/var/run** 就可以看到这种文件类型

- **管道（FIFO，pipe）`p`**

    - `first - in - first - out`
    - **FIFO** 也是一种特殊的文件类型，主要目的在于解决多个程序同时访问一个文件所造成的错误问题


#### 文件扩展名

**常见：**

- **脚本或批处理文件**
  - `*.sh`
- **打包过的压缩文件**
  - `*Z`
  - `*.tar`
  - `*.tar.gz`
  - `*.zip`
  - `*.tgz`
  - …（不同的压缩文件，不同的扩展名）
- **网页相关文件**
  - `*.html`
  - `*.php`
- ...

#### 文件名长度限制（默认：Ext2/Ext3 系统）

- **单一文件或目录的文件名最长为255个字符**
- **包含完整路径名称及绝对目录 `/` 的完整文件名为4096个字符**

#### 文件名限制

- **避免一些特殊字符**
  - `*`，`?`，`<`，`>`，`;`，`&`，`!`，`[`，`]`，`|`，`\`，`'`，`"`，`` `，`(`，`)`，`{`，`}`，`-`，`+`，`.`

---

# 第7章  Linux文件与目录管理

## 7.1目录与路径

### 7.1.1  相对路径与绝对路径

- **绝对路径写法一定要从 `根目录/` 写起**
- **对于文件名的正确性而言，`绝对路径` 的正确度要比相对路径好**

------

### 7.1.2  目录的相关操作

#### 较为特殊的目录

- **所有目录都会存在的两个目录**
  - `.`：代表此层目录
  - `..`：代表上一层目录
  - 根目录的这两个目录都是指根目录自己


- **`-`代表前一层工作目录**
- **`~`：代表“目前用户身份”所在主文件夹**
- **`~account(账户名)`：代表用户的主文件夹**

#### 常用的处理目录的命令

- **`cd`：Change  Directory，即切换目录**
  - `cd ~`或`cd`：回到主文件夹
  - `cd ~smartestee`：进入 **smartestee** 这个用户的主文件夹
  - `cd ..`：回到目前的上层目录
  - `cd -`：回到刚才的目录
- **`pwd`：Print  Working  Directory显示当前目录**
  - `pwd -p`：显示正确的完整路径，而不是显示连接文件的路径（例：`/var/spool/mail` 是 ` /var/mail` 的连接文件）
- **`mkdir`：新建一个新的目录**
  - `-p`：创建多层目录（例：`mkdir -p test1/test2/test3/test4`，即使目录存在，也不会报错）
  - `-m`：创建目录并强制设置权限（例：`mkdir -m 711 test2` ）
- **`rmdir`：删除一个空的目录**
  - `-p`：连同上层空的目录一起删除（例：`rmdir -p test1/test2/test3/test4` ）

------

### 7.1.3  关于执行文件路径的变量：$PATH

**将 /root 加入 PATH 中：`PATH="$PATH":/root`**

**PATH 哪个路径先被查询，就会优先执行该目录下的命令**

------

## 7.2  文件目录管理

### 7.2.1  查看文件与目录：`ls`

#### `-a`：查看全部文件，包括隐藏文件（常用）

#### `-A`：同上，但不包括 `.` 和 `.. `这两个目录

#### `-d`：仅列出目录本身，而不是目录内的文件数据（常用）

#### `-f`：直接列出结果，而不进行排序（ `ls` 默认以文件名从小到大排序）

#### `-F`：根据文件、目录等附加数据结构

- `*`：代表可执行文件
- `/`：代表目录
- `=`：代表 **socket** 文件
- `|`：代表 **FIFO** 文件

#### `-h`：将文件容量以 GB、KB 等形式列出来（默认为 B ）

#### `-i`：列出 inode 号码

#### `-l`：列出长数据串，包含文件属性和权限等（常用）

#### `-n`：列出 UID 与 GID ，而非用户与用户组的名称

#### `-r`：将排序结果反向输出

#### `-R`：连同子目录内容一起列出来，即将该目录下所有文件显示出来

#### `-S`：按文件容量从大到小排列

#### `-t`：按时间由近及远排序

#### `—color=`

- **`never`**：不要依据文件特性给予颜色显示
- **`always`**：显示颜色
- **`auto`**：根据系统自行依据设置来判断是否给予颜色

#### `--full-time`：以完整时间模式（年、月、日、时、分）显示

#### `--time={atime，ctime}`：输出访问时间或改变权限属性时间

------

### 7.2.2  复制、删除与移动：`cp`，`rm`，`mv`

#### `cp`：copy，复制文件或目录

**`cp [-a...] 源文件 目的文件`**

- **`-a`：相当于 `-pdr`（常用），将所有文件特性进行复制**
- **`-d`：若源文件为连接文件的属性，则复制连接文件属性而非文件本身**
- **`-f`：force，即强制**
- **`-i`：若目标文件已经存在，覆盖时会先询问操作的进行（常用）**
- **`-l`：进行硬连接的连接文件创建，而非复制文件本身**
- **`-p`：连同文件的属性一起复制**
- **`-r`：递归持续复制，用于目录的复制行为（常用）**
- **`-s`：软连接，复制成为符号连接文件，即“快捷方式”文件**
- **`-u`：若目标文件比源文件旧才进行复制**
- **默认条件下，目的文件的属性跟命令执行者有关**
- **复制前，必须了解到：**
  - 是否需要完整保留源文件的信息
  - 源文件是否为软连接文件？
  - 源文件是否为特殊的文件，例如：**FIFO**，**socket** 等
  - 源文件是否为目录？

#### `rm`：remove，移除文件或目录

- **`-f`：force，强制**
- **`-i`：互动模式，即删除前会询问用户是否操作**
  - **`rm -i bashrc*`**：将文件名开头为 **bashrc** 的文件全部删除
    - `*`：通配符，代表0到无穷多个任意字符


- **`-r`：递归删除，最常用在目录的删除**
  - 系统会一直询问是否删除，若想终止删除，可以按下 `ctrl + c `
  - 若确定要删除此目录而不要询问，在命令前加上 `\` 即可

#### `mv`：move，移动文件与目录，或更名

- **`-f`：force，强制，若目标文件已经存在，则系统不会询问就直接覆盖**
- **`-i`：若目标文件存在，就会询问是否覆盖**
- **`-u`：若目标文件存在，且源文件比较新，才会覆盖**
- **`mv  文件1  文件2`：重命名，将 “文件1” 改名为 “文件2”**
- **`mv  文件1  文件2 ... 文件3`：最后一个 “文件3” 一定是“目录”，即将前面所有数据移动到该目录**

------

### 7.2.3  取得路径的文件名与目录名称

#### dirname：取得目录名

```shell
bogon:test root#   dirname /Users/smartestee/test

/Users/smartestee
```

#### basename：取得最后的文件名

```shell
bogon:test root#   basename /Users/smartestee/test

test
```

------

## 7.3  文件内容查阅

### 7.3.1  直接查看文件内容

#### `cat`：由第一行开始显示文件内容

- **`-A`：相当于 `-vET` 的整合参数，可列出一些特殊字符，而不是空白而已**
- **`-b`：列出行号，仅对非空白行有效**
- **`-E`：将结尾的断行字符 `$` 显示出来**
- **`-n`：连同非空白行一起列出行号**
- **`-T`：将 `[Tab]` 按键以 `^I` 显示出来**
- **`-v`：列出一些看不出来的特殊字符**

#### `tac`：反向列式，即从最后一行开始显示（ `cat` 的倒写形式）

#### `nl`：添加行号打印

- **`-b`：指定行号指定的方式**
  - `-b a`：连同非空白行一起列出行号（类似于 `cat -n` ）
  - `-b t`：列出行号，对空行无效（类似于 `cat -b` ）

- **`-n`：列出行号表示的方法**
  - `-n ln`：行号显示在屏幕的最左方
  - `-n rn`：行号显示在字段的最右方，且不加0
  - `-n rz`：行号显示在字段的最右方，且加0，默认字段6位数

- **`-w`：行号字段占用的位数**

- 例：

  ```shell
  nl -b a -n rz -w 3 test2.txt
  ```

------

### 7.3.2  可翻页查看

#### `more`：一页一页地显示文件内容

**`more` 运行过程中：**

- `空格键`：向下翻一页，读取完文件，再按下此键即退出
- `Enter`：向下滚动一行，读取完文件，再按下此键即退出
- `/'字符串'`：查询功能，在显示的内容中查询“字符串”
- `:f`：显示出文件名及目前显示的行数
- `q`：退出 `more`
- `b 或 ctrl + b`：往回翻页，只对文件有用，对管道无用

#### `less`：与 `more` 类似，但是比 `more` 更好的是，它可以往前翻页

**`less`执行时：**

- `空格键`：向下翻动一页
- `[PageDown]`：向下翻动一页
- `[PageUp]`：向上翻动一页
- `/'字符串'`：向下查询“字符串”，附带颜色显示
- `?'字符串'`：向上查询“字符串”，附带颜色显示
- `n`：重复前一个查询
- `N`：反向重复前一个查询
- `q`：退出 `less`


- **`less` 使用的界面与 `man page` 非常类似，因为 `man` 就是调用 `less` 来显示说明文件的内容的**

------

### 7.3.3  数据选取

#### `head`：只看头几行

**`head [-n number] 文件`**

- `-n`：后面接数字，代表显示几行
  - 若为负数，即为除了后面的 `number` 行，其余的都显示出来

#### `tail`：只看结尾几行

**`tail [-n number] 文件`**

- `-n`：后面接数字，代表显示几行
- `-f`：持续检测后面所接的文件名，有数据写入就会显示在屏幕上，直到按下 `ctrl + c` 才会退出

------

### 7.3.4  非纯文本文件

#### od：以二进制的方式读取文件内容

**`od [-t TYPE] 文件`**

- `-t`：后面接各种 `TYPE`
  - `a`：利用默认的字符来输出
  - `c`：使用ASCII字符来输出
  - `d[size]`：利用十进制来输出数据，每个整数占用 `size` bytes
  - `f[size]`：利用浮点数来输出数据，每个数占用 `size` bytes
  - `o[size]`：利用八进制来输出数据，每个整数占用 `size` bytes
  - `x[size]`：利用十六进制来输出数据，每个整数占用 `size` bytes

------

### 7.3.5  修改时间或创建新文件

#### modification time（mtime）

- **当该文件的 “内容数据” 即文件的内容更改时，就会更新这个时间，使用 `ls` 默认的就是显示 mtime**

#### status time（ctime）

- **当该文件的 “状态”(status) 即权限或属性改变时，就会更新这个时间**

#### access time（atime）

- **当 “该文件的内容被取用” 时，就会更新这个读取时间(access)，例：使用 `cat` 读取某个文件**

#### touch

**`touch [-acdmt] 文件`**

- `-a`：仅修改访问时间
- `-c`：仅修改文件的时间，若文件不存在则不创建新文件
- `-d`：后面可接欲修改的日期而不用目前的日期，也可以使用 `--date=“日期或时间”`
  - 例：

    ```shell
    touch -d "2 days ago" test.sh		//将日期调整为两天前
    ```
- `-m`：仅修改 **mtime**
- `-t`：后面可接欲修改的日期而不用目前的日期，格式为 **[YYMMDDhhmm]**
  - 例：

    ```shell
    touch -t 151228 1000 test.sh		//将日期改为 2016/12/28 10:00
    ```


- **`touch`最常被使用的情况：**
  - 创建一个空的文件
  - 将某个文件日期修改为目前日期（ **mtime** 与 **atime** ）

------

## 7.4  文件与目录的默认权限与隐藏权限

### 7.4.1  文件默认权限：`umask`

#### 默认权限

- **创建文件**

  默认没有执行(**x**)权限，即默认权限为 `- rw- rw- rw-`


- **新建目录**

  默认所有权限开放，即默认权限为 `d rwx rwx rwx`

#### `umask`

```shell
bogon:~ smartestee$   umask -S

u=rwx,g=rx,o=rx		<==查询新建的目录的权限


bogon:~ smartestee$   umask

0022	<==与一般权限有关的是后面三个数字，第一个与特殊权限有关
```

- **`umask` 的分数指的是“默认权限需要减掉的权限”**

  - 新建文件

    ```shell
    (- rw- rw- rw-) - (- --- -w- -w-) ==> - rw- r-- r--
    ```

  - 新建目录

    ```shell
    (d rwx rwx rwx) - (d --- -w- -w-) ==> d rwx r-x r-x
    ```

  - 例：

    ```shell
    bogon:test02 smartestee$ umask
    0022

    bogon:test02 smartestee$ touch test01

    bogon:test02 smartestee$ mkdir test02

    bogon:test02 smartestee$ ls -l 
    total 0
    -rw-r--r--  1 smartestee  everyone   0 Feb 18 11:15 test01
    drwxr-xr-x  2 smartestee  everyone  68 Feb 18 11:15 test02
    ```

- **设置 umask**

  ```
  bogon:test02 smartestee$ umask 002

  bogon:test02 smartestee$ umask
  0002
  ...
  ```

- **注意：不要使用默认权限数值与 umask 数值进行权限相减**

  - 例如：**umask** 数值为003，新建文件时，666 - 663 = 663，即 `- rw- rw- -wx`，可原本文件的权限就已经去除 **`x`** 的默认属性

------

### 7.4.2  文件隐藏属性 `chattr`，`lsattr`

#### `chattr`，`lsattr ` 命令只能在 Ext2/Ext3 的文件系统上才会生效

#### `chattr`（设置文件的隐藏属性）

- **`+`，`-`，`=`：增加，删除，等于后面接的参数**
- **`a`：可让文件只能增加数据，而不能删除也不能修改数据，只有 root 才能设置**
- **`i`：可让文件 “不能被删除、改名、设置连接，也无法写入或添加数据”，只有 root 才能设置**

#### lsattr（显示文件的隐藏属性）

- **`-a`：隐藏文件的属性也列出来**
- **`-d`：如果接的是目录，仅列出目录本身的属性而非目录内的文件名**
- **`-R`：连同子目录的数据也一并列出来**

------

### 7.4.4  查看文件类型：`file`

**查看文件的基本数据，例如是属于 `ASCII` 或者是 `data` 文件，或者是 `binary`，抑或是其中有没有用到动态函数库(share library)等等的信息**

```shell
bogon:test02 smartestee$   file ~/.bash_profile

/Users/smartestee/.bash_profile: ASCII text
```

------

### 7.5  命令与文件的查询

#### 7.5.1  脚本文件名的查询

- **连续按两次 `[tab]` 就可以知道用户有多少命令可以执行**

- **which（寻找“执行文件”）**

  - `-a`：将所有由 `PATH` 中可以找到的命令均列出，而不只第一个被找到的命令

  ```shell
  bogon:~ smartestee$   which cd
  /usr/bin/cd

  bogon:~ smartestee$   which which
  /usr/bin/which

  bogon:~ smartestee$   which ls
  /bin/ls
  ```

------

#### 7.5.2  文件名的查找

**`whereis` 和 `locate` 是利用数据库来查找数据，速度快，而且没有实际查询硬盘，较节省时间**

**`whereis`（寻找特定文件）**

- `-b`：只寻找二进制格式文件
- `-m`：只找在说明文件 `manual` 路径下的文件
- `-s`：只找 `source` 源文件
- `-u`：查找不在上述三个选项中的其他特殊文件

```shell
bogon:~ root#   whereis cd

/usr/bin/cd
```

**locate**

- `-i`：忽略大小写
- `-r`：可接正则表达式的显示方式
- 依照 `/var/lib/mlocate` 内的数据库记载，找出用户输入的关键字文件名
- **updatedb**：根据 `/etc/updatedb.conf` 的设置去查找系统硬盘内的文件名，并更新 `/var/lib/mlocate` 内的数据库

------

### 7.6  权限与命令间的关系（极重要）

#### 让用户能进入某目录成为“可工作目录”的基本权限

- **可使用的命令：例如 `cd`**
- **目录所需权限：用户至少需要具有 `x` 的权限**
- **额外需求：若用户想在这个目录内利用 `ls` 查阅文件名，则用户还需要 `r` 的权限**

#### 用户在某个目录内读取一个文件的基本权限

- **可使用命令：例如 `cat，more，less` 等**
- **目录所需权限：用户至少需要具有 `x` 权限**
- **文件所需权限：用户至少需要具有 `r` 权限**

#### 用户修改一个文件的基本权限

- **可使用命令：例如 `nano，vi` 等**
- **目录所需权限：用户至少需要具有 `x` 权限**
- **文件所需权限：用户至少需要具有 `r，w` 权限**

#### 用户创建一个文件的基本权限

- **目录需要权限：用户至少需要具有 `w，x` 权限，重点在 `w`**

#### 用户执行某目录下的某个文件的基本权限

- **目录所需权限：用户至少需要具有 `x` 权限**
- **文件所需权限：用户至少需要具有 `x` 的权限**

#### 例：

(1).执行 `cp /dir1/file1 /dir2`，用户对dir1、file1、dir2至少需要具有什么权限？

- dir1：`x`
- file1：`r`
- dir2：`w，x`

(2).有一个目录及文件权限如下：

```shell
drwxr-xr-x   23   root        root      4096   Sep   22   12:09   /

drwxr-xr-x    6   root        root      4096   Sep   29   02:21   /home

drwx------    6   student     student   4096   Sep   29   02:23   /home/student

drwxr-xr-x    6   student     student   4096   Sep   29   02:24   /home/student/www

-rwxr--r--    6   student     student    369   Sep   29   02:27   /home/student/www/index.html
```

请问用户 `smartestee` 能否读取 `index.html` 这个文件？

## 未完待续...