---
title: go的time包使用
date: 2019-10-08 23:16:21
tags: go
---

<details>
<summary>详细信息</summary>

### 1.和时间记录有关

###### time.Now()

```go
t := time.Now()
fmt.Println("当前时间：",t)
fmt.Println("type:", reflect.TypeOf(t))
```

```go
当前时间： 2019-10-09 00:36:37.7604236 +0800 CST m=+0.002002501
type: time.Time
```

返回一个Time结构体，使用time.Now()的时候调用了一堆location有关的东西，自动生成了时区

###### time.Now().Unix()

```go
fmt.Println("时间戳：",time.Now().Unix())
fmt.Println("type:", reflect.TypeOf(time.Now().Unix()))
```

```go
时间戳： 1570549117
type: int64
```

是对结构体Time的方法

```go
func (t Time) Unix() int64 { return t.unixSec()}
```

返回当前的时间戳,int64类型，存储总用到，需要和time.Unix()区分开

###### time.Now().Format()

```go
fmt.Println("格式化的时间：",time.Now().Format("2006.01.02 15:04:05"))fmt.Println("type:", reflect.TypeOf(t.Format("2006.01.02 15:04:05")))
```

```go
格式化的时间： 2019.10.09 00:36:37
type: string
```

其中Format有很多写法，源码写了一堆case，这个直接存储为string挺不错的，需要注意的是，那个2006.01.02必须是2006.01.02。

注意Format()函数是对Time结构体的方法，例如time.Unix()，time.Now()返回的都是Time结构体，可以直接跟Formate()

###### time.Unix（）

这函数用于把一个int64的时间戳还原成Time结构体，具体实现如下

```
func Unix(sec int64, nsec int64) Time {
	if nsec < 0 || nsec >= 1e9 {
		n := nsec / 1e9
		sec += n
		nsec -= n * 1e9
		if nsec < 0 {
			nsec += 1e9
			sec--
		}
	}
	return unixTime(sec, int32(nsec))
}
func unixTime(sec int64, nsec int32) Time {
	return Time{uint64(nsec), sec + unixToInternal, Local}
}
```

<!--注：纳秒:nanosecond(ns|nsec);物理学上,其数值为10的负9次方秒，好像不怎么用，一般填0就可以了，据说有时候32位系统会用到-->

把存储的时间戳转化为Time结构体以实现字符串传输经常用到,例如

```go
t := time.Unix(1570549117, 0).Format("2006-01-02 03:04:05 PM")
```

###### time.Now.Second()

源码如下：

```go
func (t Time) Second() int {   return int(t.abs() % secondsPerMinute)}
// Second returns the second offset within the minute specified by t, in the range [0, 59].
```

用于把一个Time结构体转化为秒数，类似的，还有

```go
func (t Time) Hour() 
func (t Time) Minute() 
func (t Time) Nanosecond() 
func (t Time) YearDay() 
```

Time的结构体是这样的：

```go
type Time struct {  

// wall and ext encode the wall time seconds, wall time nanoseconds,  
// and optional monotonic clock reading in nanoseconds.   
// From high to low bit position, wall encodes a 1-bit flag (hasMonotonic),   
// a 33-bit seconds field, and a 30-bit wall time nanoseconds field.   
// The nanoseconds field is in the range [0, 999999999].   
// If the hasMonotonic bit is 0, then the 33-bit field must be zero   
// and the full signed 64-bit wall seconds since Jan 1 year 1 is stored in ext.   
// If the hasMonotonic bit is 1, then the 33-bit field holds a 33-bit  
// unsigned wall seconds since Jan 1 year 1885, and ext holds a   
// signed 64-bit monotonic clock reading, nanoseconds since process start.   

wall uint64  
ext  int64   

// loc specifies the Location that should be used to   
// determine the minute, hour, month, day, and year   
// that correspond to this Time.   
// The nil location means UTC.   
// All UTC times are represented with loc==nil, never loc==&utcLoc.  

 loc *Location
}
```

注释里有具体含义，其中的成员都是私有，不容易直接显示出来



### 2.进程阻塞

阻塞多少时间的函数：

```
func Sleep(d Duration)
```

比如 `Sleep(time.Second*2)`

time.Second或者其他单位数据类型都是Duration，也可以直接Sleep(1000)这样子

其中Duration也是int64数据类型，表示阻塞多少时间，等同Nanosecond（十亿分之一秒），还有个范围，用的位运算表示

```go
const (   
minDuration Duration = -1 << 63   
maxDuration Duration = 1<<63 - 1
)
```

最小是-1乘以2的63次方纳秒，   计算的结果是-2562047h47m16.854775808s,   -2.5620477880152157e+06h

最大是1乘以2的63次方纳秒减1, 计算结果就不写了，官方给出的最大时间为 2540400h10m10.000000000s

```
const (   
Nanosecond  Duration = 1   
Microsecond          = 1000 * Nanosecond   
Millisecond          = 1000 * Microsecond   
Second               = 1000 * Millisecond   
Minute               = 60 * Second   
Hour                 = 60 * Minute
)
```

它本身是int64，对它封装了一系列方法操作，感觉就有点像面向对象编程

举几个例子：

将Duration变量转化为其他单位

```go
func (d Duration) Seconds() float64
func (d Duration) Minutes() float64
func (d Duration) Hours() float64
func (d Duration) Nanoseconds() int64 { return int64(d) }  （这个操作有点意思）
```

 将Duration变量输出为合适的String

```go
func (d Duration) String() string
```

<!--注：fmt.Println()输出时默认输出Duration的String函数转化后的字符串-->

求时间的近似值

```go
func (d Duration) Round(m Duration) Duration
```

比较好用的还有时间相加减的函数，是对Time对象的操作

```go
func (t Time) Add(d Duration)
```

比如`time.Now().Add(time.Second)`

如果想两个时间相加，两个Duration对象直接相加就行

```go
fmt.Println(time.Second+time.Second)     //2s
```



</details>