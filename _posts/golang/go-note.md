---
layout: post
title:  "GO学习笔记"
date:   2015-10-31 13:00
categories: golang
---

## defer

1. defer

	defer语句会延迟函数的执行直到上层函数返回。延迟函数的参数会立刻生成，但是在上层函数返回前函数不会被调用。

	```
	func myDefer(){
	    defer fmt.Println("world")
	    fmt.Println("hello")
	    fmt.Println("  ")
	}
	```

2. defer栈

	延迟函数调用被压入一个栈中。当函数返回时，会按照后进先出的顺序调用被延迟的函数调用。

	```
	func myDeferStatck(){
	    fmt.Println("counting")
	    for i:=0;i<10;i++{
	        defer fmt.Println(i)
	    }

	    fmt.Println("Done")
	}
	```

	输出：
	counting
	Done
	9
	8
	7
	6
	5
	4
	3
	2
	1
	0

##slice
1. slice

	slice可以重新切片，创建一个新的slice值指向相同的数组
	表达式
	s[lo:hi]
	表示从lo到hi-1的slice元素，含前端，不包含后端。因此
	s[lo:lo]是空的，而
	s[lo:lo+1]有一个元素。

2. 构造slice

	slice由make函数创建。会分配一个全是零值的数组并且返回一个slice指向这个数组
	a:=make([]int,5) //len(a)=5
	为了指定容量，可传递第三个参数到make:

	```
	b:=make([]int,0,5)  //len(b)=0,cap(b)=5
	b=b[:cap(b)] //len(b)=5,cap(b)=5
	b=b[1:] //len(b)=4,cap(b)=4
	```

##goroutine
1. goroutine

	goroutine是由go运行时环境管理的轻量级线程
	goroutine在相同的地址空间中运行，因此访问共享内存必须进行同步。sync提供了这种可能，不过在go中并不经常用到。

2. channel

	channel是有类型的管道，可以用channel操作符<-对其发送或者接收值。

	```
	ch <- v //将v送入channel ch
	v:=<-channel 	//从ch接收，并且赋值给v
	```

	和map和slice一样，channel使用前必须创建

	```
	ch:=make(chan int)
	```

	默认情况下，在另一端转备好之前，发送和接收都会阻塞，这使得goroutine可以在没有明确的锁或者竞态变量的情况下进行同步。

3. 缓冲channel

	channel可以是带缓冲的。为make提供第二个参数作为缓冲长度来初始化一个缓冲channel。

	```
	ch:=make(chan int,100)
	```

	向channel发送数据时，只有在缓冲区满的时候才会阻塞。当缓冲区清空的时候接受阻塞。

4. range 和close

	发送者可以close一个channel来表示没有值会被发送了。
	接收者可以通过赋值语句的第二个参数来测试channel是否被关闭，当没有值可以接受并且channel已经被关闭，那么经过v,ok:=<-ch之后ok会被设置为false
	循环 for i:=range c 会不断从channel接收值，直到它被关闭。

	* 只有发送者才能关闭channel，而不是接收者。向一个已经关闭的channel发送数据会引起panic。
	* channel与文件不同，通常无需关闭。只有在需要告诉接收者没有更多的数据的时候才有必要进行关闭。例如终端一个range。