---
title: Golang slice 之 append 时原数组发生变化
date: 2019-05-28 19:22:34
toc: true
tags:
- Go
---

## 背景

使用 append 可以在 slice 之后追加元素，例如

```go
a := []int{1,2,3}
result := append(a,4)
fmt.Println(result) // output: [1 2 3 4]
```

问题在于，进行这种操作时，原来的 slice（即 a）所基于的数组值会不会发生变化？在 Golang 中，如果有多个 slice 基于同一个数组，则这些 slice 的数据是共享的（而不是每个 slice 复制一份，复制的是指针）。也就说，如果改变了数组的内容，则基于它的所有 slice 的值都会发生变化。这段代码中 a 的值没有发生变化，是有原因的。

<!-- more -->

## 分析

回答这个问题，首先需要了解 append 函数的实现原理：

- 如果 a 的 cap 够用，则会直接在 a 指向的数组后面追加元素，返回的 slice 和原来的 slice 是同一个对象。显然，这种情况下原来的 slice 值发生了变化！
- 如果 a 的 cap 不够用（上述代码就是这种情况），则会重新分配一个数组空间来存储数据，并且返回指向新数组的 slice。这时候原来的 a 指向的数组并没有发生任何变化！（后面讲给出例子）
- 当然，在任何情况下，append 返回的结果都是追加后的 slice，这一点没有问题。

以下代码用来验证这个问题：

1. 在函数 `test1` 中 a 的值发生变化了，因为 a[:2] 的 len=2，cap=3，所以追加一个元素时，cap 依然够用
2. 在函数 `test2` 中 a 的值没有发生变化，因为 a[:2] 的 cap 不够用，因此会重新分配一个数组用来存储新的数据，而 a 存储的仍然是老数组

```go
func test1() {
    a := []int{1, 2, 3}
    fmt.Printf("&a:[%p], a:[%p], &a[0]:[%p], a:%v, len(a): %d, cap(a): %d \n", &a, a, &a[0], a, len(a), cap(a))
    // &a:[0xc0000be0e0], a:[0xc0000a0140], &a[0]:[0xc0000a0140], a:[1 2 3], len(a): 3, cap(a): 3
    a = append(a[:2], 4)
    fmt.Printf("&a:[%p], a:[%p], &a[0]:[%p], a:%v, len(a): %d, cap(a): %d \n", &a, a, &a[0], a, len(a), cap(a))
    // &a:[0xc0000be0e0], a:[0xc0000a0140], &a[0]:[0xc0000a0140], a:[1 2 4], len(a): 3, cap(a): 3

    // 指向 a 的指针没有发生变化，数组 a 的地址没有发生变化，是因为 cap 够用，不需要扩容
}
```

```go
func test2() {
    a := []int{1, 2, 3}
    fmt.Printf("&a:[%p], a:[%p], &a[0]:[%p], a:%v, len(a): %d, cap(a): %d \n", &a, a, &a[0], a, len(a), cap(a))
    // &a:[0xc0000b80e0], a:[0xc00009a140], &a[0]:[0xc00009a140], a:[1 2 3], len(a): 3, cap(a): 3
    c := append(a[:2], []int{4, 5, 6}...)
    fmt.Printf("&a:[%p], a:[%p], &a[0]:[%p], a:%v, len(a): %d, cap(a): %d \n", &a, a, &a[0], a, len(a), cap(a))
    // &a:[0xc0000b80e0], a:[0xc00009a140], &a[0]:[0xc00009a140], a:[1 2 3], len(a): 3, cap(a): 3
    fmt.Printf("&c:[%p], c:[%p], &c[0]:[%p], c:%v, len(c): %d, cap(c): %d \n", &c, c, &c[0], c, len(c), cap(c))
    // &c:[0xc0000b8140], c:[0xc0000f8000], &c[0]:[0xc0000f8000], c:[1 2 4 5 6], len(c): 5, cap(c): 6

    // a 没有发生改变，因为 cap 不够用，发生了扩容，重新分配了一个数组来 append 元素，原有的 a 仍然执行原数组，而 append 返回了新分配的数组
}
```

- 为了避开这个“坑“，推荐在截取 slice 时使用三参数的方式：

```go
func main() {
    a := []int{1, 2, 3, 4, 5}
    fmt.Printf("len(a): %d, cap(a): %d \n", len(a), cap(a)) // len(a): 5, cap(a): 5

    b := a[:3]
    fmt.Printf("len(b): %d, cap(b): %d \n", len(b), cap(b)) // len(b): 3, cap(b): 5

    c := a[:3:3]
    fmt.Printf("len(c): %d, cap(c): %d \n", len(c), cap(c)) // len(c): 3, cap(c): 3
}
```

从输出结果可以看出，使用三参数方式时，截取后的 slice 的 cap 将会重新设置。如果第二个参数和第三个参数相同，那么截取后的 slice 的 len == cap，这样在执行 append 的时候一定会重新分配数组，从而保证原始的数组 a 不会发生改变。

PS: 三参数表达式：a[start : end : cap]

- 同样的分析一下数组作为参数传递所发生的事情：

```go
func main() {
    var s = []string{"1", "2", "3"}

    fmt.Printf("&s:[%p], s:[%p], &s[0]:[%p], s:%v, len(a): %d, cap(a): %d \n", &s, s, &s[0], s, len(s), cap(s))
    // &s:[0xc0000ba0e0], s:[0xc00008e450], &s[0]:[0xc00008e450], s:[1 2 3], len(a): 3, cap(a): 3
    // here we call t as s[:2]
    fmt.Printf("&s:[%p], t:[%p], &t[0]:[%p], t:%v, len(s): %d, cap(s): %d \n", &s, s[:2], &s[:2][0], s[:2], len(s[:2]), cap(s[:2]))
    // &s:[0xc0000ba0e0], t:[0xc00008e450], &t[0]:[0xc00008e450], t:[1 2], len(s): 2, cap(s): 3
    test(s[:2]...)
    fmt.Printf("&s:[%p], s:[%p], &s[0]:[%p], s:%v, len(a): %d, cap(a): %d \n", &s, s, &s[0], s, len(s), cap(s))
    // &s:[0xc0000ba0e0], s:[0xc00008e450], &s[0]:[0xc00008e450], s:[1 2 3], len(a): 3, cap(a): 3
}

func test(a ...string) {
    fmt.Printf("&a:[%p], a:[%p], &a[0]:[%p], a:%v, len(a): %d, cap(a): %d \n", &a, a, &a[0], a, len(a), cap(a)) // 说明复制了一个指针，指向了数组
    // &a:[0xc0000fa000], a:[0xc00008e450], &a[0]:[0xc00008e450], a:[1 2], len(a): 2, cap(a): 3
    // here we call t as a[:len(a):len(a)]
    fmt.Printf("&a:[%p], t:[%p], &t[0]:[%p], t:%v, len(a): %d, cap(a): %d \n", &a, a[:len(a):len(a)], &a[:len(a):len(a)][0], a[:len(a):len(a)], len(a[:len(a):len(a)]), cap(a[:len(a):len(a)]))
    // &a:[0xc0000fa000], t:[0xc00008e450], &t[0]:[0xc00008e450], t:[1 2], len(a): 2, cap(a): 2
    a = append(a[:len(a):len(a)], "4")
    fmt.Printf("&a:[%p], a:[%p], &a[0]:[%p], a:%v, len(a): %d, cap(a): %d \n", &a, a, &a[0], a, len(a), cap(a))
    // 数组第一个元素的指针发生了改变，指向数组的指针没有改变，说明数组扩容重新分配了数组地址
    // &a:[0xc0000fa000], a:[0xc000100000], &a[0]:[0xc000100000], a:[1 2 4], len(a): 3, cap(a): 4
}
```
