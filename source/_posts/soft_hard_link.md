---
title: Linux 中硬连接（hard link）与软连接（symbolic link）的区别
date: 2019-04-22 12:17:10
toc: true
tags:
- 技术名词
- Linux
---

首先要简单了解一下 Linux 的 Ext 文件系统，此处[传送门](https://www.cnblogs.com/justmine/p/9128730.html)。

## Linux Ext文件系统

Linux 的 Ext 文件系统是如何与磁盘内存产生对应的呢？我们知道，在使用磁盘内存之前，需要为磁盘分区，然后为所分区域格式化出一个统一的文件系统（也有例外，如LVM与磁盘阵列技术）。那么，在这样一个统一的文件系统中，根据数据的不同，就可以将内存分为以下 3 种类型：

- inode：记录文件的属性，一个文件占用一个 inode，同时记录此文件的数据所在的 block 号码。当用户搜索或者访问一个文件时，UNIX 系统通过 inode 表查找正确的 inode 编号。在找到 inode 编号之后，相关的命令才可以访问该 inode，并对其进行适当的更改。

    > 例如使用vi来编辑一个文件。当您键入vi <filename> 时，在 inode 表中找到 inode 编号之后，才允许您打开该inode。在 vi 的编辑会话期间，更改了该inode中的某些属性，当您完成操作并键入 :wq 时，将关闭并释放该 inode。通过这种方式，如果两个用户试图对同一个文件进行编辑， inode 已经在第一个编辑会话期间分配给了另一个用户 ID (UID)，因此第二个编辑任务就必须等待，直到该 inode 释放为止。

<!-- more -->

- block：存储的文件内容，也叫数据区块(data block)，每个block都有自己的编号，Ext2支持的单位block容量仅为1k、2k、4k。如果文件太大，则会占用多个block。

- super block：记录文件系统的整体信息，包括inode/block的总量、使用量、剩余量，以及文件系统的格式与相关信息等。

因此对于一个文件来说，它的inode号就类似于id的作用，用于存放关于该文件的一些基本信息。

## 硬连接（hard link）

若两个文件，拥有同样的 inode，那么就产生了所谓的硬连接。

1、创建文件 a，并查看属性

```bash
$ touch a
$ ll a
-rw-r--r--  1 nobody  staff     0B  4 22 12:29 a
```

tips：如果不是很懂 Linux 中文档属性、拥有者、群组、权限、差异等，点击[传送门](https://www.cnblogs.com/justmine/p/9053419.html)。

2、创建 a 的硬连接 b，并查看 a 和 b 的属性

```bash
$ ln a b
$ ll a b
-rw-r--r--  2 nobody  staff     0B  4 22 12:29 a
-rw-r--r--  2 nobody  staff     0B  4 22 12:29 b
```

可以发现，a和b的属性完全一致，注意其中第二列数字表示的就是该文件的硬连接数。

3、对于硬连接来说，它的好处就是“安全”。你将其中任何一个文件名删除(rm -f)，文件都是存在的，因为文件的inode一直存在

```bash
$ rm -f a
$ ll b
-rw-r--r--  1 nobody  staff     0B  4 22 12:29 b
```

`并且由于两个文件都是对应的相同的 inode 和 block，因此对任何其中一个进行修改编辑，结果都会生效。`

4、如果此时重建 a，则新建的 a 与 b 没有任何关系，故存在硬连接关系

```bash
-rw-r--r--  1 nobody  staff     0B  4 22 12:40 a
-rw-r--r--  1 nobody  staff     0B  4 22 12:29 b
```

## 软连接（symbolic link）

软连接的意义就如同 windows 平台下的快捷方式，它是一个类型为 l 的文件，拥有自己的 inode 和 block，对它进行操作也相当于对被 link 文件进行操作。

`语法： ln -s source_file target_file`

1、创建 a 和 a 的软连接 b

```bash
$ touch a
$ ll a
-rw-r--r--  1 nobody  staff     0B  4 22 12:47 a
$ ln -s a b
$ ll a b
-rw-r--r--  1 nobody  staff     0B  4 22 12:47 a
lrwxr-xr-x  1 nobody  staff     1B  4 22 12:49 b -> a
```

2、为 a 写入数据，查看 b 中的内容。删除 a，再查看 b 的内容会发现 b 的连接失效了

```bash
$ cat > a <<EOF
heredoc> This is a test !
heredoc> EOF
$ cat b
This is a test !
$ rm -f a
$ cat b
cat: b: No such file or directory
```

3、重新创建文件 a，然后查看 b 的内容会发现b重新连接成功了，并且连接至了原来同名的a文件。

```bash
$ ll b
lrwxr-xr-x  1 nobody  staff     1B  4 22 12:49 b -> a
$ touch a
$ cat > a << EOF
heredoc> This is another test !
heredoc> EOF
$ cat b
This is another test !
```

## 总结

对于硬连接来说，共享的是 inode，重新创建的同名文件不具有相同的 inode，因此于原来毫无干系。而对于软连接来说，它连接记录的是一个文件的绝对路径，因此当重新创建同名文件时候，软连接又重新生效了。

## 相关链接

- [Linux 中硬连接（hard link）与软连接（symbolic link）的区别](https://blog.csdn.net/gxzc936733992/article/details/49340429)