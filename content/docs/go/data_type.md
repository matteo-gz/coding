---
title: 数据类型
---
## 值类型
- int
- bool
## 引用类型
- chan
- slice
- map
- 指针

## make 和 new
make 初始化内置的数据结构
```go
slice := make([]int, 0, 100)
hash := make(map[int]bool, 10)
ch := make(chan int, 5)
```
new 返回类型指针
```go
i := new(int)

var v int
i := &v
```

## 支持平台
```go
// 可列出支持的平台 GOOS GOARCH
go tool dist list
```