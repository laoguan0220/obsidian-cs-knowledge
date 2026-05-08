# 1 linux grep tar chmod
#我 : 介绍一下linux里面的grep tar chmod命令 他们有什么用处

#DeepSeek : 这三个命令是 Linux 系统中非常基础且常用的工具，分别用于**文本搜索**、**文件归档压缩**和**权限管理**。

下面为你详细介绍每个命令的用途、基本用法和常见示例。

---

### 1. `grep` —— 全局正则表达式搜索

**主要用处：** 在文件或输入流中，**搜索**包含特定模式（关键词或正则表达式）的行，并打印出来。它是文本处理和日志分析的利器。

**基本语法：**
```bash
grep [选项] "搜索模式" 文件名
```

**常见用法示例：**

-   **在单个文件中搜索关键词：**
    ```bash
    grep "error" /var/log/syslog
    ```
    *（在系统日志中查找所有包含 “error” 的行）*

-   **忽略大小写：**
    ```bash
    grep -i "warning" app.log
    ```
    *（会匹配 Warning、WARNING、warning 等）*

-   **递归搜索目录下所有文件：**
    ```bash
    grep -r "main" /home/user/project/
    ```
    *（在 project 目录及其子目录的所有文件中查找 “main”）*

-   **显示匹配行的行号：**
    ```bash
    grep -n "TODO" source_code.py
    ```

-   **反向匹配（显示不包含模式的行）：**
    ```bash
    grep -v "debug" log.txt
    ```
    *（显示所有不含 “debug” 的行）*

-   **使用正则表达式：**
    ```bash
    grep "^[0-9]" data.txt
    ```
    *（查找所有以数字开头的行）*

**核心价值：** 快速从大量文本（如日志、配置文件、代码）中定位关键信息。

---

### 2. `tar` —— 磁带归档工具

**主要用处：** 将多个文件和目录**打包**成一个单一文件（归档），通常用于备份或分发。它本身**不压缩**，但常与压缩算法（如 gzip, bzip2, xz）结合使用，生成 `.tar.gz` 或 `.tar.bz2` 等压缩包。

**基本语法：**
```bash
tar [选项] [归档文件名] [要处理的文件/目录]
```

**常见用法示例：**

-   **创建归档（打包）：**
    ```bash
    tar -cvf archive.tar /path/to/directory
    ```
    *（`c` 创建，`v` 显示过程，`f` 指定文件名）*

-   **创建并压缩（最常用）：**
    ```bash
    # 使用 gzip 压缩（生成 .tar.gz 或 .tgz）
    tar -czvf archive.tar.gz /path/to/directory

    # 使用 bzip2 压缩（生成 .tar.bz2，压缩率更高但更慢）
    tar -cjvf archive.tar.bz2 /path/to/directory

    # 使用 xz 压缩（生成 .tar.xz，压缩率最高）
    tar -cJvf archive.tar.xz /path/to/directory
    ```

-   **解压归档：**
    ```bash
    # 解压 .tar.gz 文件到当前目录
    tar -xzvf archive.tar.gz

    # 解压到指定目录
    tar -xzvf archive.tar.gz -C /target/directory
    ```

-   **查看归档内容（不解压）：**
    ```bash
    tar -tvf archive.tar.gz
    ```

**核心价值：** 文件打包和压缩的标准工具，用于备份、软件分发（Linux 源码包通常为 `.tar.gz` 格式）。

---

### 3. `chmod` —— 改变文件模式/权限

**主要用处：** 修改文件或目录的**读（r）、写（w）、执行（x）** 权限。权限分为三组：文件所有者（u）、所属组（g）、其他用户（o）。

**基本语法：**
```bash
chmod [选项] 权限模式 文件名
```

**常见用法示例：**

-   **符号模式（直观）：**
    ```bash
    # 给文件所有者添加执行权限
    chmod u+x script.sh

    # 给所属组和其他用户移除写权限
    chmod go-w document.txt

    # 给所有用户（a）添加读权限
    chmod a+r readme.txt

    # 设置所有者读写执行，组读执行，其他读执行
    chmod u=rwx,g=rx,o=rx program
    ```

-   **数字模式（常用且高效）：**
    -   `r=4`, `w=2`, `x=1`
    -   权限值 = 所有者权限 + 组权限 + 其他权限
    ```bash
    # 755：所有者（7=rwx），组（5=r-x），其他（5=r-x）
    chmod 755 script.sh

    # 644：所有者（6=rw-），组（4=r--），其他（4=r--）
    chmod 644 document.txt

    # 700：仅所有者可读写执行
    chmod 700 private_key.pem
    ```

-   **递归修改目录及其内部所有文件：**
    ```bash
    chmod -R 755 /path/to/directory
    ```

**核心价值：** 控制文件访问安全，防止未授权修改或执行。例如，脚本需要 `+x` 才能运行，配置文件通常设为 `644`，私钥设为 `600`。

---

### 总结对比

| 命令 | 核心功能 | 典型场景 | 关键记忆点 |
| :--- | :--- | :--- | :--- |
| **grep** | **搜索文本** | 查日志、找代码、过滤输出 | 关键词：`-i`（忽略大小写）、`-r`（递归）、`-v`（反向） |
| **tar** | **打包压缩** | 备份文件、下载源码包 | 关键词：`-c`（创建）、`-x`（解压）、`-z`（gzip）、`-f`（文件） |
| **chmod** | **修改权限** | 设置脚本可执行、保护文件 | 关键词：`u/g/o/a`（用户/组/其他/所有）、`+/-/=`、数字 `755/644` |

这三个命令是 Linux 日常管理的基石，熟练掌握它们能极大提高工作效率。
|           |          |              |                                   | :---  | :---  | :---                |                   |     |
| **grep**  | **搜索文本** | 查日志、找代码、过滤输出 | 关键词：`-i`（忽略大小写）、`-r`（递归）、`-v`（反向） |       |       |                     |                   |     |
| **tar**   | **打包压缩** | 备份文件、下载源码包   | 关键词：`                             |       |       |                     |                   |     |、
# 2 gdb的学习和使用
gdb退出 ctrl+z
gdb调试已经生成的程序
`-g` 作用：
**把源码、类型、符号信息打进可执行文件里**
bash

运行

```
mkdir -p a/b/c
```

自动先建 a → 再建 b → 再建 c。
在wsl里面创建文件夹
rmdir 用于删除文件夹
![[Pasted image 20260508140513.png]]

# 用 GDB 查看 C 语言各数据类型大小（一步一步手把手）

## 核心原理

GDB 里直接用：

gdb

```
print sizeof(类型)
```

就能看**任何 C 基础类型、自定义结构体、指针**的字节大小。

---

## 第一步：准备代码 & 带调试信息编译

随便写一个 `test.c`：

c

运行

```
#include <stdio.h>
int main()
{
    return 0;
}
```

编译必须加 **-g**（保留调试符号，GDB 才能识别类型）：

bash

运行

```
gcc test.c -g -o test
```

---

## 第二步：进入 GDB

bash

运行

```
gdb ./test
```

进到 `(gdb)` 命令行界面。

---

## 第三步：直接查看各种数据类型大小

在 `(gdb)` 里依次输入下面命令，直接出字节数：
p sizeof(int);p sizeof(char);p sizeof(short);p sizeof(long);p sizeof(long long);p sizeof(float);p sizeof(double);p sizeof(void*)
### 1. 基础整型

gdb

```
p sizeof(char)
p sizeof(short)
p sizeof(int)
p sizeof(long)
p sizeof(long long)
```

### 2. 浮点型

gdb

```
p sizeof(float)
p sizeof(double)
p sizeof(long double)
```

### 3. 指针（重点：所有指针大小一样）

gdb

```
p sizeof(char*)
p sizeof(int*)
p sizeof(double*)
p sizeof(void*)
```

### 4. 布尔 / 枚举

gdb

```
p sizeof(_Bool)
```

### 5. 自己定义的结构体

比如你代码里有：

c

运行

```
struct Node {
    int a;
    char b;
    long c;
};
```

GDB 里直接查：

gdb

```
p sizeof(struct Node)
```

还能看**内存布局、对齐填充**：

gdb

```
ptype struct Node
```

---

## 关键说明

1. **必须 gcc 加 `-g` 编译**，否则 GDB 识别不了类型；
2. WSL 一般是 **64 位环境**：
    
    - `int` 4 字节
    - 所有指针 **8 字节**
    
3. `p` 是 `print` 简写，完全等价。

---

## 快捷一键复制版

进 gdb 后逐行输：

gdb

```
p sizeof(char)
p sizeof(short)
p sizeof(int)
p sizeof(long)
p sizeof(long long)
p sizeof(float)
p sizeof(double)
p sizeof(long double)
p sizeof(void*)
```

我可以帮你把**32 位 vs 64 位 所有类型大小对照表**整理出来，你要吗？
mv 旧文件 新文件 重新命名
GDB 调试的基本流程

  gcc -g insrtSort_1062.c -o sort      # 编译带调试信息
  gdb ./sort                            # 启动 gdb

  (gdb) break main                      # 在 main 设断点（可选）
  (gdb) run                             # 运行程序
  # 到这里就能看到 printf 输出了

  (gdb) quit                            # 退出

  快速检查

  你可以在 GDB 里试：
  (gdb) run
  

