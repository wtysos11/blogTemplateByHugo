---
title: "golang基础问题：X does not implement Y (... method has a point receiver)"
date: 2021-06-12T10:38:14+08:00
tags: ["Go","问题解答"]
categories: ["Go基础学习"]
---

## 遇到问题
这个问题的来源是今天在复习接口的时候遇到了一份代码
```go
package main

import "fmt"

type RPCError struct {
	Code    int64
	Message string
}

func (e *RPCError) Error() string {
	return fmt.Sprintf("%s, code=%d", e.Message, e.Code)
}

func main() {
	fmt.Println("In main function")
	var rpcErr error = NewRPCError(400, "unknown err") // typecheck1
	err := AsErr(rpcErr) // typecheck2
	println(err)
}

func NewRPCError(code int64, msg string) error {
	return RPCError{ // typecheck3， must be &RPCError
		Code:    code,
		Message: msg,
	}
}

func AsErr(err error) error {
	return err
}
```
这份代码有一个compile-time error，在func NewRPCError的返回值处会报错误
```
./main.go:22:17: cannot use RPCError{...} (type RPCError) as type error in return argument:
	RPCError does not implement error (Error method has pointer receiver)
```
解决方案是将返回的`RPCError`改为`&RPCError`，即传回指针而不是实值。

但是这显然是与常识相违背的，因为error类型本身也并不是什么指针之类的东西，那这个指针是定义在哪里的？
## 找到共性
在[SO](https://stackoverflow.com/questions/40823315/x-does-not-implement-y-method-has-a-pointer-receiver)上找到了一个对应的问题，回答中构造了一个类似的情况
```go
package main

import "fmt"

type Stringer interface {
	String() string
}

type MyType struct {
	value string
}

func (m *MyType) String() string { return m.value }

func main(){
	m := MyType{value:"something"}
	var s Stringer
	s = m // s=&m
	fmt.Println(s)
}
```
大意就是，为了实现接口Stringer，结构体必须有一个方法String。而在实际的实现中，String是被绑在方法\*MyType上而不是MyType，所以接口对应的类型也只能是\*MyType，因此必须传递一个指针。

## 回溯
而在原问题中，对Error方法的实现为
```go
func (e *RPCError) Error() string {
	return fmt.Sprintf("%s, code=%d", e.Message, e.Code)
}
```
所以如果将这个地方进行修改的话一样是可行的。
问题解决。

## 根源
该问题的根源在于，结构体与结构体的指针本质上是两种不同的类型。就像不能向一个接受指针的函数传递结构体一样，反过来也是不行的。只是这两种类型可以比较方便地进行快速转换，所以可能导致一些理解上的偏差

比如下面两种实现接口的方法，其实都是可以的，而且是两种不同的实现方法。（当然，下面这样写肯定是不行的，指针类型和非指针类型对接口的实现不能同时出现）
```
type Duck interface{
    Walk()
    Quack()
}
type Cat struct{}
func (c *Cat) Walk(){
    fmt.Println("Star Cat Walk")
}
func (c *Cat) Quack(){
    fmt.Println("Star Cat meow")
}
func (c Cat) Walk(){
    fmt.Println("Cat Walk")
}
func (c Cat) Quack(){
    fmt.Println("Cat meow")
}
```
详情可以看[面向信仰编程-4.2接口](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-interface/)