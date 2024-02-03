# `v2ray-core\common\buf\buffer.go`

```go
package buf

import (
    "io"

    "v2ray.com/core/common/bytespool"
)

const (
    // Size of a regular buffer.
    Size = 2048
)

// Buffer is a recyclable allocation of a byte array. Buffer.Release() recycles
// the buffer into an internal buffer pool, in order to recreate a buffer more
// quickly.
type Buffer struct {
    v     []byte  // 存储字节数据的数组
    start int32   // 缓冲区的起始位置
    end   int32   // 缓冲区的结束位置
}

// Release recycles the buffer into an internal buffer pool.
func (b *Buffer) Release() {
    if b == nil || b.v == nil {
        return
    }

    p := b.v  // 保存缓冲区的引用
    b.v = nil  // 清空缓冲区
    b.Clear()  // 调用 Clear 方法清空缓冲区内容
    pool.Put(p)  // 将缓冲区放回内部缓冲池
}

// Clear clears the content of the buffer, results an empty buffer with
// Len() = 0.
func (b *Buffer) Clear() {
    b.start = 0  // 将起始位置重置为 0
    b.end = 0    // 将结束位置重置为 0
}

// Byte returns the bytes at index.
func (b *Buffer) Byte(index int32) byte {
    return b.v[b.start+index]  // 返回指定位置的字节数据
}

// SetByte sets the byte value at index.
func (b *Buffer) SetByte(index int32, value byte) {
    b.v[b.start+index] = value  // 设置指定位置的字节数据
}

// Bytes returns the content bytes of this Buffer.
func (b *Buffer) Bytes() []byte {
    return b.v[b.start:b.end]  // 返回缓冲区的内容字节数据
}

// Extend increases the buffer size by n bytes, and returns the extended part.
// It panics if result size is larger than buf.Size.
func (b *Buffer) Extend(n int32) []byte {
    end := b.end + n  // 计算扩展后的结束位置
    if end > int32(len(b.v)) {  // 如果结束位置超出了缓冲区的大小
        panic("extending out of bound")  // 抛出异常
    }
    ext := b.v[b.end:end]  // 获取扩展部分的字节数据
    b.end = end  // 更新结束位置
    return ext  // 返回扩展部分的字节数据
}

// BytesRange returns a slice of this buffer with given from and to boundary.
func (b *Buffer) BytesRange(from, to int32) []byte {
    if from < 0 {
        from += b.Len()  // 如果起始位置为负数，则转换为相对于结束位置的偏移量
    }
    if to < 0 {
        to += b.Len()  // 如果结束位置为负数，则转换为相对于结束位置的偏移量
    }
    return b.v[b.start+from : b.start+to]  // 返回指定范围内的字节数据
}

// BytesFrom returns a slice of this Buffer starting from the given position.
func (b *Buffer) BytesFrom(from int32) []byte {
    if from < 0 {
        from += b.Len()  // 如果起始位置为负数，则转换为相对于结束位置的偏移量
    }
    return b.v[b.start+from : b.end]  // 返回从指定位置开始到结束位置的字节数据
}

// BytesTo returns a slice of this Buffer from start to the given position.
// BytesTo returns a slice of bytes from the buffer, starting from the beginning and ending at the specified position.
func (b *Buffer) BytesTo(to int32) []byte {
    // If the specified position is negative, calculate the actual position from the end of the buffer
    if to < 0 {
        to += b.Len()
    }
    return b.v[b.start : b.start+to]
}

// Resize cuts the buffer at the given positions, updating the start and end positions accordingly.
func (b *Buffer) Resize(from, to int32) {
    // If the 'from' position is negative, calculate the actual position from the end of the buffer
    if from < 0 {
        from += b.Len()
    }
    // If the 'to' position is negative, calculate the actual position from the end of the buffer
    if to < 0 {
        to += b.Len()
    }
    // Check if the 'to' position is less than the 'from' position, and panic if true
    if to < from {
        panic("Invalid slice")
    }
    // Update the end position of the buffer
    b.end = b.start + to
    // Update the start position of the buffer
    b.start += from
}

// Advance moves the start position of the buffer by the specified amount.
func (b *Buffer) Advance(from int32) {
    // If the specified amount is negative, calculate the actual position from the end of the buffer
    if from < 0 {
        from += b.Len()
    }
    // Update the start position of the buffer
    b.start += from
}

// Len returns the length of the buffer content.
func (b *Buffer) Len() int32 {
    // Check if the buffer is nil, and return 0 if true
    if b == nil {
        return 0
    }
    // Return the length of the buffer content
    return b.end - b.start
}

// IsEmpty returns true if the buffer is empty.
func (b *Buffer) IsEmpty() bool {
    // Check if the length of the buffer content is 0, and return true if true
    return b.Len() == 0
}

// IsFull returns true if the buffer has no more room to grow.
func (b *Buffer) IsFull() bool {
    // Check if the buffer is not nil and the end position is equal to the length of the buffer, and return true if true
    return b != nil && b.end == int32(len(b.v))
}

// Write implements the Write method in io.Writer, writing the given data into the buffer.
func (b *Buffer) Write(data []byte) (int, error) {
    // Copy the given data into the buffer, starting from the current end position
    nBytes := copy(b.v[b.end:], data)
    // Update the end position of the buffer
    b.end += int32(nBytes)
    return nBytes, nil
}

// WriteByte writes a single byte into the buffer.
func (b *Buffer) WriteByte(v byte) error {
    // Check if the buffer is full, and return an error if true
    if b.IsFull() {
        return newError("buffer full")
    }
    // Write the single byte into the buffer at the current end position
    b.v[b.end] = v
    // Update the end position of the buffer
    b.end++
    return nil
}

// WriteString implements io.StringWriter, writing the given string into the buffer.
func (b *Buffer) WriteString(s string) (int, error) {
    // Call the Write method with the byte representation of the given string
    return b.Write([]byte(s))
}

// Read implements io.Reader.Read(), reading data from the buffer into the given data slice.
func (b *Buffer) Read(data []byte) (int, error) {
    // Check if the buffer is empty, and return io.EOF if true
    if b.Len() == 0 {
        return 0, io.EOF
    }
    // Copy data from the buffer into the given data slice, starting from the current start position
    nBytes := copy(data, b.v[b.start:b.end])
    // If the number of bytes copied is equal to the length of the buffer content, clear the buffer; otherwise, update the start position
    if int32(nBytes) == b.Len() {
        b.Clear()
    } else {
        b.start += int32(nBytes)
    }
    return nBytes, nil
}

// ReadFrom implements io.ReaderFrom, reading data from the given reader into the buffer.
func (b *Buffer) ReadFrom(reader io.Reader) (int64, error) {
    // Read data from the given reader into the buffer, starting from the current end position
    n, err := reader.Read(b.v[b.end:])
    // Update the end position of the buffer
    b.end += int32(n);
    # 返回一个64位整数和错误信息
    return int64(n), err
// ReadFullFrom 从给定的 reader 中读取指定大小的字节，直到发生错误为止
func (b *Buffer) ReadFullFrom(reader io.Reader, size int32) (int64, error) {
    // 计算读取结束位置
    end := b.end + size
    // 如果结束位置超出了缓冲区大小，则返回错误
    if end > int32(len(b.v)) {
        v := end
        return 0, newError("out of bound: ", v)
    }
    // 从 reader 中读取数据到缓冲区中
    n, err := io.ReadFull(reader, b.v[b.end:end])
    // 更新缓冲区的结束位置
    b.end += int32(n)
    // 返回读取的字节数和可能的错误
    return int64(n), err
}

// String 返回该缓冲区的字符串形式
func (b *Buffer) String() string {
    return string(b.Bytes())
}

// 创建一个大小为 2K 的缓冲区池
var pool = bytespool.GetPool(Size)

// New 创建一个长度为 0，容量为 2K 的缓冲区
func New() *Buffer {
    return &Buffer{
        v: pool.Get().([]byte),
    }
}

// StackNew 在堆栈上创建一个新的缓冲区对象
// 该方法用于在同一函数中释放缓冲区
func StackNew() Buffer {
    return Buffer{
        v: pool.Get().([]byte),
    }
}
```