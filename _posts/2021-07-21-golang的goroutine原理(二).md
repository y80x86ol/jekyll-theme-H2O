---
layout: post
title: 'golang的goroutine原理(二)'
date: 2021-07-21
author: ken
postPatterns: 'ticTacToe'
tags: goroutine
---

> golang的goroutine原理理解，使用简单的代码对goroutine协程运行进行展示

在上一篇我们了解了一下go的goroutine的大致实现原理，知道go为了实现高性能，采用了线程和协程的MPG模型，下面我们就通过具体的代码示例简单验证一下。

首先，我们给下面的程序只分配一个逻辑处理器，即相当于只开有一个线程

```
package main

import (
	"fmt"
	"runtime"
	"sync"
)

func main() {
	// 分配一个逻辑处理器给调度器使用
	runtime.GOMAXPROCS(1)

	// wg用来等待程序完成
	// 计数加2，表示要等待两个goroutine
	var wg sync.WaitGroup
	wg.Add(2)

	fmt.Println("开始执行goruntine\n")

	// goroutine运行打印大写字母3次
	go printUpperChar(&wg)

	// goroutine运行打印小写字母3次
	go printLowerChar(&wg)

	// 等待goroutine结束
	fmt.Println("等待goroutine执行结束\n")
	wg.Wait()

	fmt.Println("程序终止")
}

//printLowerChar 打印小写字母表3次
func printLowerChar(wg *sync.WaitGroup) {
	// 在函数退出时调用Done来通知main函数工作已经完成
	defer wg.Done()

	// 显示字母表3次
	for count := 0; count < 3; count++ {
		for char := 'a'; char < 'a'+26; char++ {
			fmt.Printf("%c ", char)
		}
	}

	fmt.Println("\n")
}

//printUpperChar 打印大写字母表3次
func printUpperChar(wg *sync.WaitGroup) {
	// 在函数退出时调用Done来通知main函数工作已经完成
	defer wg.Done()

	// 显示字母表3次
	for count := 0; count < 3; count++ {
		for char := 'A'; char < 'A'+26; char++ {
			fmt.Printf("%c ", char)
		}
	}

	fmt.Println("\n")
}

```

执行结果：
```
开始执行goruntine

等待goroutine执行结束

a b c d e f g h i j k l m n o p q r s t u v w x y z a b c d e f g h i j k l m n o p q r s t u v w x y z a b c d e f g h i j k l m n o p q r s t u v w x y z

A B C D E F G H I J K L M N O P Q R S T U V W X Y Z A B C D E F G H I J K L M N O P Q R S T U V W X Y Z A B C D E F G H I J K L M N O P Q R S T U V W X Y Z

程序终止

```

我们可以看到，执行结果是协程执行的两个方法，都是一个方法执行完了，才会有另外一个方法执行输出。无论我们执行多少次都是这样。

这是因为我们就只有一个调度器P，只会由一个线程用到一个CPU资源。

下面我们将调度器修改为2个，这个时候go会自动分配2个CPU资源。

```
// 其他没有变化，只是将runtime.GOMAXPROCS(2)中的1修改为了2
func main() {
	// 分配一个逻辑处理器给调度器使用
	runtime.GOMAXPROCS(2)

	// wg用来等待程序完成
	// 计数加2，表示要等待两个goroutine
	var wg sync.WaitGroup
	wg.Add(2)

	fmt.Println("开始执行goruntine\n")

	// goroutine运行打印大写字母3次
	go printUpperChar(&wg)

	// goroutine运行打印小写字母3次
	go printLowerChar(&wg)

	// 等待goroutine结束
	fmt.Println("等待goroutine执行结束\n")
	wg.Wait()

	fmt.Println("程序终止")
}
```

执行结果：
```
开始执行goruntine

等待goroutine执行结束

a b c d e f g A B C D E F G H I J K L M N O h i j k l m n o p q r s t u P Q R S v w x y z a b c d e f g h i j T U V W k l m n o p q r s t u v w x y z a b X Y Z A B C D E F G H I J K L M N O P Q R S T U V c d e f W X Y Z A B C D E F G H I J K L M N O P Q R S T U V W X Y Z

g h i j k l m n o p q r s t u v w x y z

程序终止

```

这次我们发现，输出不再是之前的一个方法执行完了接着执行另外一个方法，而是乱序的了，大小写夹杂着输出。着就是多个线程在同时执行，就是所谓了并行。

![并行](https://raw.githubusercontent.com/y80x86ol/img/main/2021/20210721141249.png)

不用说，并行的效率肯定会更高效。那我们有必要在每次都指定`runtime.GOMAXPROCS(1)`的个数吗？

答案是不用，因为go默认会启动4个逻辑处理器，但是不知道后续版本中是会根据CPU核数启动，还是指定的4个，这个需要查阅相关文档。

如果我们编写了一个应用程序，需要将性能最大化，可以进行一定的修改调整，然后进行性能测试。