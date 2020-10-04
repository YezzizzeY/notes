---
title: go中的struct和json
date: 2019-10-09 02:33:27
tags: go
---
<details>
<summary>详细信息</summary>
## 1.先说结构体

首先定义一个结构体很容易，比如说：

```go
type Person struct {   
  Id string `json:"id"`   
  Name string `json:"name"`   
  Sex    string `json:"sex"`   
  Born   string `json:"born"`
}
```

不过要多多注意，go没有特意声明私有成员，大写代表公有，小写就代表私有，结构体内部的命名也一样
$$
如果结构体命名为小写，则整个结构体其他包不能导出；

如果内部名称有小写，则改结构体内部的成员其他包不能导出
$$
**也就是说，使用json.Marshal的时候会出大问题，（这容易被忽视掉）**

结构体的声明：go的结构体声明方法还是很多的

佚名结构体：

```go
    p2 := struct {
        Name string
        Age  int
    }{
        "张三", 20,
    }
```

声明并赋值：

```go
Ming := Person{Id:"1",Name:"Ming",Sex:"Male",born:"1999-11-06"}
```

正常声明：

```go
Xiaoli := new(Person)
var Ming Person = Person{}
Feng := &Person{}
```

结构体还可以嵌套，这玩意有点像继承：

```go
type Person struct {   
	Id string `json:"id"`   
	Name string `json:"name"`   
	Sex    string `json:"sex"`   
	Born   string `json:"born"`
}
type Pig struct{   
	Person   
	Tag string `json:"tag"`
}
```

而且可以直接对Pig结构体P 使用 P.Id这样的式子，很舒服

## go 中的Json操作：

标准的encoding/json包中使用的最多的是json.Marshal和json.Unmarshal

```go
b,err := json.Marshal(Xiaoli)
fmt.Println(b)
fmt.Println("b type :",reflect.TypeOf(b))
```

```go
[123 34 105 100 34 58 34 49 50 51 34 44 34 110 97 109 101 34 58 34 88 105 97 111 108 105 34 44 34 115 101 120 34 58 34 70 101 109 97 108 101 34 44 34 98 111 114 110 34 58 34 49 57 57 57 45 48 57 45 50 57 34 125]
b type : []uint8
```

Marshal出来的是个字节码，想显示还得用string()

上面的成员变量都是已知的类型，只能接收指定的类型，比如string类型的Name只能赋值string类型的数据。
但有时为了通用性，或使代码简洁，我们希望有一种类型可以接受各种类型的数据，并进行json编码。这就用到了interface{}类型。

<!--interface{}类型其实是个空接口，即没有方法的接口。go的每一种类型都实现了该接口。因此，任何其他类型的数据都可以赋值给interface{}类型。-->

```go
type Person struct {   
	Id interface{} `json:"id"`   
	Name interface{} `json:"name"`   
	Sex    interface{} `json:"sex"`   
	Born interface{} `json:"born"`
}
```

关于接收json的操作，我推荐用个别的包：https://github.com/tidwall/gjson

```go
package main

import "github.com/tidwall/gjson"

const json = `{"name":{"first":"Janet","last":"Prichard"},"age":47}`

func main() {
	value := gjson.Get(json, "name.last")
	println(value.String())
}
```

比官方那个包好用多了。
</details>