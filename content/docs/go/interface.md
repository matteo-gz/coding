---
title: interface
---
## 数据结构
```go
// 结构体表示包含方法的接口
type iface struct {
	tab  *itab
	data unsafe.Pointer
}
// 结构体表示不包含任何方法的 interface{} 类型
type eface struct {
	_type *_type
	data  unsafe.Pointer
}
```
Golang中的Interface可以被看作是一个Wrapper，它是一个包含了value和type的二元组

```go
// 必须类型和值都为nil才算真正的nil

var a interface{} = nil         // tab = nil, data = nil
var b interface{} = (*int)(nil) // tab 包含 *int 类型信息, data = nil
fmt.Println(a == nil)           // true
fmt.Println(b == nil)           // false
```

### 判断动态值为nil
```go
func IsNil(i interface{}) bool {
    vi := reflect.ValueOf(i)
    if vi.Kind() == reflect.Ptr {
        return vi.IsNil()
    }
    return false
}
```

## 用处


```go
// 编译器会由此检查 *myWriter 类型是否实现了 io.Writer 接口
var _ io.Writer = (*myWriter)(nil)
```
