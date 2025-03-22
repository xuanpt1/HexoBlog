---
title: go-interface详解
date: 2025-03-22 02:43:31
tags:
---

# Go interface详解及源码阅读

~~太晚了睡了先，接着写下明天计划~~

~~详细翻看interface相关，相比其与之前用的Java有何特色、异同？~~

~~看源码，详细理解分享 append、make等~~

~~**Go基本架构**这个之前没注意看过，得看看；Java还了解一些，Go确实不熟~~~~

お休み

## interface定义

Go中并没有显式的继承，大致都是通过匿名字段等实现的

```go
type human struct{
    int age
}
type man struct{
    human
    //直接将父struct写在子struct前，这就是隐式的继承
    //相比之下 Java中使用了extend关键字
    int height
}

man := new(man)
man.age = 23
//可直接调用修改man的继承来的age属性
man.height = 180
fmt.Println(man.age)
fmt.Println(man.height)

//output： 23
//			180
```

同样的，Go中没有明确支持多态，但可以通过一些手段实现类似多态的效果

**interface**是Go中最重要的特性之一，**interface**类型可以定义**一组方法**，而这些不需要实现（Java亦然

> tips：此处限定是**一组方法**，不能是变量。**一组**表明可以有多个方法。interface本质上也是一种类型，确切地说，是**指针类型**。

interface是为实现多态功能，如果某一类型实现了某个接口，所有使用这个接口的地方，都可以支持这种类型的值。

```go
type name interface{
    func1(arg...) returns...
    func2(arg...) returns...
    ...
}
```

接口通常以er作为后缀（这个好理解，这组方法是用来干嘛的，这个接口就是什么er），方法名是生命组成部分，但参数名可不同或省略。如果接口没有任何方法声明，那它就是一个空接口(`interface{}`),用途类似面向对象语言中的根类型Object，可被赋值为任何类型对象。接口变量默认值为nil。如果实现接口的类型支持，可做相等运算。

```go
var t1, t2 interface{}
fmt.Println(t1 == nil, t1 == t2)

t1, t2 = 100, 100
fmt.Println(t1 == t2)

t1, t2 = map[string]int{}, map[string]int{}
fmt.Println(t1 == t2)

//output: true true
//			true
//			panic: runtime error: comparing uncomparable type map[string]int
```



此外，接口也可以同匿名字段那样。嵌入其他接口。目标类型方法集中必须拥有把汗嵌入接口方法在内的全部方法才算实现了该接口。

```go
//测试接口及struct定义
type base interface {
	base()
}

type stringer interface {
	string() string
}

type tester interface {
	base
	stringer
    // 接口中嵌入其他接口
	test()
}

type data struct {
	str string
}

func (data) test() {
	fmt.Println("func test run")
}

func (data) base() {
	fmt.Println("func base run")
}

func (d data) string() string {
	return d.str
}
```

然后我们测试上述接口的实现

```go
data := data{
    str: "hello world",
}
var t tester = &data
t.test()
t.base()
fmt.Println(t.string())
```

得到预期输出

```
func test run
func base run
hello world
```



## interface使用场景

### 类型转换

**类型推断**可将接口变量还原为原始类型，或用来判断是否实现了某个更具体的接口类型

仍旧使用上文定义的struct及interface

```go
d := data{
		str: "hello world",
	}
	var x interface{} = d

	if s, ok := x.(stringer); ok {
		fmt.Println(s.string())
	}

	if d2, ok := x.(data); ok {
		d2.test()
	}

	e := x.(error)
	fmt.Println(e)
```

得到预期输出

```
hello world
func test run
panic: interface conversion: main.data is not error: missing method Error
```



### 实现多态

```go
type notifier interface {
	notify()
}
type user struct {
	name  string
	email string
}

func (u *user) notify() {
	fmt.Printf("Sending user email to %s<%s>\n", u.name, u.email)
}

type admin struct {
	name  string
	email string
}

func (a *admin) notify() {
	fmt.Printf("Sending admin email to %s<%s>\n", a.name, a.email)
}
func sendNotification(n notifier) {
	n.notify()
}
```

测试

```go
	u := user{"user", "user@email.com"}
	a := admin{"admin", "admin@email.com"}
	sendNotification(&u)
	sendNotification(&a)
```

输出：

```
Sending user email to user<user@email.com>
Sending admin email to admin<admin@email.com>
```

上述代码中，`sendNotification`函数接受一个实现了`notifier`接口的值作为参数。

既然我们可以为任何实体类型实现该接口，那么这个函数也就可以针对任何实体类型值来执行`notify`方法，在调用`notify`方法时，根据对象的实际impl来执行可能不同的行为，从而实现多态。



### 解耦合及提高代码复用性

> 这里和Java的逻辑基本一致

依旧是上述例子，`sendNotification`函数只依赖底层`notifier`接口，而不依赖具体实现

如果要新增其他需要send的对象，只需为其实现`notifier`接口即可



## interface底层实现

首先，GO语言的interface底层实现主要设计两个struct，`iface`和`eface`

源码在`/src/runtime/runtime2.go`中

```go
type iface struct {
    tab  *itab
    data unsafe.Pointer
}

type eface struct {
    _type *_type
    data  unsafe.Pointer
}
```

**iface：**用于表示包含方法的接口。它包含两个指针，一个指向具体类型信息的`itab`，另一个指向实际的数据

**eface：**用于表示空接口（不包含任何方法的接口）。它也包含两个指针，一个指向具体类型信息，另一个指向实际的数据

> 空接口在实际使用中很常见，所以实现时GO使用了单独的类型

从结构体可以看出，其中只包含*类型指针*和*数据指针*，所以任何类型都可以转化为空接口，即**空接口能表示任何类型**

我们可以更深一步看向`itab`

源码在`src/internal/abi/iface.go`中

```go
// The first word of every non-empty interface type contains an *ITab.
// It records the underlying concrete type (Type), the interface type it
// is implementing (Inter), and some ancillary information.
//
// allocated in non-garbage-collected memory
type ITab struct {
	Inter *InterfaceType
	Type  *Type
	Hash  uint32     // copy of Type.Hash. Used for type switches.
	Fun   [1]uintptr // variable sized. fun[0]==0 means Type does not implement Inter.
}

// EmptyInterface describes the layout of a "interface{}" or a "any."
// These are represented differently than non-empty interface, as the first
// word always points to an abi.Type.
type EmptyInterface struct {
	Type *Type
	Data unsafe.Pointer
}
```

每个非空接口类型的第一个字（通常指内存中的第一个地址单元）包含一个指向 `ITab` 结构体的指针。这意味着在内存中，非空接口类型的起始部分存储的是这个 `ITab` 结构体的地址。

`ITab` 结构体记录了三个重要信息：

- `Inter *InterfaceType`：指向 InterfaceType 结构体的指针，该结构体描述了接口的类型信息。
- `Type *Type`：指向 Type 结构体的指针，该结构体描述了具体类型的信息。
- `Hash uint32`：Type.Hash 的副本，用于类型切换（type switches）。在进行类型切换时，可以使用这个哈希值来快速判断类型是否匹配。
- `Fun [1]uintptr`：一个可变大小的数组，用于存储具体类型实现接口方法的函数指针。fun[0] == 0 表示具体类型没有实现该接口。

> 可以发现，在这个`abi`包下也有个`EmptyInterface`存在，这不是和`eface`重复了吗？
>
> 它们本质上描述的是同一概念，也就是空接口（`interface{}`或者`any`）的底层表示形式
>
> `eface`在`runtime`包中定义，其作用为在运行时（runtime）层面处理空接口
>
> `eface` 的设计是为了**满足运行时系统的高效处理需求**
>
> `EmptyInterface`定义于`internal/abi`包，它主要用于描述Go语言的应用二进制接口（ABI）。
>
> `internal/abi` 包专注于定义 Go 语言的 ABI，也就是不同编译单元之间的接口规范。`EmptyInterface` 的定义是为了**确保不同编译单元在处理空接口时能够保持一致**



## interface在做类型转换时会发生什么？

测试代码：

```go
	var user1 *user
	user2 := user1
	var n notifier = user2
	if n == nil {
		fmt.Println("n is nil")
	} else {
		fmt.Println("n is not nil")
	}
```

输出：

```
n is not nil
```

了解底层之后，我们能明白，n变量在经过类型转换后（*struct -> interface），此时**变量已经被初始化为内存中的`eface`空接口，而非`nil`**



## 几道面试题

### 1.空接口有什么用

可用于表示通用类型，例如用在函数的参数或者返回值：

```go
func PrintValue(v interface{})
```

也可实现延迟类型判断

```go
var value interface{} = 114514
if v, ok := value.(int); ok {
    fmt.Println("It's a int: ", v)
}
```



### 2.interface是值类型还是引用类型？

首先，值类型和引用类型区别在于

- 值类型储存和传递值的实际数据
- 引用类型存储和传递的是实际值的地址（指针

初始化interface后，GO编译器会在内存中创建`iface`或`eface`结构体，不同接口变量拥有不同的结构体，所以应为值类型



### 3.当一个类型实现了interface的部分方法，会发生什么？

如果一个类型只实现了某个接口的部分方法，而不是全部方法，则该类型不被视为实现了该接口。Go 是静态类型语言，要求类型必须完全实现接口的所有方法才能满足接口的要求。



### 4.如何判断一个接口变量实际指向的具体类型？

通过断言判断：`value, ok := interfaceVariable.(Type)`，另外，使用 switch 语句可以根据接口变量的实际类型执行不同的逻辑。

```go
var value interface{} = 42

if v, ok := value.(int); ok {
	fmt.Println("It's an int:", v)
}
 
switch v := value.(type) {
case int:
    fmt.Println("It's an int:", v)
case string:
    fmt.Println("It's a string:", v)
default:
    fmt.Println("Unknown type")
}
```



### 5.interface的底层实现原理？

见上文

总结即

interface

- based on iface
  - itab
  - data

`data`指向具体数据

其他 接口类型、具体类型、包含的函数指针数组等在`itab`中

- based on eface
  - _type
  - data

`_type`指向具体类型的元数据

`data`指向具体数据

### 6.描述一个用interface解耦代码的场景

见上文
