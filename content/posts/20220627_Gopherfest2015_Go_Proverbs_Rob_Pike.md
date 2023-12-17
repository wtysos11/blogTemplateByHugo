---
title: Gopherfest 2015：Go Proverbs with Rob Pike
date: 2022-06-27T21:13:15+08:00
tags:
  - 待完善
  - 计算机/go
  - 学习/学习笔记
categories:
  - 计算机视频学习笔记
summary: 阅读关于布隆过滤器的综述文章，该论文通过多个方面分析了现有的布隆过滤器及其变体的实现与性能
---
# Gopherfest 2015：Go Proverbs with Rob Pike


* [原视频](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=568s)


Proverb：最早是围棋中的概念

一些Proverb
* Don't communicate by sharing memory, share memory by communicating.
* Concurrency is not parallelism.
  * Talk in Waza.
  *  Concurrency is a way of structuring your program to make it easier to understand and scalable.
  *  Parallelism is simply the execution of multiple go routines in parallel.
*  Channels orchestrate; mutexes serialize.
   *  Beginner's question: when to use mutex, cond ... all these stuffs
   *  Mutex: fine-grained, serialize execution. Only allow one thing to go.
   *  go routine and channel give you a way to structure the program, arrange how the pieces work.
*  The bigger the interface, the weaker the abstraction.
   *  与java的接口风格不同。例子：空接口，io.Reader和io.Writer，平均方法数量为2/3，但是非常实用
   *  个人的其他思考：接口最好不要作为暴露给其他包的第一层，特别当包的逻辑是复杂逻辑时（比如算法）。
      *  一个是之前用的时候Go的编译器还不支持，跳转起来很麻烦。简单的包还好说，复杂的包就比较头大了。
      *  而且在具体使用这个包的时候，由于暴露的是接口，很难第一时间对应到具体的实现。
*  Make the zero value useful.
   *  例子：bytes buffer/ sync.Mutex可以直接被放在结构体中，不需要任何的初始化。
   *  bytes buffer认为zero value is a valid buffer ready to use.
   *  A way to sturcutre your data to make sure that the programs are much easier to use.
      *  ==> less api, always good thing.
*  interface{} says nothing
   *  interface{}在接口作为参数的情况下非常好用。但是interface{}方法如果被作为参数传入给其他函数，可能会执行一些方法，并导致异常的退出。而且很难检查出来，因此被称为says nothing.
   *  因此，如果要将interface{}作为参数的时候，必须要有足够的原因与对应的检查。【不要滥用！】
*  Gofmt's style is no one's favorite, yet gofmt is everyone's favorite.
   *  这个我倒是比较认同的，我记得rust也有formatter，vscode的python插件中我自己也经常使用Formatter。
   *  I don't like how it formats, but I like it formats.
*  A little copying is better than a little dependency.
   *  保持依赖树的简单
   *  如果需要的并不是整行代码，而是其中的几行，那就应该直接拷贝。
      *  但如果需要同步修改则是另外一回事，比如频繁更新的业务代码即使只有几行，也应该引用更新频率不高的基础代码。因为一般人肯定不想再基础代码更新的时候从浩如烟海的业务代码中找出引用这几行的部分并进行更新。
   *  Example: strings和unicode都有IsPrint函数，其中strings库中的这个函数是对Unicode的简单实现（而并没有引用）
   *  Don't be afraid to copy when it makes sense.
   *  这个也是我看这个视频的主要原因。可以用一个问题来概述：什么时候应该引用？
      *  比如说，如果有一个redis client在其他的库中【生产环境】，当前库的逻辑与该库的逻辑基本一致，是否应该引用这个client？
*  Syscall must always be guarded with build tags.
   *  syscall isn't protable, it's system specific package
   *  如果代码中引用了syscall，需要确定保持版本一致，确保对每个不同的system和architecture都有对应的实现和测试，确保有效。否则，就应该使用os包或者其他类似的库。
*  Cgo must always be guarded with build tags.
   *  C is not portable. 
   *  同样的原因
*  Cgo is not Go.
   *  Cgo很可能会导致runtime error，非常的危险
   *  垃圾回收等有可能会失效，或者说Go的所有对于类型和指针的约束都可能会被破坏
*  With the unsafe package there are no guarantees.
   *  之前版本使用unsafe包的代码在之后包可能会变得不再泛用
   *  尽量不要在长期运行的程序中使用unsafe（Don't use unsafe unless you're prepared to have your program break one day）
*  Clear is better than clever.
   *  Go的风格，更多为可读性、可维护性等妥协
*  Reflection is never clear
   *  只有极少数人需要使用，it's a very powerful but very very difficult to use feature.
*  Errors are values.
*  Don't just check errors, handle them gracefully
   *  这个比较有感触。前段时间刚碰上一个bug，原因就是有一个地方直接把报的错误给截断了，导致debug的时候很难定位到问题。
*  Design the architecture, name the components, document the details.
   *  当设计大型系统的时候，命名是非常重要的。命名需要能区分不同的部分，并传递其中的意义。
*  Documentation is for users.
   *  这个也是老问题了，文档不是写给自己的，是写给不懂的人。
   *  Go文档的另外一个特点是轻量级（没有annotation）
*  

一些其他的思想：
* KISS：Keep it simple and stupid
* DRY：Don't repeat yourself

感想：Rob Pike究竟对java有多大的怨念