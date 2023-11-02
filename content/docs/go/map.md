---
title: map
---
## 结构
```go
// Go map 的头部。
type hmap struct {
// 注意：hmap 的格式也被编码在 cmd/compile/internal/reflectdata/reflect.go 中。
// 确保这个定义与编译器的定义保持同步。
count     int   // 存储在 map 中的键值对数量。必须是第一个字段（用于内置函数 len()）
flags     uint8 // 表示 map 的状态标志，包括了迭代器是否在使用中、是否正在进行扩容等信息。
B         uint8 // 存储桶的数量的对数，实际桶的数量为len(buckets) == 2^B(bucketShift bucket的位移值)。 
noverflow uint16 // 溢出桶数量的估计值。 number of overflows
hash0     uint32 // 哈希种子。

buckets    unsafe.Pointer // 存储键值对的桶数组，其长度为 2^B。如果 count 为 0，则可能为 nil。
oldbuckets unsafe.Pointer // 扩容时旧的桶数组，长度为 2^(B-1)，用于数据搬迁。如果没有扩容，则为 nil。
nevacuate  uintptr        // 扩容时已经完成搬迁的桶数量。 "not evacuate"

extra *mapextra // 可选字段，指向了一些额外的 map 属性，例如 map 的类型信息和哈希函数。
}

// mapextra 包含了一些不是所有 map 都有的字段。
type mapextra struct {
// 如果 key 和 elem 都不包含指针，并且它们都可以内联，那么我们标记 bucket 的类型不包含指针。
// 这样可以避免扫描这样的 map。
// 然而，bmap.overflow 是一个指针。为了保持溢出桶的存活状态，我们在 hmap.extra.overflow 和 hmap.extra.oldoverflow 中存储了指向所有溢出桶的指针。
// 只有当 key 和 elem 都不包含指针时才使用 overflow 和 oldoverflow。
// overflow 存储 hmap.buckets 的溢出桶。
// oldoverflow 存储 hmap.oldbuckets 的溢出桶。
// 间接存储允许在 hiter 中存储一个指向切片的指针。
overflow *[]*bmap
oldoverflow *[]*bmap
// nextOverflow 指向一个空闲的溢出桶。
nextOverflow *bmap
}

// Go map 的 bucket。 bmap equal bucket map
type bmap struct {
// tophash 通常包含此 bucket 中每个 key 的 hash 值的高字节。 存储了键的哈希的高 8 位
// 如果 tophash[0] < minTopHash，则 tophash[0] 是一个 bucket 撤离状态。
tophash [bucketCnt]uint8 // bucketCnt equal bucket count
// 接下来是 bucketCnt 个 key，然后是 bucketCnt 个 elem。
// 注意：将所有 key 放在一起，然后将所有 elem 放在一起比交替 key/elem/key/elem/... 代码更复杂，但可以消除需要填充的情况，例如 map[int64]int8。
// 最后是一个溢出指针。
}

src/cmd/compile/internal/reflectdata/reflect.go::MapBucketType

type bmap struct {
topbits  [8]uint8
keys     [8]keytype
values   [8]valuetype
pad      uintptr
overflow uintptr
}
```