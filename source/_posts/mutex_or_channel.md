---
title: 加锁 Mutex 和 Channel 性能对比
date: 2019-04-19 14:38:40
toc: true
tags:
- Go
---

## 性能对比

首先在一个目录下新建两个文件：

main.go

```go
// main.go
package main

import "sync"

var mutex = sync.Mutex{}
var ch = make(chan bool, 1)

func UseMutex() {
    mutex.Lock()
    mutex.Unlock()
}

func UseChan() {
    ch <- true
    <-ch
}
```

<!-- more -->

main_test.go

```go
// main_test.go
package main

import "testing"

func BenchmarkUseMutex(b *testing.B) {
    for n := 0; n < b.N; n++ {
        UseMutex()
    }
}

func BenchmarkUseChan(b *testing.B) {
    for n := 0; n < b.N; n++ {
        UseChan()
    }
}
```

然后在该文件路径下执行 benchmark：

```bash
go test -bench=.
```

结果如下：

```bash
goos: darwin
goarch: amd64
pkg: testproject/race
BenchmarkUseMutex-4     100000000               15.9 ns/op
BenchmarkUseChan-4      30000000                50.1 ns/op
PASS
ok      testproject/race        3.173s
```

从压测结果来看，加锁的方式是使用 Channel 方式的 3.1 倍

## 原因分析

* channel 的成本高于 Mutex

    1. channel 内部有 Mutex，是通过共享内存实现的。(TODO:这里少一个传送门)
    2. channel 内部可能有 Cond，用来等待或唤醒满足条件的 goroutine(TODO:这里少一个传送门)
    3. 出让 cpu 并且让另一个 goroutine 获得执行机会，这个切换周期不低，远高于 Mutex 检查竞争状态的成本（后者通常只是一个原子操作）

## 相关链接

* [加锁 Mutex 和 Channel 性能对比](https://www.colabug.com/278134.html)
* [Golang并发：再也不愁选channel还是选锁](http://lessisbetter.site/2019/01/14/golang-channel-and-mutex/)
