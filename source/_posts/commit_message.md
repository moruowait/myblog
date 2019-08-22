---
title: 怎么写 Commit Message？
date: 2019-06-12 20:21:44
toc: true
tags:
- 技术名词
---

Commit Message 是告诉其他人你在提交中完成了哪些操作的最简单方法。通过阅读 Commit Message，您应该能够在不查看提交的情况下说出更改的范围和更改背后的一般思想。

## Commit Message

一条好的 Commit Message 应该包含以下部分。

### 标题

这是一个单行摘要，它应该帮助其他人快速浏览列表并找到他们想要的 commit。

这一行不应该超过 80 个字符，因为这是 Commit Message 中惟一保证在所有工具中都能看到的部分，而 Commit Message 的其余部分很可能是隐藏的。例如，“git log”、“git blame” 和 BitBucket commit 视图只显示这一行。

<!-- more -->

### 为什么进行这些更改

本节应详细说明正在进行的更改。它至少应该包括变更的范围、要解决的问题和变更的期望。

In practice,

- bullet
- points
- are
- preferred

### Metadata

Metadata 意味着被自动化系统读取(例如：Bitbucket、Bamboo 等)。虽然它们不打算被人们使用，但它们应该是可读的，并且易于输入。

例如，我们使用 `LABEL=value[，value]*` 来组织这些元数据。

最常用的标签是 `BUG= BUG-id`。

#### BUG

将这个 commit 和 JIRA 或者其他 BUG 追踪平台的 Issue 联系起来。

## 示例

```plain
xxxxxx: Implement PrintHelloWorld method

- add output for method `PrintHelloWorld`
- rename `hiWorld` to `helloWord`

BUG=XXXX-45
```
