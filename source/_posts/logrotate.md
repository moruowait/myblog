---
title: 什么是 logrotate ？
date: 2019-04-18 18:28:44
toc: true
tags:
- 技术名词
- 运维
- Log
---

## 什么是 logrotate

logrotate 旨在简化生成大量日志文件的系统的管理。它允许自动循环、压缩、删除和邮寄日志文件。每个日志文件可以按每天、每周、每月的粒度来处理，也可以在其增长过大时处理。

通常来说，logrotate 作为日常 cron 作业运行。它一天内修改日志的次数不会超过一次，除非该日志的标准基于日志的大小，并且 logrotate 每天运行一次以上，或者使用 `-f` 或 `-force` 的选项。

在命令行上可以给出任意数量的配置文件。稍后的配置文件可能会覆盖前面文件中给出的选项，因此列出 logrotate 配置文件的顺序很重要。通常，应该使用一个配置文件，其中包含需要的任何其他配置文件。有关如何使用 include 指令来完成此任务的更多信息，请参见 man page。如果在命令行上给出一个目录，则该目录中的每个文件都是配置文件。

如果没有给出命令行参数，logrotate 将打印版本和版权信息，以及一个简短的使用总结。如果在循环日志时发生任何错误，logrotate 将以非零状态退出。

<!-- more -->

## 如何安装 logrotate

### 如何在 Mac 上安装 logrotate

* 先安装 Homebrew

    ```bash
    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
    ```

* 利用 Homebrew 安装 logrotate

    ```bash
    brew install logrotate
    ```

### 可能会遇到的问题

安装 logroate 后可能会遇到 logrotate 命令找不到的问题，其原因是 logrotate 安装到了 `/usr/local/sbin` 下，如果你的 PATH 环境变量没有该路径就找不到 logrotate，所以你需要将该路径加入 PATH 下。

## 查看 logrotate 文档

* `man logrotate` 命令可以在终端查看 logrotate 文档
* 如果你没有安装 logrotate 在终端 man page 下是找不到 logrotate 文档的，你可以点击[传送门](https://linux.die.net/man/8/logrotate)查看文档

## 怎么使用 logrotate

* 启动与停止

    ```bash
    brew services start logrotate   # 启动
    brew services stop logrotate    # 停止
    ```

* 选项

  * `-?，--help`：帮助。
  * `-d，--debug`：打开 debug 模式，日志和 logrotate 的状态文件不会被更新，只会打印 debug 信息。详细显示指令执行过程，便于排错活了解程序执行的情况。
  * `-f，--force`：让 logrotate 强制执行一次循环。有时，在向 logrotate 配置文件添加新条目之后，或者在手动删除旧日志文件时，这是非常有用的，因为将创建新文件，并且日志记录将正确地继续。
  * `-l，--log file`：让 logrotate 的 log 详细输出到 log_file 文件中。记录到该文件的详细输出与使用 -v 运行 logrotate 时相同。每次执行 logrotate 时都会覆盖日志文件。
  * `-m，--mail command`：邮寄日志，接受以下参数
    1. `-s subject`：标题
    2. 收件人

    然后命令必须读取标准输入上的消息并将其发送给收件人。默认的邮件命令是 `/bin/mail`。
  * `-s，--state statefile`：让 logrotate 使用别用状态文件。如果 logrotate 作为不同的用户运行于不同的日志文件集，这将非常有用。默认的状态文件为 `/usr/local/var/lib/logrotate.status`。
  * `usage`：使用指南
  * `-v，--verbose`：打开 verbose 模式，例如在循环期间显示消息
  * `--version`：版本
