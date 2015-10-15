title: 使用golang模拟MapReduce过程
date: 2015-10-06 22:10:00
tags: ["golang","mapreduce","并行计算"]
---
MapReduce是并行计算中非常重要的一部分，本文为MapReduce扫盲贴，所谓的MapReduce主要分为Map和Reduce两部分内容，也就是先进行任务拆分给多个CPU进行计算然后再将计算结果进行归约...
<!-- more -->
#### 并发与并行（Concurrency and Parallelism）
在开始之前，我们先普及一下并发和并行的概念。并发和并行是两个截然不同的概念，他们之间的区别要讲起来是比较晦涩难懂的，网上看的很多解释都使用逻辑控制流来进行解释，有的人反而越看越不明白。以下引用[Rob Pike的演讲中的一小段进行解释](http://blog.golang.org/concurrency-is-not-parallelism)上的这样一句话：
> Concurrency is about dealing with lots of things at once. Parallelism is about doing lots of things at once.   
> Concurrency is goal for structure, Parallelism is goal for execute   

极端一点，简单来说就是“并发”指的是同时处理很多事情（可以是相同的或不同的事情），“并行”指的是同时做大量的工作（强调的是量，通常情况下是做相同的工作，当然从某种情况下来讲也可以是不相同的）。

#### MapReduce思想
MapReduce核心思想就是将一个任务分成多份，并行的去执行，执行完成后再把所有结果统一起来。所以它分为两个主要过程  
- 任务拆分，就是Map过程
- 结果整合，就是Reduce过程

#### 使用golang的goroutine进行模拟
MapReduce的目的就是为了让计算变得更快，我们这里使用经典的求和任务来进行模拟   
- 任务：计算 1 + 2 + 3 + ... + 100000 的结果
- 传统的实现方法--一次循环搞定（以下代码都使用golang语法演示）   
```
func add1(start, end int) int {
	sum := 0
	for i := start; i <= end; i++ {
		sum += i
	}
	return sum
}
```

- 哈哈哈，当然还有更聪明的办法
```
func add2(start, end int) int{
	return (end - start + 1) * (end + start) / 2
}
```

- 前面介绍的两种方法，写过代码的都知道，下面使用goroutine与channel进行模拟
```
func add3(start, end int) int{
	rout_num := 4  // 定义使用4个goroutine进行计算
	ch := make(chan int)  // 定义一个channel用来传递每个goroutine计算结束后的结果

	// Map操作，将计算任务分配下去
	// 第一个goroutine计算 1 + 5 + 9 + ... 
	// 第二个goroutine计算 2 + 6 + 10 + ...
	// ...
	for i := 0; i < rout_num; i++ {
		go func(start, end, step int) {
			sum := 0
			for i := start; i <= end; i += step {
				sum += i
			}
			ch <- sum  // goroutine计算完成后将结果送给channel等待主程序接收
		}(start + i, end, rout_num)
	}

	// Reduce操作
	// 从channel中取出各个goutine计算的结果
	// 并将各个goroutine的计算结果加和得到最终结果
	sum := 0
	for i := 0; i < rout_num; i++ {
		sum += <-ch
	}

	return sum
}
```

#### 最后说明
虽然使用goroutine进行模拟，但golang中goroutine的设计初衷还是用来实现更高的并发，本文重点还请放在MapReduce思想上，因为也是菜鸟一枚，欢迎各位大牛留言交流。



