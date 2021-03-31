---
layout: post
title: Go之URL Query String编码器和解码器
date: 2020-12-01
author: Tinson
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: QueryString
---

### 项目地址
https://github.com/hetiansu5/urlquery

### 简介
使用Go语言实现的URL Query字符串编码器和解码器。写好后才发现官方已有实现的[querystring](https://github.com/google/go-querystring)，但只实现了编码器，没有解码器，且只支持顶层数据结构为结构体，实现上不算特别完善。

### 特性
- 支持丰富的Go数据结构互转：
    - 基础数据类型: 有符号整型[8,16,32,64] 无符号整形[8,16,32,64] 字符串 布尔值 浮点型[32,64] 字节 字面量
    - 复合数据类型: 数组 切片 哈希 结构体
    - 嵌套结构体
- 支持顶层的数据结构为数组 切片 哈希，不仅仅是结构体
- 支持自定义的URL-Encode编码规则，支持全局、局部设置方式，支持默认规则
- 支持自定义的键名映射规则（结构体Tag示例：`query:"name"`）
- 支持开启或者关闭忽略结构体零值编码（默认开启），以减少编码后字符串长度


### 快速入门
更多查看[example](https://github.com/hetiansu5/urlquery/blob/master/example/withoption.go)

```golang
package main

import (
    "github.com/hetiansu5/urlquery"
    "fmt"
)

type SimpleChild struct {
    Status bool `query:"status"`
    Name   string
}

type SimpleData struct {
    Id         int
    Name       string          `query:"name"`
    Child      SimpleChild
    Params     map[string]int8 `query:"p"`
    Array      [3]uint16
}

func main() {
    data := SimpleData{
        Id:   2,
        Name: "http://localhost/test.php?id=2",
        Child: SimpleChild{
            Status: true,
        },
        Params: map[string]int8{
            "one": 1,
        },
        Array: [3]uint16{2, 3, 300},
    }

    //Marshal: from go structure to url query string
    bytes, _ := urlquery.Marshal(data)
    fmt.Println(string(bytes))

    //Unmarshal: from url query  string to go structure
    v := &SimpleData{}
    urlquery.Unmarshal(bytes, v)
    fmt.Println(*v)
```

### 注意事项
- 针对Map数据类型，Marshal可以支持map[基础数据类型]基础数据类型|复合数据类型，Unmarshal只能支持map[基础数据类型]基础数据类型
- 结构体零值忽略编码默认开启，可以通过[Option](example/withoption.go)关闭此功能
- 字节实际上是uint8，字面量是int32，所以编码后其实是整型，解码的时候也需要接收的是整型