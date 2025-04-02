---
title: go-string源码阅读
date: 2025-03-20 15:07:47
tags:
top: True
---

# GO string源码阅读及详解

在刷leetcode过程中碰到了使用string的题，发现自己还没专门看够Go中的string使用及实现等，故专门参考网络及MarsCode等产出了本篇文章

## Q：如何在GO中定义string？

#### A_1：

采用双引号赋值字符串，其中转义字符会进行转义（\t \n等

```go
str := "hello\tstring\n"
//output: hello string
```

#### A_2：

采用反引号定义字符串，转义字符不会进行转义

```go
str := `hello\tstring\n`
//output: hello\tstring\n
```

**tips:使用反引号可创建多行字符串**

```go
str := `hello
					string`
//output: hello
//					string
```



## Q：string底层？

> 基于Go SDK 1.23.0

在源码包中`src/runtime/string.go`

stringStruct定义了string的数据结构

```go
type stringStruct struct {
	str unsafe.Pointer	//字符串首地址，指向底层字节数组（大致同cpp
	len int				//字符串长度
}
```

在`src/builtin/builtin.go`中有对string的注释

```go
// string is the set of all strings of 8-bit bytes, conventionally but not
// necessarily representing UTF-8-encoded text. A string may be empty, but
// not nil. Values of string type are immutable.
type string string
```

即：

- 从底层来讲，Go 语言里的 string 类型是由 **8 位字节组成的序列**。这里的 8 位字节意味着它可以存储任意的字节数据，不一定非得是常规的文本字符所对应的字节。
- 注释里提到“conventionally but not necessarily representing UTF - 8 - encoded text”，意思是在实际应用中，大多数时候 string 用来存储 UTF - 8 编码的文本，这是一种常见的做法。不过，string 也可以存储其他编码格式的文本，甚至是二进制数据，例如图片、音频等数据的字节序列。
- 注释指出“ A string may be empty, but not nil”，这表明 string 类型**可以有一个空字符串值**，也就是 **""**，但它**不能是 nil**。在 Go 里，nil 一般用于表示指针、切片、映射、通道等类型的零值，而 string 不是这些类型，所以它不能被赋值为 nil。
- “Values of string type are immutable”表明 **string 类型的值是不可变的**（Java中也是类似的处理，String创建出来后就是作为一个常量存在常量池中，不可直接修改，在Java中只能另用stringbuilder/stringbuffer等修改）。一旦创建了一个 string，就不能修改它的内容。如果需要对字符串进行修改，**实际上是创建了一个新的字符串**。



## Q：使用string注意事项？

1. 通过string下标访问中文时，需转为rune序列，才能正确以UTF-8读取
2. 使用`range`遍历string时会将其拆成一个字节序列，再遍历其包含的每个UTF-8编码值，即每个Unicode字符



## Q：string创建过程？

```go
str := "hello string"
```

会调用`src/runtime/string.go`中的`gostringnocopy`方法

```go
//go:nosplit
func gostringnocopy(str *byte) string {
	ss := stringStruct{str: unsafe.Pointer(str), len: findnull(str)}
	s := *(*string)(unsafe.Pointer(&ss))
	return s
}
```

接收一个byte指针作为参数，返回一个string类型值

`unsafe.Pointer`用于将`*byte`转换成`unsafe.Pointer`，从而绕过Go的安全检查机制，直接访问内存中的数据

`findnull()`则可以理解为一个保护措施，在 `gostringnocopy` 函数里，`findnull` 函数被用来确定传入的 `*byte` 指针所指向的 C 风格字符串的长度，然后用这个长度来创建一个 Go 字符串。

在 Go 语言里，字符串一般是用 UTF-8 编码的，并且有明确的长度信息。不过在和 C 语言交互或者处理一些以空字符结尾的字符串数据时，就需要找出空字符的位置来确定字符串的长度。`findnull` 函数就是为了实现这个目的而存在的。

```go
func findnull(s *byte) int {
	//...执行流程略
}
```

最后的`s := *(*string)(unsafe.Pointer(&ss))`可理解为在告诉编译器如何理解`ss`指向的内存布局，如何将其视为一个合法的`string`对象

> ss取地址 -> 视为unsafe.Pointer类型指针 -> 强转为*string指针 -> *操作取指针指向的对象的值（将其视为\*string后



## Q：为什么要使用stringStruct？

```go
type stringStruct struct {
	str unsafe.Pointer	//字符串地址，指向底层字节数组（大致同cpp
	len int				//字符串长度
}
```

stringStruct作为byte序列和string的中间层，其包含两个信息

- 指向数据地址的指针
- 字符串长度

有这个中间层存在，在处理大字符串，如对其切片等操作时，不会再次复制整个字符串的数据，而是操作`stringStruct`只调整指针指向地址以及字符串长度

> 当然，以上都为Go runtime内部实现，我们非必须了解其背后细节



## Q：GO是否存在string常量池？

直接上代码跑跑就知道了

```go
	str1 := "abc"
	str2 := "abc"
	
	fmt.Println(str1 == str2)
	fmt.Println(&str1, &str2)

	fmt.Println(unsafe.StringData(str1), unsafe.StringData(str2))

//output: 	true
//			0xc0000222a0 0xc0000222b0 	不是同一stringStruct，地址不同
//			0xfb9fc6 0xfb9fc6			底层是同一字符数组
```

即，**Go虽然没有明确的常量池机制，但是Go编译器会对字符常量进行优化**



## Q：如何修改string？

`string`类型初始化后，直接对其修改会报错

```go
str := "hello go"
str[0] = "X"
//output: 报错 cannot assign to...
```



#### string与[]byte相互转换

可将string转为[]byte修改后，再转回string

```go
	str := "hello go"
	fmt.Println(str)
	fmt.Println(&str)
	fmt.Println(unsafe.StringData(str))
	bytes := []byte(str)
	bytes[0] = 'X'
	str = string(bytes) //此时str指向的仍是原来的stringStruct，但是stringStruct的data指针已经指向了新的byte数组
	fmt.Println(str)
	fmt.Println(&str)
	fmt.Println(unsafe.StringData(str))

//output: 	hello go
//			0xc00008e260
//			0x10a4bc
//			Xello go
//			0xc00008e260
//			0xc00008c0d0
```

> 转换过程中存在**内存拷贝**



## Q：为什么不允许修改string？

Java同理

修改字符串太麻烦，不如直接重新创建

当字符串内容固定不变时，Go可以**安全地共享相同的字符串实例，减少不必要的内存复制**。例如子串提取和拼接可以通过简单的指针调整实现，而无需复制整个数据，**显著提升了处理*大字符串*或*频繁操作*时的性能**。



## Q：字符串拼接常用方法

#### 1.加号拼接

最理所当然想到的方式

```go
str1 := "hello"
str2 := "go"
str := str1 + str2
fmt.Println(str)
//output: hellogo
```

使用`+`拼接时，底层会开辟一块新空间，将两个str内容进行复制，最终将复制的str合并在新空间中。

对于频繁的字符串拼接来说效率不高。



#### 2.fmt.Sprintf

`fmt.Sprintf`为格式化输出函数之一，接收一个格式化字符串和一系列参数，返回一个格式化后的字符串。

适合需要插入变量或控制输出格式的场合（典例手搓`toString()`方法

```go
str1:="hello"
str2:="go"
str:= fmt.Sprintf("%s%s", str1, str2)
fmt.Println(str)
//output: "hellogo"
```

**由于它涉及到解析格式化字符串，性能上会有一定影响**。特别是当使用`%v`或`%+v`等通用格式化动词时，它需要了解传递给它的值的实际类型以便正确地进行格式化。为了做到这一点，fmt包会使用到**反射**，对大量数据进行操作时，会对性能会有较大的影响。



#### 3.strings.Builder

`strings.Builder`是`Go1.10`引入的一个高效可变字符串缓冲区，特别适合于需要进行多次追加操作的场景。

```go
str1:="hello"
str2:="go"
var sb strings.Builder
sb.WriteString(str1)
sb.WriteString(str2)
str:= sb.String()
fmt.Println(str)
//输出："hellogo"
```

源码在`src/strings/builder.go`

```go
// A Builder is used to efficiently build a string using [Builder.Write] methods.
// Builder结构体用于通过Builder.Write系列方法高效构建字符串
// It minimizes memory copying. The zero value is ready to use.
// 能最大程度减少内存复制操作，从而提高性能。同时Builder的零值可以直接使用，无需额外的初始化
// Do not copy a non-zero Builder.
// 不要复制一个已使用（即非零）的Builder
type Builder struct {
	addr *Builder // of receiver, to detect copies by value

	// External users should never get direct access to this buffer, since
	// the slice at some point will be converted to a string using unsafe, also
	// data between len(buf) and cap(buf) might be uninitialized.
	buf []byte
}
```

`addr`为一个指向`Builder`结构体的指针，它的作用是检测`Builde`是否被按值复制。当调用`Builder`的方法时，会检查`addr`是否等于当前实例的指针，如果不相等，就会触发panic，提示非法使用按值复制的`Builder`，即同时存在两个`Builder`向同一个`buf`写值

`buf`为一个字节切片，用于存储构建过程中的字符串数据

注释额外提醒不要直接访问此缓冲区：

- 这个切片在某个时刻会使用`unsafe`包的方法转换为字符串，直接访问可能会破坏内部实现。
- `buf`的长度`len(buf)`和容量`cap(buf)`的数据可能是未初始化的，直接访问可能会导致未定义行为。



与+和fmt.Sprintf相比，**strings.Builder在处理大量字符串拼接时性能更好，因为它提取分配了buf内存，减少了内存分配次数。**



#### 4.strings.join

`strings.Join`接受一个字符串切片和一个分隔符作为参数，然后将切片中的所有元素用分隔符连接成一个单一的字符串。

当我们有一个已经分割好的字符串列表并且想要用特定字符连接它们时，该方式是最理想选择。

```go
str1:="hello"
str2:="go"
strs := []string{str1, str2}
str = strings.Join(strs, "")
fmt.Println(str)
//output: "hellogo"
```

源码在`/src/strings/strings.go`中

```go
// Join concatenates the elements of its first argument to create a single string. The separator
// string sep is placed between elements in the resulting string.
func Join(elems []string, sep string) string {
	switch len(elems) {
	case 0:
		return ""
	case 1:
		return elems[0]
	}
    //判断特况，直接返回

	var n int
	if len(sep) > 0 {
		if len(sep) >= maxInt/(len(elems)-1) {
			panic("strings: Join output length overflow")
		}
		n += len(sep) * (len(elems) - 1)
	}
	for _, elem := range elems {
		if len(elem) > maxInt-n {
			panic("strings: Join output length overflow")
		}
		n += len(elem)
	}
    //计算join后的字符串总长度，即n，过长则报panic

	var b Builder
	b.Grow(n)
    //Grow方法基于预计算的n预分配空间
	b.WriteString(elems[0])
	for _, s := range elems[1:] {
		b.WriteString(sep)
		b.WriteString(s)
	}
    //基于Buider.WriteString实现join
	return b.String()
}
```

`strings.Join`基于`strings.builder`来实现的,并且可以自定义分隔符，能**提前预分配buf的空间**，减少了内存分配消耗的性能



#### 5.bytes.Buffer

`bytes.Buffer`是一个实现了`io.Writer`接口的类型，它可以像`strings.Builder`一样被用来构建字符串，但它实际上是为处理字节流设计的。

源码在`src/bytes/buffer.go`中

```go
// A Buffer is a variable-sized buffer of bytes with [Buffer.Read] and [Buffer.Write] methods.
// The zero value for Buffer is an empty buffer ready to use.
type Buffer struct {
	buf      []byte // contents are the bytes buf[off : len(buf)]
	off      int    // read at &buf[off], write at &buf[len(buf)]
	lastRead readOp // last read operation, so that Unread* can work correctly.
}
```

在`bytes.Buffer`也存在字节切片buf，通过`WriteString`方法在底层的`[]byte切片`中进行拼接。

```go
str1:="hello"
str2:="go"
var buffer bytes.Buffer
buffer.WriteString(str1)
buffer.WriteString(str2)
str:= buffer.String()
fmt.Println(str)
//output: "hellogo"
```

但`bytes.Buffer`的性能略逊于`strings.Builder`，因为`strings.Buider`对字符串拼接有专门优化。
