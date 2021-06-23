第 29 章：数据存储：文件和数据库

### **1、你将在本章节学到什么？**

* 如何新建一个文件。
* 如何将数据写入文件。
* 如何新建 CSV 格式的文件。
* 如何查询以下数据库：
  * MySQL
  * PostgreSQL
  * MongoDB
  * ElasticSearch

###  **2、技术概念梳理**

* 文件权限
* 八进制
* SQL注入
* 预查询
* 无模式
* 关系数据库管理系统

### **3、介绍**

此章节，会通过 Go 使用不同技术存储数据。

### **4、新建一个文件**

使用 <code>os.Create</code> 方法可以新建一个文件：

```go
// data-storage/file-save/simple/main.go 
package main

import (
    "fmt"
    "os"
)

func main() {
    f, err := os.Create("test.csv")
    if err != nil {
        fmt.Println(err)
        return
    }
    //... success
}
```

我们新建一个命名为 <code>"test.csv"</code> 的文件。如果有报错的话，程序会打印错误并返回，程序代码如下：

```go
f, err := os.OpenFile("test.csv", os.O_RDWR|os.O_CREATE|os.O_TRUNC, 0666)
if err != nil {
    fmt.Println(err)
    return
}
```

<code>os.Create</code>  的内部通过以下参数调用 <code>os.Openfile</code> 方法，参数如下：

* 文件名（比如：<code>"test.csv"</code>）
* 文件模式（比如：<code>os.O_RDWR|os.O_CREATE|os.O_TRUNC</code> ）
* 文件权限模式（比如：<code>0666</code> ）

我们将在下面两个章节中知晓 **文件模式 和**文件权限模式**是什么。当然，如果你已经熟悉了这些章节内容，可以跳过。

### **5、文件模式标记**

你将通过系统调用的方式，使用 <code>os.Openfile</code> 打开一个文件。通过调用，系统需要清楚**文件路径**以及需要达到目的其他额外信息。这一系列信息包含在一个模式标记列表中，他们有不同类型的模式标记：

* **允许访问模式标记**：在新建一个文件时，必须指定其中一个标记作为入参参数。
* **允许执行模式标记**：系统执行文件操作时会被限制。
* **文件状态模式标记**：当文件打开时，会得到一个对文件操作的状态结果反馈。



| **标记类型** | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| os.O_RDONLY  | 以只读模式打开，只能读。                                     |
| os.O_WRONLY  | 以只写模式打开，只能写不能读。                               |
| os.O_RDWR    | 以读写模式打开。                                             |
| os.O_CREATE  | 新建文件若不存在。                                           |
| os.O_EXCL    | 若已使用 <code>os.O_CREATE</code> 模式，且文件已存在，则返回一个错误。 |
| os.O_TRUNC   | 打开文件时会被清空，若文件有内容则会清除。                   |
| os.O_APPEND  | 将内容写入追加到文件文本后面。                               |
| os.O_SYNC    | 以同步模式打开，程序会等待所有数据被完全写入为止。           |

文件打开模式标记

每种模式标记的值都是整型。你必须使用按位计算使用这些模式标记（看章节 [Bitmasks]）。当你使用 <code>os.Create</code> 时，文件将会通过以下模式标记打开：

```
os.O_RDWR|os.O_CREATE|os.O_TRUNC
```

这样意味着文件是通过**读写模式**打开的，若文件不存在则会新建。另外，系统会进行文件内容清空操作。

举例：若需要确定文件是否新建，可以通过添加模式标记：<code>os.O_EXCL</code>

```go
f, err := os.OpenFile("test.csv", os.O_RDWR|os.O_CREATE|os.O_TRUNC|os.O_EXCL, 0666)
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println(f)
```

上面程序执行后将输出：

```
open test.csv: file exists
```

哪种模式标记才是期望的呢？

### **6、文件模式 [sec: file-mode]**

在 UNIX 系统中，每一个文件都有权限设置：

* 文件权限属性的拥有者
* 文件权限所属用户组别
* 文件权限所属其他用户

若你想查看文件的权限信息，可以打开你的终端窗口，并输入 <code>ls -l</code> 命令。

```shell
$ ls -al
-rw-r--r--  1 maximilienandile  staff  242 25 nov 19:47 main.go
```

* **-rw-r--r--** ：文件权限模式
* **1** ：链接数字
* **maximilienandile** ：文件权限属性的拥有者
* **staff** ：文件权限所属用户组别
* **242** ：以 <code>byte</code> 为单位的文件大小
* **25 nov 19:47** ：最后修改日期时间

#### 6.0.0.1 符号表示

文件权限模式由三个区块组成（权限拥有者 user，权限所属组别 group，以及其他人 others ）

![File mode schema[fig:File-mode-schema]](https://www.practical-go-lessons.com/img/permissions.7a548570.png)

当看到第一个区块时，可以通过 3 个字母去定义权限，它们总是以这样的方式排序：

* **r** ：读
* **w** ：写
* **x** ：执行

第一个字符是连接号 <code>-</code> 。当文件是一个目录类型时，相当于目录英文第一个字母 <code>d</code>。如果文件权限模式的第一个字符是<code>-</code> ，这表示权限不可用。

下面通过 **-rw-r--r--** 例子的方式，说明：

* **First char**：这是一个文件
* **User（rw-）**：文件拥有者可读、可写、不可执行。
* **Group（r--）**：文件所属组别可读、不可写、不可执行。
* **Others（r--）**：文件所属组别可读、不可写、不可执行。

下面用 Go 实现文件模式：

```go
// data-storage/file-save/mode/main.go 

// 打开文件
f, err := os.Open("myFile.test")
if err != nil {
    fmt.Println(err)
    return
}
// 读取文件信息
info, err := f.Stat()
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println(info.Mode())
```

上述程序将输出：-rw-r--r-- 。

#### 6.0.0.2、数字表示

此模式还可以通过转换为 八进制 数字转换 （通过 <code>%o</code> 格式化）。

```go
fmt.Printf("Mode Numeric : %o\n", info.Mode())
fmt.Printf("Mode Symbolic : %s\n", info.Mode())
// 数字模式 : 644
// 字符模式 : -rw-r--r--
```

Go 在调用 <code>os.Openfile</code> 使用数字表示，由 3个数字组成：

![File mode numeric notation[图2]](https://www.practical-go-lessons.com/img/octal_file_mode.65204000.png)

数字(八进制)表示一组权限(见图2)。每个八进制数字(从0到7)都有特定的含义。

|         权限         | 符号             | 八进制 |
| :------------------: | ---------------- | ------ |
|          空          | <code>---</code> | 0      |
|        只执行        | <code>--x</code> | 1      |
|         只写         | <code>-w-</code> | 2      |
|    可写 & 可执行     | <code>-wx</code> | 3      |
|         只读         | <code>r--</code> | 4      |
|    可读 & 可执行     | <code>r-x</code> | 5      |
|     可读 & 可写      | <code>rw-</code> | 6      |
| 可读 & 可写 & 可执行 | <code>rwx</code> | 7      |

让我们举个例子: 644，我们分解：

* Owner（拥有者）：<code>6</code> 表示可读、可写
* Group（所属组别）： <code>4</code> 表示可读
* Others（其他人）： <code>4</code> 表示可读

另一个例子 777：

* Owner（拥有者）：<code>7</code> 表示可读、可写、可执行
* Group（所属组别）： <code>7</code> 表示可读、可写、可执行
* Others（其他人）： <code>7</code> 表示可读、可写、可执行

#### 6.0.0.3 通过 Go 更改文件权限

在一个文件上，你可以使用 <code>Chmod</code> 方法来更改文件模式

```go
// data-storage/file-save/mode/main.go 

err = f.Chmod(0777)
if err != nil {
    fmt.Println(err)
}
```

这里我们将文件的模式更改为0777。这意味着所有者、组和其他人的都更改为全部权限。

为什么 <code>0777</code> 要加0 ？它向 Go 表示该数不是十进制数而是八进制数。

### **7、写文件：以 CSV 为例**

### 7.1 什么是 CSV

CSV 表示逗号分隔值，它是一种用于存储数据的文件格式。

* 一个 CSV 文件由多行数据组成
* 每一行代表数据记录
* 每一个数据由 多个“区域” 组成
* 每一个逗号分隔符，分割每个“区域”
* 第一行的位置，通常被用作表头标题信息进行描述

举例如下：

```
age,genre,name
23,M,Hendrick
65,F,Stephany
```

#### 7.2、Code

在本节中，我们将看到如何将数据写入文件。为了立即应用我们的知识，我们将使用一个真实的用例:从一个切片创建一个CSV文件。

```go
// data-storage/file-save/writing/main.go 
package main

import (
    "bytes"
    "fmt"
    "os"
)

func main() {
    s := [][]string{
        {"age", "genre", "name"},
        {"23", "M", "Hendrick"},
        {"65", "F", "Stephany"},
    }
    // 打开文件或新建
    f, err := os.Create("myFile.csv")
    defer f.Close()
    if err != nil {
        fmt.Println(err)
        return
    }
    var buffer bytes.Buffer
    // 将切片 s 进行迭代
    for _, data := range s {
        buffer.WriteString(fmt.Sprintf("%s,%s,%s\n", data[0], data[1], data[2]))
    }
    n, err := f.Write(buffer.Bytes())
    fmt.Printf("%d bytes written\n", n)
    if err != nil {
        fmt.Println(err)
        return
    }
}
```

首先,我们新建一个切片 <code>s</code>，这是我们想要写入 CSV 文件的数据。

我们新建一个类型为 <code>bytes.Buffer</code> 变量 <code>buffer</code> 。

然后我们迭代 <code>s</code> 将数据添加到 <code>buffer</code>中，每个值由逗号分隔，并且通过字符“\ n”添加新行。
当 <code>buffer</code> 已就绪时，我们将其通过方法 <code>write</code> 写入文件中。 此方法将 **字节类型的切片变量** 作为参数，并返回两个值：

* 写入文件的字节数
* 错误结果

#### 7.3、 bytes.Buffer

我们在前面的程序中使用了一个数据缓冲区，缓冲区是物理存储器的一个区域，当数据从一处移动到另一处时，用来临时存储数据。