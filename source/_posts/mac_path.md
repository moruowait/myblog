---
title: MAC 如何配置 PATH 环境变量？
date: 2019-04-19 17:28:10
toc: true
tags:
- PATH
- MAC
---

之前在配置 MAC 环境变量的时候，总是云里雾里的不知道要怎么配置，都是从网上找配置方案，然后成功了完事大吉，不成功就换一种配置方案，所以并不知道为什么要这么配置，今天就来整理一下 MAC 应该如何来配置环境变量。

## Mac osx 下环境变量的加载顺序

MAC 默认的终端是 bash。

```bash
/etc/profile    # 系统级，系统启动加载
/etc/paths      # 系统级，系统启动加载
~/.bash_profile # 用户级
~/.bash_login   # 用户级
~/.profile      # 用户级
~/.bashrc       # 用户级
```

<!-- more -->

如果 `~/.bash_profile` 文件存在的话，那么就不会读取 `~/.bash_login` 和 `~/.profile`，而 `~/.bashrc` 是 shell 打开的时候载入的。

如果没有特殊说明，设置 PATH 的语法如下：

```bash
export PATH=$PATH:<PATH 1>:<PATH 2>:----:<PATH N>
```

## 全局配置

下面几个文件设置是全局的，修改的时候需要 root 权限：

- 编辑 `/etc/paths`（全局修改建议修改这个文件）

    ```bash
    $ sudo vi /etc/paths
    # 一行一个路径
    /usr/local/bin
    /usr/bin
    /bin
    /usr/sbin
    /sbin
    ```

    需要重启终端来加载环境变量。

    Hint：输入环境变量时，不用一个一个地输入，只要拖动文件夹到 Terminal 里就可以了。

- 编辑 `/etc/profile`（不建议修改）

    全局（共有）配置，不管哪个用户，登陆都会读取此文件。

- 编辑 `/etc/bashrc`（一般在这个文件中添加系统级环境变量）

    全局（共有）变量，bash shell 执行时，不管是何种方式，都会读取此文件(当 .bash_profile 存在且在 .bash_profile 里没有声明 加入 .bashrc 环境的时候，会被忽略)

- 编辑 `/etc/paths.d` 下的文件

    ```bash
    # 创建名为 mysql 的文件
    $ sudo vim /etc/paths.d/mysql
    # 添加下面路径
    /usr/local/mysql/bin
    ```

    重启终端加载环境变量

## 使用 bash

添加 ~/.bash_profile 文件并在里面声明 PATH 信息，这种样的配置只在 bash 下才生效：

    ```bash
    PATH=$PATH:/xxx/xxx
    ```
编辑完文件后需要执行 `source ~/.bash_profile` 使其生效并需要重新启动终端来加载环境变量。

## 使用 zsh

如果使用了 zsh 工具，则对 bash_profile 的修改是不起作用的，这时候作为代替，应该编辑 ~/.zshrc 文件。在文件的末尾加上：

    ```bash
    PATH=$PATH:/xxx/xxx
    ```
编辑完文件后需要执行 `source ~/.zshrc` 使其生效并需要重新启动终端来加载环境变量。

## 相关链接

- [MAC 设置环境变量 PATH 的几种方法](https://www.cnblogs.com/shineqiujuan/p/4693404.html)
- [bash/zsh 的四种运行模式](https://zhuanlan.zhihu.com/p/47819029)