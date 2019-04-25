---
title: 什么是闭包 ？
date: 2019-04-24 18:21:44
toc: true
tags:
- Go
- Javascript
- 技术名词
---

闭包（closure）是 Javascript 语言的一个难点，也是它的特色，很多高级应用都要依靠闭包实现。

## 变量作用域

要理解闭包，首先要理解 Javascript 特殊的变量作用域。

变量的作用域无非就是两种：全局变量和局部变量。

Javascript 语言的特殊之处，就在于函数内部可以直接读取全局变量。

```Javascript
var n = 999;
function f1(){
    alert(n);
}
f1(); // 999
```

<!-- more -->

另一方面，在函数外部自然无法读取函数内的局部变量。

```Javascript
function f1(){
    var n=999;
}
alert(n); // error
```

这里有一个地方需要注意，函数内部声明变量的时候，一定要使用 var 命令。如果不用的话，你实际上声明了一个全局变量！

```Javascript
function f1(){
　　n=999;
}
f1();
alert(n); // 999
```

## 如何从外部读取局部变量？

出于种种原因，我们有时候需要得到函数内的局部变量。但是，前面已经说过了，正常情况下，这是办不到的，只有通过变通方法才能实现。

那就是在函数的内部，再定义一个函数。

```Javascript
function f1(){
　　var n=999;
　　function f2(){
　　　　alert(n); // 999
　　}
}
```

在上面的代码中，函数 f2 就被包括在函数 f1 内部，这时 f1 内部的所有局部变量，对 f2 都是可见的。但是反过来就不行，f2 内部的局部变量，对 f1 就是不可见的。这就是 Javascript 语言特有的"链式作用域"结构（chain scope），子对象会一级一级地向上寻找所有父对象的变量。所以，父对象的所有变量，对子对象都是可见的，反之则不成立。

既然 f2 可以读取 f1 中的局部变量，那么只要把 f2 作为返回值，我们不就可以在 f1 外部读取它的内部变量了吗！

```Javascript
function f1(){
　　var n=999;
　　function f2(){
　　　　alert(n);
　　}
　　return f2;
}
var result=f1();
result(); // 999
```

## 闭包的概念

上一节代码中的 f2 函数，就是闭包。

各种专业文献上的"闭包"（closure）定义非常抽象，很难看懂。我的理解是，`闭包就是能够读取其他函数内部变量的函数`。

由于在 Javascript 语言中，只有函数内部的子函数才能读取局部变量，因此可以把闭包简单理解成"定义在一个函数内部的函数"。

所以，在本质上，闭包就是将函数内部和函数外部连接起来的一座桥梁。

## 闭包的用途

`闭包可以用在许多地方。它的最大用处有两个，一个是前面提到的可以读取函数内部的变量，另一个就是让这些变量的值始终保持在内存中`。

```Javascript
function f1(){
　　var n=999;
　　nAdd=function(){n+=1} // 没有用 var 声明，所以是全局函数

　　function f2(){
　　　　alert(n);
　　}
　　return f2;
}
var result=f1();
result(); // 999
nAdd();
result(); // 1000
```

在这段代码中，result 实际上就是闭包 f2 函数。它一共运行了两次，第一次的值是 999，第二次的值是 1000。这证明了，函数 f1 中的局部变量 n 一直保存在内存中，并没有在 f1 调用后被自动清除。

为什么会这样呢？原因就在于 f1 是 f2 的父函数，而 f2 被赋给了一个全局变量，这导致 f2 始终在内存中，而 f2 的存在依赖于 f1，因此 f1 也始终在内存中，不会在调用结束后，被垃圾回收机制（garbage collection）回收。`什么时候回收？只有当 f2 销毁的时候才会回收 f1 及 f1 中 定义的各种变量`

这段代码中另一个值得注意的地方，就是"nAdd=function(){n+=1}"这一行，首先在 nAdd 前面没有使用 var 关键字，因此 nAdd 是一个全局变量，而不是局部变量。其次，nAdd 的值是一个匿名函数（anonymous function），而这个匿名函数本身也是一个闭包，所以 nAdd 相当于是一个 setter，可以在函数外部对函数内部的局部变量进行操作。

## 使用闭包的注意点

- 由于闭包会使得函数中的变量都被保存在内存中，内存消耗很大，所以不能滥用闭包，否则会造成网页的性能问题，在 IE 中可能导致内存泄露。解决方法是，在退出函数之前，将不使用的局部变量全部删除。

- 闭包会在父函数外部，改变父函数内部变量的值。所以，如果你把父函数当作对象（object）使用，把闭包当作它的公用方法（Public Method），把内部变量当作它的私有属性（private value），这时一定要小心，不要随便改变父函数内部变量的值。

## Golang 并发中的闭包

Go语言的并发时，一定要处理好循环中的闭包引用的外部变量。如下代码：

```go
func main() {
    runtime.GOMAXPROCS(runtime.NumCPU())
    var wg sync.WaitGroup
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func() {
            fmt.Print(i)
            wg.Done()
        }()
    }
    wg.Wait()
}
// 输出结果 5 5 5 5 5
```

这种现象的原因在于闭包共享外部的变量 i，注意到，每次调用 go 就会启动一个 goroutine，这需要一定时间；但是，启动的 goroutine 与循环变量递增不是在同一个 goroutine，可以把 i 认为处于主 goroutine 中。启动一个 goroutine 的速度远小于循环执行的速度，所以即使是第一个 goroutine 刚起启动时，外层的循环也执行到了最后一步了。由于所有的 goroutine 共享 i，而且这个 i 会在最后一个使用它的 goroutine 结束后被销毁，所以最后的输出结果都是最后一步的 i==5。

我们可以使用循环的延时在验证上述说法：

```go
func main() {
    runtime.GOMAXPROCS(runtime.NumCPU())
    var wg sync.WaitGroup
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func() {
            fmt.Print(i)
            wg.Done()
        }()
        time.Sleep(1 * time.Second)   // 设置时间延时1秒
    }
    wg.Wait()
}
// 输出结果 4 0 1 2 3
```

每一步循环至少间隔一秒，而这一秒的时间足够启动一个 goroutine 了，因此这样可以输出正确的结果。

在实际的工程中，不可能进行延时，这样就没有并发的优势，一般采取下面两种方法：

### 共享的环境变量作为函数参数传递

```go
func main() {
    runtime.GOMAXPROCS(runtime.NumCPU())

    var wg sync.WaitGroup
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(i int) {
            fmt.Println(i)
            wg.Done()
        }(i)
    }
    wg.Wait()
}
// 输出结果 4 0 1 2 3
```

输出结果不一定按照顺序，这取决于每个 goroutine 的实际情况，但是最后的结果是不变的。可以理解为，函数参数的传递是瞬时的，而且是在一个 goroutine 执行之前就完成，所以此时执行的闭包存储了当前i的状态。

### 使用同名的变量保留当前的状态

```go
func main() {
    runtime.GOMAXPROCS(runtime.NumCPU())

    var wg sync.WaitGroup
    for i := 0; i < 5; i++ {
        wg.Add(1)
        i := i
        go func() {
            fmt.Println(i)
            wg.Done()
        }()
    }
    wg.Wait()
}
// 输出结果 4 0 1 2 3 (结果不一定与传参的方式一致)
```

同名的变量 i 作为内部的局部变量，覆盖了原来循环中的 i，此时闭包中的变量不再是共享外循环的 i，而是都有各自的内部同名变量 i，赋值过程发生于循环 goroutine，因此保证了独立。

## 相关链接

- [阮一峰：学习Javascript闭包（Closure）](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html?20120612141317)
- [Golang 中闭包的理解](https://blog.csdn.net/qq_35976351/article/details/81986496)