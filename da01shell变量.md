
# Shell 脚本基础

> 本文档涵盖 Shell 脚本的执行方式、变量定义、环境变量设置、位置参数变量和预定义变量等核心知识。

---

## 1. Shell 脚本的执行方式

脚本首行必须声明 Shebang 行：

```bash
#!/bin/bash
echo "Hello, World!"
```

### 方式一：赋予执行权限后运行

```bash
# 赋予可执行权限
chmod +x helloworld.sh

# 运行脚本（支持绝对路径或相对路径）
./helloworld.sh
```

> **说明**：`chmod +x` 为文件添加可执行权限，之后通过 `./` 相对路径或绝对路径执行。

### 方式二：直接调用解释器运行（无需执行权限）

```bash
# 使用 bash 解释器
bash helloworld.sh

# 使用 sh 解释器
sh helloworld.sh
```

> **注意**：`bash` 和 `sh` 在某些语法上存在差异（如数组、`[[ ]]` 条件测试等），建议统一使用 `bash`。

---

## 2. Shell 变量（基础变量）

### 2.1 系统自带变量

| 变量 | 含义 |
|------|------|
| `$HOME` | 当前用户家目录 |
| `$PWD` | 当前工作目录 |
| `$SHELL` | 当前使用的 Shell 程序路径 |
| `$USER` | 当前登录用户名 |
| `$PATH` | 命令搜索路径 |
| `$UID` | 当前用户 ID |
| `$HOSTNAME` | 主机名 |
| `$LANG` | 系统语言环境 |

### 2.2 自定义变量的定义与撤销

```bash
# 定义变量（等号两边绝对不能有空格）
A=100

# 撤销变量
unset A

# 设置只读变量（一旦设置，无法 unset，除非退出当前 Shell）
readonly B=200
# 等同于
export -r C=300
```

### 2.3 变量命名规则

- 由**字母、数字、下划线**组成，**不能以数字开头**。
- 区分大小写，`name` 和 `NAME` 是两个不同的变量。
- 习惯上使用**大写字母**命名（如 `JAVA_HOME`、`CATALINA_HOME`）。
- 不能使用 Shell 关键字（如 `if`、`then`、`for`）作为变量名。

### 2.4 变量引用

```bash
name="World"

# 使用 $name 或 ${name} 引用
echo "Hello, $name"
echo "Hello, ${name}!"

# 推荐使用 ${} 形式，避免歧义
fruit="apple"
echo "I like ${fruit}s"    # 正确：输出 "I like apples"
echo "I like $fruits"      # 错误：会查找变量 $fruits
```

### 2.5 将命令返回值赋给变量（命令替换）

```bash
# 反引号形式
A=`date`

# $() 形式（推荐，更直观，支持嵌套）
A=$(date)

# 嵌套示例
echo $(basename $(dirname /opt/tomcat/conf/server.xml))
```

> **推荐**：`$()` 形式优于反引号，原因是：
> 1. 可读性更好，不易与单引号混淆。
> 2. 支持嵌套，反引号嵌套需要转义，写法复杂。

### 2.6 变量类型声明（bash 扩展）

```bash
# 声明整数变量（支持算术运算）
declare -i num=10+20
echo $num    # 输出 30

# 声明只读变量
declare -r pi=3.14

# 声明数组变量
declare -a arr=(a b c)
echo ${arr[1]}    # 输出 b

# 声明关联数组（bash 4.0+）
declare -A map=([key1]=val1 [key2]=val2)
echo ${map[key1]}  # 输出 val1
```

---

## 3. 环境变量

环境变量可以被**当前 Shell** 以及它的**子 Shell（子进程）** 继承和使用。

### 3.1 临时生效（仅当前终端及子进程有效）

```bash
export TOMCAT_HOME=/opt/tomcat

# 验证
echo $TOMCAT_HOME
```

> 关闭终端后变量自动失效。

### 3.2 永久生效（写入配置文件）

| 配置文件 | 生效范围 | 适用场景 |
|----------|----------|----------|
| `/etc/profile` | 所有用户 | 全局环境变量 |
| `~/.bash_profile` | 当前用户 | 登录时加载 |
| `/etc/bashrc` | 所有用户 | 每次打开终端加载 |
| `~/.bashrc` | 当前用户 | 每次打开终端加载 |
| `~/.profile` | 当前用户 | 登录 Shell 加载 |

**操作步骤：**

```bash
# 1. 将 export 语句追加到配置文件末尾
echo 'export TOMCAT_HOME=/opt/tomcat' >> ~/.bashrc

# 2. 让配置在当前窗口立即生效
source ~/.bashrc
# 或等价写法
. ~/.bashrc
```

### 3.3 查看与删除环境变量

```bash
# 查看所有环境变量
env
# 或
printenv

# 查看指定环境变量
echo $TOMCAT_HOME

# 删除（取消）环境变量
unset TOMCAT_HOME
```

---

## 4. 位置参数变量（向脚本传参）

在执行脚本时，可通过命令行传递参数，脚本内部通过以下变量获取：

| 变量 | 含义 |
|------|------|
| `$0` | 脚本命令本身（脚本文件名） |
| `$1` ~ `$9` | 第 1 到第 9 个参数 |
| `${10}` | 第 10 个及以后的参数（需用大括号包裹） |
| `$*` | 所有参数，视为**一个整体字符串** |
| `$@` | 所有参数，每个参数**独立对待**（常用于循环遍历参数） |
| `$#` | 参数的**总个数** |

### 示例脚本

```bash
#!/bin/bash
echo "脚本名称: $0"
echo "第1个参数: $1"
echo "第2个参数: $2"
echo "第10个参数: ${10}"
echo "参数总个数: $#"
echo "所有参数(*): $*"
echo "所有参数(@): $@"
```

执行：

```bash
bash demo.sh a b c d e f g h i j k
```

### `$*` 与 `$@` 的区别

```bash
#!/bin/bash
echo "--- 使用 \$* ---"
for var in "$*"; do
    echo "$var"
done

echo "--- 使用 \$@ ---"
for var in "$@"; do
    echo "$var"
done
```

当传入参数 `A B C` 时：
- `"$*"` 循环输出 1 次：`A B C`
- `"$@"` 循环输出 3 次：`A`、`B`、`C`

---

## 5. 预定义变量（系统自动赋值）

Shell 预先定义好的特殊变量，不需要手动赋值，直接使用：

| 变量 | 含义 |
|------|------|
| `$$` | 当前 Shell 进程的 PID（进程号） |
| `$!` | 后台运行的最后一个进程的 PID |
| `$?` | 上一条命令的返回状态码（`0` 表示成功，非 `0` 表示失败） |
| `$_` | 上一条命令的最后一个参数 |
| `$-` | 当前 Shell 的选项标志 |

### 示例

```bash
#!/bin/bash

# 获取当前脚本进程 PID
echo "当前进程 PID: $$"

# 后台运行命令，获取其 PID
sleep 100 &
echo "后台进程 PID: $!"

# 检查上一条命令是否成功
ls /nonexistent_dir 2>/dev/null
if [ $? -eq 0 ]; then
    echo "命令执行成功"
else
    echo "命令执行失败，返回码: $?"
fi
```



