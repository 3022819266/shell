
# Shell 脚本基础知识



## 目录
- [1. 流程控制与判断语句（if 语句）](#1-流程控制与判断语句if-语句)
- [2. 判断语句](#2-判断语句)
- [3. 算术运算](#3-算术运算)
- [4. 常用判断条件](#4-常用判断条件)
- [速查要点](#速查要点)

## 1. 流程控制与判断语句（if 语句）

**基本语法：**

```bash
if [ 条件判断式 ]
then
    代码
fi
```

**示例：判断是否成年**

```bash
#!/bin/bash
age=20

if [ $age -ge 18 ]
then
    echo "已成年，年龄为 $age"
fi
```

运行输出：

```text
已成年，年龄为 20
```

**多分支语法：**

```bash
if [ 条件判断式 ]
then
    代码
elif [ 条件判断式 ]
then
    代码
fi
```

**示例：根据成绩输出等级**

```bash
#!/bin/bash
score=85

if [ $score -ge 90 ]
then
    echo "优秀"
elif [ $score -ge 60 ]
then
    echo "及格"
else
    echo "不及格"
fi
```

运行输出：

```text
及格
```

> [!WARNING]
> 中括号 `[ ]` 和内部的条件判断式之间**必须有空格**，否则会报错。

## 2. 判断语句

- **基本语法**：`[ condition ]`
- **注意事项**：`condition` 前后必须有空格。
- **返回值**：非空返回 `true`，可以使用 `$?` 变量来验证返回值（`0` 为 `true`，大于 `1` 为 `false`）。

**示例：用 `$?` 验证判断结果**

```bash
#!/bin/bash
[ 5 -gt 3 ]
echo $?    # 条件成立，输出 0（true）

[ 5 -lt 3 ]
echo $?    # 条件不成立，输出 1（false）
```

运行输出：

```text
0
1
```

**逻辑运算符应用：**

```bash
[ condition ] && echo OK || echo notok
```

条件满足执行 `OK`，不满足执行 `notok`。

**示例：短路判断目录是否存在**

```bash
#!/bin/bash
[ -d /etc ] && echo OK || echo notok
[ -d /no_such_dir ] && echo OK || echo notok
```

运行输出：

```text
OK
notok
```

## 3. 算术运算

**语法：**

1. `$((运算式))` 或 `$[运算式]`
2. `expr m + n`

> [!WARNING]
> 使用 `expr` 命令时，运算符前后必须有空格。

```bash
# 方式一
echo $((1 + 2))
echo $[3 * 4]

# 方式二（注意运算符两侧空格）
echo `expr 5 + 6`
```

**示例：计算两个变量的和与积**

```bash
#!/bin/bash
a=10
b=3

echo "和 = $((a + b))"      # 方式一：$(( ))
echo "积 = $[a * b]"         # 方式一：$[ ]
echo "差 = `expr $a - $b`"   # 方式二：expr，注意两侧空格
echo "商 = $((a / b))"
echo "余 = $((a % b))"
```

运行输出：

```text
和 = 13
积 = 30
差 = 7
商 = 3
余 = 1
```

**常用运算符：**

| 运算符 | 含义 |
| --- | --- |
| `+` | 加法 |
| `-` | 减法 |
| `\*` | 乘法 |
| `/` | 除法 |
| `%` | 取余 |

**示例：用 `expr` 演示各运算符（乘法需转义为 `\*`）**

```bash
#!/bin/bash
m=7
n=2

echo "加: `expr $m + $n`"   # 9
echo "减: `expr $m - $n`"   # 5
echo "乘: `expr $m \* $n`"  # 14（* 前需反斜杠）
echo "除: `expr $m / $n`"   # 3
echo "余: `expr $m % $n`"   # 1
```

## 4. 常用判断条件

**字符串比较**：使用 `=`

```bash
[ "ok" = "ok" ]
```

**示例：判断字符串是否相等**

```bash
#!/bin/bash
str1="hello"
str2="hello"

if [ "$str1" = "$str2" ]
then
    echo "两个字符串相等"
else
    echo "两个字符串不相等"
fi
```

运行输出：

```text
两个字符串相等
```

**两个整数比较：**

| 运算符 | 全称 | 含义 |
| --- | --- | --- |
| `-lt` | less than | 小于 |
| `-le` | little equal | 小于等于 |
| `-eq` | equal | 等于 |
| `-gt` | greater than | 大于 |
| `-ge` | greater equal | 大于等于 |
| `-ne` | not equal | 不等于 |

**示例：比较两个整数大小**

```bash
#!/bin/bash
x=8
y=5

[ $x -gt $y ] && echo "$x 大于 $y"
[ $x -eq $y ] && echo "$x 等于 $y" || echo "$x 不等于 $y"
```

运行输出：

```text
8 大于 5
8 不等于 5
```

**文件权限判断：**

| 选项 | 含义 |
| --- | --- |
| `-r` | 有读的权限 |
| `-w` | 有写的权限 |
| `-x` | 有执行的权限 |

**示例：判断脚本文件是否可执行**

```bash
#!/bin/bash
file="./test.sh"

if [ -x "$file" ]
then
    echo "$file 具有可执行权限"
else
    echo "$file 没有可执行权限"
fi
```

**文件类型判断：**

| 选项 | 含义 |
| --- | --- |
| `-f` | 文件存在并且是一个常规文件 |
| `-e` | 文件存在 |
| `-d` | 文件存在并是一个目录 |

**示例：判断路径是文件还是目录**

```bash
#!/bin/bash
target="/etc"

if [ -d "$target" ]
then
    echo "$target 是一个目录"
elif [ -f "$target" ]
then
    echo "$target 是一个普通文件"
else
    echo "$target 不存在"
fi
```

运行输出：

```text
/etc 是一个目录
```

### 综合示例：用位置参数变量比较两个数大小

**位置参数变量**：`$1`、`$2` … 分别代表脚本运行时传入的第 1、第 2 个参数，`$0` 为脚本名，`$#` 为参数个数。

```bash
#!/bin/bash
# 用法: ./compare.sh 88 66

num1=$1
num2=$2

if [ $num1 -gt $num2 ]
then
    echo "$num1 大于 $num2"
elif [ $num1 -lt $num2 ]
then
    echo "$num1 小于 $num2"
else
    echo "$num1 等于 $num2"
fi
```

运行方式：

```bash
chmod +x compare.sh
./compare.sh 88 66
```

运行输出：

```text
88 大于 66
```

> [!NOTE]
> `$1`、`$2` 接收的是命令行传入的参数，无需在脚本里写死数值，复用性更强。

## 速查要点
- `[ ]` 与条件之间、`condition` 前后**必须留空格**。
- `$?` 用于验证上一条命令返回值，`0` 代表 `true`。
- `expr` 运算符两侧**必须留空格**，乘法需写成 `\*`。
- 整数比较用 `-lt/-le/-eq/-gt/-ge/-ne`，**不要用 `>` `<`**（那是重定向）。
