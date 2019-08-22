---
title: Shell Expansion
date: 2019-08-21 17:55:57
toc: true
tags:
- shell
---

## 说明

在将命令行拆分为 tokenS 后，将在命令行上执行扩展。

## 七种 expansion

### [Brace Expansion](https://www.gnu.org/software/bash/manual/html_node/Brace-Expansion.html#Brace-Expansion)

括号内扩展，可以嵌套，每个扩展的结果都没有排序，保持从左到右的顺序。

#### 表达式一，用逗号分割

```bash
$ echo a{b,c,d}e
abe ace ade
```

#### 表达式二，`{x..y[..incr]}`

x 和 y 可以是数字或者字母(单字母), incr 是 int 类型递增量，默认是 1 或者 -1:

- 数字

```bash
$ echo a{10..40..10}c
a10c a20c a30c a40c
```

- 子母

```bash
$ echo a{a..c}d
aad abd acd
```

#### 注意点

- Brace expansion 会优先于其他扩展执行，并且在结果中保留对其他扩展特殊的任何字符。这是严格的文字话的，bash 不对 brace expansion 之间的文本应用任何语法解释

- 包含不带引号的 `{` 或 `}`，以及至少一个不带引号的 `,` 或者有效的序列表达式

- `{` 或 `,` 可以用 `\` 引用来防止它们被视为 brace expansion 表达式的一部分。为了避免冲与 parameter expansion 冲突，字符串 `${` 对 brace expansion 来说是非法的，并且禁止 brace expansion 直至遇到一个 `}`

### [Tilde Expansion](https://www.gnu.org/software/bash/manual/html_node/Tilde-Expansion.html#Tilde-Expansion)

波浪线扩展，下面显示了 bash 如何处理不带引号的波浪号前缀：

表达式 | 描述
--- | ---
`~` | $HOME 值
`~/foo`| $HOME/foo
`~fred/foo` |用户 fred 下的 foo 目录，即 /Users/fred/foo
`~+/foo` | $PWD/foo
`~-/foo` | ${OLDPWD-'~-'}/foo
`~N` | 等同于 `dirs +N`
`~+N` | 等同于 `dirs +N`
`~-N` | 等同于 `dirs -N`

#### 注意点

- 当 Bash 作为简单命令的参数出现时，Bash 还会对满足变量赋值条件的单词执行波浪扩展（请参阅[Shell参数](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameters.html#Shell-Parameters）)。在 POSIX 模式下，Bash 不执行此操作，但上面列出的声明命令除外。

### [Parameter Expansion](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html#Shell-Parameter-Expansion)

参数扩展：

- 普通扩展，如果省略冒号，则不检查是否为 null，而是只检查是否存在

表达式 | 描述
--- | ---
${parameter:-word} | 如果 parameter 未设置或者为 null，则替换为 word
${parameter:=word} | 如果 parameter 未设置或者为 null，则将 word 赋值给 parameter，不能以这种方式赋值给 positional parameters 和 special parameters
${parameter:?word} | 如果 parameter 未设置或者为 null，则报错并写入 stderr，如果 shell 不是交互式的就退出，否则参数值将被替换
${parameter:+word} | 如果 parameter 未设置或者为 null，则不替换，否则替换

- 间接扩展

```bash
${!parameter}
# 相当于${var}，而var=${parameter}。比如说：
$ param="var"
$ var="Hello, Bread"
$ echo ${!param}
Hello, Bread
```

- 子串扩展（截取子串）

表达式 | 描述
--- | ---
${parameter:offset} | 省略 length，从 offset 至末尾
${parameter:offset:length} | 如果 offset 小于零，则该值将用作 parameter 末尾的字符偏移量。如果 length 小于零，则将其解释为字符与 parameter 末尾的偏移量而不是字符数。请特别注意：负偏移量必须与冒号分隔至少一个空格以避免 `:-` expansion。

以下是一些示例，说明 parameter 和下标数组的字符串扩展：

```bash
$ str=01234567890abcdefgh
$ echo ${str:7}
7890abcdefgh
$ echo ${str:7:0}

$ echo ${str:7:2}
78
$ echo ${str:7:-2}
7890abcdef
$ echo ${str: -7}
bcdefgh
$ echo ${str: -7:0}

$ echo ${str: -7:2}
bc
$ echo ${str: -7:-2}
bcdef
```

如果参数是 `'@'`，结果是从 offset 开始的 length 长度的参数。相对于大于最大位置参数的负偏移，因此偏移 -1 为最后一个参数的位置。如果 length 小于零，则报错。

```bash
$ set -- 1 2 3 4 5 6 7 8 9 0 a b c d e f g h
$ echo ${@:7}
7 8 9 0 a b c d e f g h
$ echo ${@:7:0}

$ echo ${@:7:2}
7 8
$ echo ${@:7:-2}
bash: -2: substring expression < 0
$ echo ${@: -7:2}
b c
$ echo ${@: -7:0}
/bash 1 2 3 4 5 6 7 8 9 0 a b c d e f g h
$ echo ${@:0:2}
/bash 1
$ echo ${@: -7:0}

```

如果参数是由 `@` 或 `*` 索引的数组，结果是从 offset 开始的 length 长度的参数。相对于大于最大位置参数的负偏移，因此偏移 -1 为最后一个参数的位置。如果 length 小于零，则报错。

```bash
$ array=(0 1 2 3 4 5 6 7 8 9 0 a b c d e f g h)
$ echo ${array[@]:7}
7 8 9 0 a b c d e f g h
$ echo ${array[@]:7:2}
7 8
$ echo ${array[@]: -7:2}
b c
$ echo ${array[@]: -7:-2}
bash: -2: substring expression < 0
$ echo ${array[@]:0}
0 1 2 3 4 5 6 7 8 9 0 a b c d e f g h
$ echo ${array[@]:0:2}
0 1
$ echo ${array[@]: -7:0}

```

应用于关联数组的子串扩展会产生未定义的结果

除非使用位置参数，否则子串索引从零开始，在这种情况下，索引默认从 1 开始。如果 offset 为0，并且使用了位置参数，则会在 list 中添加 `$@` 前缀

- 查找参数名

```bash
${!prefix*}
${!prefix@}
```

shell 会自动将其展开为所有以 prefix 开头的参数名。如：

``` bash
$ var1=
$ var2=
$ echo ${!var*}
var1 var2
```

- 数组 key 展开

```bash
${!name[@]}
${!name[*]}
```

如果 name 是数组变量，则展开到 name 中指定的数组索引列表。如果 name 不是数组，则在设置 name 时扩展为 0，否则为 null。用 `@` 的时候，扩展出现在双引号内，每个键扩展为一个单独的单词。

- 长度

```bash
${#parameter}
```

例子:

```bash
$ name=123456789
$ echo ${#name}
9
$ array=(a b c)
$ echo ${#array[@]} # echo ${#array[*]}
3
```

