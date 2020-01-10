---
layout: post
title: 'Go之Reflect快速入门'
date: 2020-01-10
author: Tinson
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: Reflect Go
---

## Reflect
本文侧重讲解reflect反射的实践应用，适合新手初窥门径。

### reflect两个基本功能
- reflect.TypeOf() 动态获取输入数据的类型
- reflect.ValueOf() 动态获取输入数据的值

```golang
func TypeOf(i interface{}) Type
func ValueOf(i interface{}) Value
```

通过reflect.Type和reflect.Value支持的方法，可以对输入的动态数据进行解析。
那么了解reflect.Type和reflect.Value提供的方法尤为重要，因为比较多，此介绍放在文末。

### reflect.Kind
在reflect.Type和reflect.Value上调用Kind()方法，可以得到reflect.Kind类型值，从而知道动态数据的类型。这个是动态解析的钥匙，通过反射后拿到具体的类型才能做相应的工作，逐层解析。  

```
const (
	Invalid Kind = iota
	Bool
	Int
	Int8
	Int16
	Int32
	Int64
	Uint
	Uint8
	Uint16
	Uint32
	Uint64
	Uintptr
	Float32
	Float64
	Complex64
	Complex128
	Array
	Chan
	Func
	Interface
	Map
	Ptr
	Slice
	String
	Struct
	UnsafePointer
)
``` 

## 解析流程
- Ptr指针和Interface 通过调用Elem()方法得到指向元素的值，再进入循环解析。
- Slice、Array、Map、Struct复合数据类型，通过各自的方法拿到他们组成类型值，再进入循环解析
- 直到为基础数据类型或者Func、Chan、UnsafePointer。
![reflect解析动态数据流程](/assets/img/reflect-1.jpg)

## 具体类型解析
下述demo中默认  

```
rt = reflect.TypeOf(data)
rv = reflect.ValueOf(data)
```

### Struct
```
//遍历结构体的Field
for i := 0; i < rt.NumField(); i++ {
    rt.Field(i) //第i个Field的StructField
    rv.Filed(i) //第i个Field的Value
}

//遍历结构体的方法
for i := 0; i < rt.NumMethod(); i++ {
    rt.Method(i) //第i个方法的Method
    rv.Method(i) //第i个方法的Value
}
```

### Map
```
//遍历Map的key和value
for _, key := range rv.MapKeys() {
    //其中key为键的Value
    rv.MapIndex(key) //键值的Value
}

rv.Len() //map的大小
rv.SetMapIndex() //map索引赋值
```

### Slice Array
```
//遍历Slice或者Array
for i := 0; i < rv.Len(); i++ {
    rv.Index(i) //子元素的Value
}

rv.Cap()    //获取切片或者Array容量
rv.SetCap(i) //调整切片容量
rv.Slice(i, j) //返回切片s[i:j]的Value
```

### Ptr Interface
```
if !rv.IsNil() {
    rv.Elem() //指针或者interface实际指向元素
}
```

### Bool Int Unit Float Complex String
```
rv.Bool() //返回bool
rv.Int() //返回int64
rv.Uint() //返回uint64
rv.Float() //返回float64
rv.Complex() //返回complex128
rv.Sting() //返回string

//修改Value结构中的值，必须类型一致
rv.Set()
rv.SetBool() 
rv.SetInt() 
rv.SetUint() 
rv.SetFloat() 
rv.SetComplex() 
rv.SetString()
rv.SetBytes()
```

### Func Chan UnsafePointer
动态解析的时候，用得比较少，这里就不说了


## Type和Value方法

### Type和Value拥有的同名方法  
|  Method   |  Type返回类型  | Value返回类型  |  备注   |
| :----  | :----  | :----  | :----  |
| Kind | Kind	| Kind	| 返回指定对象的Kind类型 |
| NumMethod	| int | int | 返回struct拥有的方法总数，包括unexported方法 |
| MethodByName | Method | Value | 根据方法名找方法 |
| Method	| Method | Value	| 返回第i个方法 |
| NumField | int | int | 返回struct所包含的field数量 |
| Field | StructField | Value	|取struct结构的第n个field |
| FieldByIndex | StructField | Value |嵌套的方式取struct的field，比如v.FieldByIndex([]int{1,2})等价于 v.field(1).field(2) |
| FieldByName | StructFiel,bool | Value | 返回名称匹配match函数的field |
| FieldByNameFunc | StructField,bool | Value 	| 返回名称匹配match函数的field |

### Type独有的方法
|  Method  |  备注   |
| :----  | :----  |
|Align	|分配内存时的内存对齐字节数 |
|FieldAlign	|作为struct的field时内存对齐字节数 |
|Name	|type名 string类型 |
|PkgPath	|包路径， "encoding/base64"， 内置类型返回empty string |
|Size	|该类型变量占用字节数 |
|String	|type的string表示方式 |
|Implements	|判断该类型是否实现了某个接口 |
|AssignableTo	|判断该类型能否赋值给某个类型 |
|ConvertibleTo	|判断该类型能否转换为另外一种类型 |
|Comparable	|判断该类型变量是否可以比较 |
|ChanDir	|返回channel的方向 recv/send/double |
|IsVariadic	|判断函数是否接受可变参数 |
|Elem	|取该类型的元素 |
|In	|函数第n个入参 |
|Out	|函数第n个出参 |
|NumIn	|函数的入参数个数 |
|NumOut	|函数的出参个数 |
|Key	|返回map结构的key类型Type |
|Len	|返回array的长度 |

### Value独有的方法
|  Method  |  备注   |
| :----  | :----  |
|Addr	| v的指针，前提时CanAddr()返回true |
|Bool	| 取值，布尔类型 |
|Bytes	| 取值，字节流 |
|Call	|调用函数 |
|CallSlice	|调用具有可变参的函数 |
|CanAddr	|判断能否取址 |
|CanInterface	|判断Interface方法能否使用 |
|CanSet	|判断v的值能否改变 | 
|Cap	|判断容量 Array/Chan/Slice |
|Close	|关闭Chan |
|Complex	| 取值，复数 |
|Convert	|返回将v转换位type t的结果 |
|Elem	| 返回interface包含或者Ptr指针的实际值 |
|Float	 | 取值，浮点型 |
|Index	索引操作 | Array/Slice/String |
|Int	 | 取值，整型 |
|Interface	|将当前value以interface{}形式返回 |
|IsNil	|判断是否为nil，chan, func, interface, map, pointer, or slice value |
|IsValid	|是否是可操作的Value，返回false表示为zero Value |
|Len	|适用于Array, Chan, Map, Slice, or String |
|MapIndex	|对map类型按key取值 |
|MapKeys	|map类型的所有key的列表 |
|OverflowComplex	 | 溢出判断 |
|OverflowFloat	| 溢出判断 |
|OverflowInt	  | 溢出判断 |
|OverflowUint	  | 溢出判断 |
|Pointer	|返回uintptr 适用于slice |
|Recv	| chan接收 |
|Send	| chan发送 |
|Set	| 将x赋值给v，类型要匹配 |
|SetBool	 | Bool赋值，需要先判断CanSet()为true |
|SetBytes	 | Bytes赋值 |
|SetCap	| slice调整切片容量 |
|SetMapIndex	| map索引赋值 |
|SetUint	 | Unit赋值 |
|SetPointer	|unsafe.Pointer赋值 |
|SetString	 | String赋值 |
|Slice	 | return v[i:j] 适用于Array/Slict/String |
|String	| return value的string表示方法 |
|TryRecv	| chan非阻塞接收 |
|TrySend	| chan非阻塞发送 |
|Type	| 返回value的Type |
|UnsafeAddr	| 返回指向value的data的指针 |


### 引用链接
Type和Value方法: [https://www.cnblogs.com/ksir16/p/9040656.html](https://www.cnblogs.com/ksir16/p/9040656.html)
