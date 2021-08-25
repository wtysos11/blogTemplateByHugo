---
title: "golang中的赋值:=的一次误用与shadow variable"
date: 2021-07-08T11:32:14+08:00
tags: ["Go"]
categories: ["事件记录"]
---

今天遇到了一个问题，我需要设置一个循环中的变量用于下一次循环之中。因为修改的时候一些问题（处理err），所以不小心将之前的`=`改为了`:=`。大致如下
```go

func getEle() (int,error){
	return rand.Intn(100),nil
}


func exchangeGradually(fp int){
	var fingerprint int = fp
	for i:=0;i<5;i++{
		oldfingerprint := fingerprint
		fingerprint,err := getEle()
		if err != nil{
			panic(err)
		}
		fmt.Println(fingerprint,oldfingerprint)

	}
}

func main() {
	exchangeGradually(10)
}
```

可以试着执行一下，原本预期中oldfingerprint是随着fingerprint改变而改变的，但是实际上是不变的。原因是很简单的，因为`:=`重新分配了一个变量覆盖掉了原有的变量。

但是我原本以为是不会覆盖的，因为之前写错误处理的时候往往也是直接`val,err := ...`这样写下来。现在看来，`:=`应该是会重新声明左侧的所有变量并覆盖作用域。

从这里可以引出shadow error的问题，类似于[shadow variable](https://magodo.github.io/golang-shadowing/)。shadow error是指很多时候需要在defer中处理error，但是被后面的错误给覆盖了，类似于
```go
func getErr1() (int,error){
	return 1,fmt.Errorf("error 1")
}

func getErr2() (int,error){
	return 2,fmt.Errorf("error 2")
}


func exchangeGradually(fp int){
	a1,err := getErr1()
	fmt.Println(a1,err)
	defer func() {
		fmt.Println(err)
	}()
	a2,err := getErr2()
	fmt.Println(a2,err)
}

func main() {
	exchangeGradually(10)
}

```
在下面这个例子中，原本预期要处理的是error1，但是实际输出的却是error2。因为defer针对的err是函数作用域的，该变量被后续的新声明给覆盖了（当然，实际上原理是不一样的，这个主要是defer中传值与传引用的问题，只要加上捕获列表即可）。
```go
func exchangeGradually(fp int){
	a1,err := getErr1()
	fmt.Println(a1,err)
	defer func(err error) {
		fmt.Println(err)
	}(err)
	a2,err := getErr2()
	fmt.Println(a2,err)
}
```