---
title: slice
---
## 数据结构

```go
type slice struct {
	array unsafe.Pointer // 指向底层数组的指针
	len   int            // 切片的长度
	cap   int            // 切片的容量
}

```
## 创建
```go
// 此时s的长度和容量都是5。
s := make([]int, 5)
// 此时s的长度是3,容量是5。
s := make([]int, 3, 5)
```
## 取值
```go
// 该方式没有指定开始和结束索引,是从索引0开始取到结束,即取出slice a的全部元素。
a[:]
// 指定开始索引startIndex,结束索引取到结尾。比如a[1:]会从索引1开始取到最后一个元素。
a[startIndex:]
// 指定结束索引endIndex,开始索引从0开始。比如a[:3]会取索引0-2的三个元素。
a[:endIndex]
// 指定开始索引和结束索引区间。比如a[1:3]会取索引1和2的两个元素。
a[startIndex:endIndex]
// 三个索引除了开始和结束索引外,还可以指定容量capacity。这种方式可以对slice进行扩容或缩容操作。
a[startIndex:endIndex:capacity]
```
- a[:] - 取所有元素
- a[1:] - 从索引1开始取到最后
- a[:3] - 取索引0-2的三个元素
- a[1:3] - 取索引1和2的两个元素
- a[1:3:5] - 取索引1-2但是扩容为5