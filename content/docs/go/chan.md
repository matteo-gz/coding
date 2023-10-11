---
title: chan
---
# chan
## 操作chan
创建chan
```go
make(chan T, [capacity])

// 无缓冲,创建一个int类型的channel
ch := make(chan int)

// 有缓冲,创建一个容量为5的int缓冲channel
ch := make(chan int, 5)
```
收发chan
```go
ch <- elem // 发送
elem := <-ch //接收

// 如果ok为false,那么可能是channel已经关闭,读到的时关闭后的零值。
i, ok <- ch
```

## panic 情况

- 向关闭chan发送
- 关闭nil chan
- 关闭已关闭 chan

| 操作      | nil chan | 已关闭chan   | 正常chan                                                |
|---------|----------|-----------|-------------------------------------------------------|
| 关闭      | panic    | panic     | 正常关闭                                                  |
| 读 | 阻塞       | 读可以继续读取元素,直到chan空,如果读完会读到对应类型的零值 | 读会阻塞,如果chan空或者没有其他goroutine写入    |
| 写 | 阻塞       | panic     | 写也可能阻塞,如果chan空间不足或者没有其他goroutine读取 |

## 数据结构
```go

type hchan struct {
	qcount   uint           // 队列中的元素总数量
	dataqsiz uint           // 循环队列的长度
	buf      unsafe.Pointer // 指向长度为 dataqsiz 的底层数组
	elemsize uint16         // 能够接受和发送的元素大小
	closed   uint32         // 是否关闭
	elemtype *_type         // 能够接受和发送的元素类型
	sendx    uint           // 已发送元素在循环队列中的索引位置
	recvx    uint           // 已接收元素在循环队列中的索引位置
	recvq    waitq          // 接受者的 sudog 等待队列
	sendq    waitq          // 发送者的 sudog 等待队列

	// lock 保护 hchan 中的所有字段，以及 blocked on 此 channel 的 sudog 中的一些字段。
	//
	// 在持有此锁时不要更改另一个 G 的状态（特别是不要准备一个 G），
	// 因为这可能会导致与栈缩减死锁。
	lock mutex
}

```