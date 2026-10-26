# Binary Serialization in Go (encoding/binary)

In Go, serialize binary data with the standard library’s `encoding/binary` package: `binary.Write` and `binary.Read` handle fixed-size structs, and `binary.BigEndian` or `binary.LittleEndian` set the byte order. The struct fields must have fixed sizes such as `int32` or `float64`. Variable-size data like `[]byte`, `string` or plain `int` needs a length prefix that you write yourself.


It's particularly useful when working with:

- Binary file formats and network protocols.
    
- Embedded systems and hardware registers.
    
- Custom serialization formats.
    
- Data that has a specific byte order, such as big-endian or little-endian.
    
- Reading and writing binary headers, packet fields, and numeric values.
    

The key idea is that computers represent integers as bytes, but a protocol or file format must specify how those bytes are arranged.

## 1. Byte order: the foundation

Consider the 32-bit integer:

go

```
var n uint32 = 0x12345678
```

### Big-endian (most significant byte first)


```
Integer: 0x12345678

Byte order:
+------+------+------+------+
|  12  |  34  |  56  |  78  |
+------+------+------+------+
   MSB                 LSB
```

| Byte 1 | Byte 2 | Byte 3 | Byte 4 |
| ------ | ------ | ------ | ------ |
| `12`   | `34`   | `56`   | `78`   |

### Little-endian (least significant byte first)

text

```
Integer: 0x12345678

Byte order:
+------+------+------+------+
|  78  |  56  |  34  |  12  |
+------+------+------+------+
   LSB                 MSB
```

| Byte 1 | Byte 2 | Byte 3 | Byte 4 |
| ------ | ------ | ------ | ------ |
| `78`   | `56`   | `34`   | `12`   |

Common in x86 data formats

The number is the same in both cases. Only its representation in memory or in a serialized byte stream differs.

In Go, `encoding/binary` exposes byte order through two main values:

- `binary.BigEndian`
    
- `binary.LittleEndian`
    

You choose the byte order explicitly rather than relying on the machine's native endianness.

## 2. Reading and writing individual numbers

The simplest API is `ByteOrder`'s `Uint16`, `Uint32`, `Uint64` and corresponding `PutUint*` methods.


```go
package main

import (
    "encoding/binary"
    "fmt"
)

func main() {
    buf := make([]byte, 4)

    binary.BigEndian.PutUint32(buf, 0x12345678)
    fmt.Printf("% x\n", buf)
    // 12 34 56 78

    n := binary.BigEndian.Uint32(buf)
    fmt.Printf("%08x\n", n)
    // 12345678
}
```

What's happening?

- `make([]byte, 4)` allocates four bytes.
    
- `PutUint32` encodes the number into those bytes.
    
- `Uint32` reconstructs the number from the bytes.
    

The destination slice must be long enough. For example, `PutUint32` needs at least four bytes, and `Uint32` reads four bytes. If the slice is too short, these methods panic.

You can also work with offsets into a larger packet:

```go
packet := make([]byte, 8)

binary.BigEndian.PutUint16(packet[0:2], 0x1234)
binary.BigEndian.PutUint32(packet[2:6], 0x56789abc)
binary.BigEndian.PutUint16(packet[6:8], 0xdef0)

fmt.Printf("% x\n", packet)
// 12 34 56 78 9a bc de f0
```

This is often the most straightforward way to implement a binary protocol when you know the exact layout of each field.

## 3. Encoding and decoding structs

Define a struct of fixed-size fields, write it to a `bytes.Buffer`, and read it back with `binary.Read`.

```go
type Rec struct {
    ID    int32
    Value float64
}

var buf bytes.Buffer
if err := binary.Write(&buf, binary.BigEndian, Rec{ID: 123, Value: 45.67}); err != nil {
    log.Fatal(err)
}

var r Rec
if err := binary.Read(bytes.NewReader(buf.Bytes()), binary.BigEndian, &r); err != nil {
    log.Fatal(err)
}
```

The buffer holds 12 bytes, `0000007b4046d5c28f5c28f6` in hex: 4 for the ID and 8 for the IEEE 754 double. After the read, `r` equals `{123 45.67}`.

## How do I serialize variable-length data?

`binary.Write` rejects a struct containing a slice or string. It returns an error like `some values are not fixed-sized`. Write a length prefix and then the bytes yourself:

```go
func pack(id uint32, name string) []byte {
    b := binary.BigEndian.AppendUint32(nil, id)
    b = binary.BigEndian.AppendUint16(b, uint16(len(name)))
    return append(b, name...)
}

func unpack(d []byte) (uint32, string, error) {
    if len(d) < 6 {
        return 0, "", errors.New("short input")
    }
    n := int(binary.BigEndian.Uint16(d[4:6]))
    if len(d) != 6+n {
        return 0, "", errors.New("bad length")
    }
    return binary.BigEndian.Uint32(d[:4]), string(d[6:]), nil
}
```

`pack(7, "Ada")` returns `000000070003416461` in hex, and `unpack` gives back `7` and `"Ada"`. `AppendUint32` needs Go 1.19 or later.

## Example

This wont work

```go
type Data struct {
    ID   uint32
    Name string
    Data []byte
}

err := binary.Write(&buf, binary.BigEndian, data)
```

We can do this 

```go
package main

import (
    "bytes"
    "encoding/binary"
    "fmt"
    "io"
)

type Data struct {
    ID   uint32
    Name string
    Data []byte
}

func (d *Data) MarshalBinary() ([]byte, error) {
    var buf bytes.Buffer

    // Fixed-size field
    if err := binary.Write(&buf, binary.BigEndian, d.ID); err != nil {
        return nil, err
    }

    // String: uint32 length + string bytes
    if err := binary.Write(&buf, binary.BigEndian, uint32(len(d.Name))); err != nil {
        return nil, err
    }
    buf.WriteString(d.Name)

    // Slice: uint32 length + slice bytes
    if err := binary.Write(&buf, binary.BigEndian, uint32(len(d.Data))); err != nil {
        return nil, err
    }
    buf.Write(d.Data)

    return buf.Bytes(), nil
}

func (d *Data) UnmarshalBinary(b []byte) error {
    r := bytes.NewReader(b)

    if err := binary.Read(r, binary.BigEndian, &d.ID); err != nil {
        return err
    }

    var nameLen uint32
    if err := binary.Read(r, binary.BigEndian, &nameLen); err != nil {
        return err
    }
    if uint64(nameLen) > uint64(r.Len()) {
        return io.ErrUnexpectedEOF
    }

    name := make([]byte, int(nameLen))
    if _, err := io.ReadFull(r, name); err != nil {
        return err
    }
    d.Name = string(name)

    var dataLen uint32
    if err := binary.Read(r, binary.BigEndian, &dataLen); err != nil {
        return err
    }
    if uint64(dataLen) > uint64(r.Len()) {
        return io.ErrUnexpectedEOF
    }

    d.Data = make([]byte, int(dataLen))
    if _, err := io.ReadFull(r, d.Data); err != nil {
        return err
    }

    return nil
}

func main() {
    original := Data{
        ID:   42,
        Name: "hello",
        Data: []byte{1, 2, 3, 4},
    }

    encoded, err := original.MarshalBinary()
    if err != nil {
        panic(err)
    }

    var decoded Data
    if err := decoded.UnmarshalBinary(encoded); err != nil {
        panic(err)
    }

    fmt.Printf("%+v\n", decoded)
}
```

## Use encoding/gob instead

`encoding/gob` encodes arbitrary Go types, including strings and slices, but only Go programs can read it. Use it for Go-to-Go traffic. For other languages, define an explicit layout like the one above. 

```go
package main

import (
    "bytes"
    "encoding/gob"
    "fmt"
)

type Data struct {
    ID   uint32
    Name string
    Data []byte
}

func main() {
    original := Data{42, "hello", []byte{1, 2, 3}}

    var buf bytes.Buffer

    err := gob.NewEncoder(&buf).Encode(original)
    if err != nil {
        panic(err)
    }

    var decoded Data
    err = gob.NewDecoder(&buf).Decode(&decoded)
    if err != nil {
        panic(err)
    }

    fmt.Printf("%+v\n", decoded)
}
```


## The new functions

Starting in Go 1.23, the `encoding/binary` package added three functions: `binary.Encode`, `binary.Decode`, and `binary.Append`. They operate on byte slices rather than `io.Writer` and `io.Reader`.

```go
n, err := binary.Encode(buf, binary.BigEndian, value)
n, err := binary.Decode(buf, binary.BigEndian, &value)

buf, err := binary.Append(buf, binary.BigEndian, value)
```

- `Encode` writes a value into an existing `[]byte` buffer.
- `Decode` reads a value from a `[]byte` buffer into a destination.
- `Append` appends the encoded value to a buffer, allocating more capacity if necessary.

Both `Encode` and `Decode` return the number of bytes written or consumed, plus an error if the operation fails. The buffer must be large enough for `Encode`.

## Example with a struct

These functions work with fixed-size structs, but they do not add automatic support for strings or variable-length slices.

```go
package main

import (
    "encoding/binary"
    "fmt"
)

type Header struct {
    ID     uint32
    Length uint16
}

func main() {
    h := Header{
        ID:     123,
        Length: 256,
    }

    buf := make([]byte, binary.Size(h))

    n, err := binary.Encode(buf, binary.BigEndian, h)
    if err != nil {
        panic(err)
    }

    fmt.Printf("Encoded (%d bytes): %x\n", n, buf)

    var decoded Header
    n, err = binary.Decode(buf, binary.BigEndian, &decoded)
    if err != nil {
        panic(err)
    }

    fmt.Printf("Decoded (%d bytes): %+v\n", n, decoded)
}
```

## What about structs containing strings or slices?

```go
type Message struct {
    ID   uint32
    Name string
    Data []byte
}

msg := Message{ID: 1, Name: "hello", Data: []byte{1, 2, 3}}

buf := make([]byte, 100)
_, err := binary.Encode(buf, binary.BigEndian, msg)
// Error: string and slice fields are not fixed-size.
```

You must encode the variable-length fields separately, for example using a length prefix and raw bytes. `binary.Append` is convenient for this:

```go
buf, err := binary.Append(nil, binary.BigEndian, msg.ID)
if err != nil {
    panic(err)
}

buf, err = binary.Append(buf, binary.BigEndian, uint32(len(msg.Name)))
if err != nil {
    panic(err)
}
buf = append(buf, msg.Name...)

buf, err = binary.Append(buf, binary.BigEndian, uint32(len(msg.Data)))
if err != nil {
    panic(err)
}
buf = append(buf, msg.Data...)
```

For decoding, use `binary.Decode` to read each fixed-size field and then slice the input buffer to read the corresponding variable-length data. Always validate the lengths against the remaining buffer before slicing.

Bottom line: Go 1.23 makes binary serialization more convenient, especially with `binary.Append`, but `Encode` and `Decode` still require fixed-size data. They are not general-purpose struct serializers.

-----
## some common mistakes?

- **Using `int`.** Its size depends on the platform, so `binary.Write` refuses it. Use `int32` or `int64`.
- **Ignoring errors.** Check the error from every `Write` and `Read`.
- **Trusting a length field.** Verify it against the remaining bytes before slicing or allocating.
- **Mixed byte order.** Pick one order for the whole format. Network protocols conventionally use big-endian.

-----
## Another Example

Suppose a binary header contains:

- A 16-bit magic number.
    
- A 16-bit version.
    
- A 32-bit payload length.
    

```go
type Header struct {
    Magic   uint16
    Version uint16
    Length  uint32
}
```

Encode it:


```go
var buf bytes.Buffer

h := Header{
    Magic:   0xCAFE,
    Version: 1,
    Length: 1024,
}

err := binary.Write(&buf, binary.BigEndian, h)
if err != nil {
    log.Fatal(err)
}

fmt.Printf("% x\n", buf.Bytes())
// ca fe 00 01 00 00 04 00
```

Decode it:


```go
var decoded Header

err := binary.Read(
    bytes.NewReader(buf.Bytes()),
    binary.BigEndian,
    &decoded,
)
if err != nil {
    log.Fatal(err)
}

fmt.Printf("%+v\n", decoded)
// {Magic:51966 Version:1 Length:1024}
```

Notice that `binary.Read` receives a pointer to the destination struct, while `binary.Write` accepts the struct value.

### Which struct fields are supported?

`encoding/binary` works with fixed-size types, including:

- `uint8`, `uint16`, `uint32`, `uint64`
    
- `int8`, `int16`, `int32`, `int64`
    
- `float32`, `float64`
    
- `bool`
    
- Arrays of fixed-size types.
    
- Structs containing only fixed-size fields.
    

There are some important restrictions:

- `string`, slices of variable length, maps, and pointers inside your struct aren't directly serialized as fields.
    
- Struct fields must be exported (capitalized) when using `Read` and `Write`; unexported fields can cause a panic.
    
- Struct layout is determined by field order, not field names or JSON-style tags.
    
- Blank fields named `_` can represent padding bytes.
    

For example, this isn't directly supported as a complete binary record:


```go
type Message struct {
    Length uint16
    Text   string // variable-length; not supported by binary.Write
}
```

You need to define how the string is represented, such as a length prefix followed by UTF-8 bytes.

## 4. Choosing between the main APIs

### ByteOrder methods

Uint32 / PutUint32

Individual numeric fields; you manage offsets and bounds.

### Read / Write

binary.Read / binary.Write

Structured records through io.Reader and io.Writer.

### Decode / Encode

binary.Decode / binary.Encode

Go 1.23+; decode from or encode into an existing byte slice.

### Append

binary.Append

Go 1.23+; append encoded values to a growing byte slice.

### Varints

binary.Varint / Uvarint

Compact variable-length integer encoding.

The newer `Encode`, `Decode`, and `Append` APIs were added in Go 1.23. If you're targeting an older Go version, use the traditional methods.

For a protocol implementation, I'd usually start with `ByteOrder` methods for precise control or `Read`/`Write` for convenient fixed-size records.

## 5. Floating-point numbers

Floating-point values are encoded using their IEEE 754 bit representation. `encoding/binary` handles this for you.


```go
f := float32(3.14)

bits := math.Float32bits(f)
buf := make([]byte, 4)

binary.LittleEndian.PutUint32(buf, bits)

fmt.Printf("% x\n", buf)

// Decode
decodedBits := binary.LittleEndian.Uint32(buf)
decoded := math.Float32frombits(decodedBits)

fmt.Println(decoded) // approximately 3.14
```

For `float64`, use `math.Float64bits` and `math.Float64frombits` alongside `Uint64` and `PutUint64`.

You'll need the `math` package for these conversions. Alternatively, `binary.Write` and `binary.Read` can encode and decode floating-point values directly.

## 6. Varints: integers that use fewer bytes

So far, we've dealt with fixed-width integers. A `uint32` always takes four bytes, even if its value is just `1`.

A varint uses a variable number of bytes, so small values can take less space. The package provides:

- `binary.PutUvarint` and `binary.Uvarint` for unsigned integers.
    
- `binary.PutVarint` and `binary.Varint` for signed integers.
    

Here's an example:


```go
buf := make([]byte, binary.MaxVarintLen64)

n := binary.PutUvarint(buf, 300)
encoded := buf[:n]

fmt.Printf("% x\n", encoded)
// ac 02

value, bytesRead := binary.Uvarint(encoded)

fmt.Println(value)     // 300
fmt.Println(bytesRead) // 2
```

Compare that with a fixed-width `uint64`, which always takes eight bytes.

A few details matter:

- `PutUvarint` requires a sufficiently large buffer; otherwise it panics.
    
- `Uvarint` returns both the decoded value and the number of bytes consumed.
    
- A return count of `0` means the input is too short to decode a complete varint; a negative count means the encoding is too long or overflows.
    
- Signed varints use a specific encoding scheme. They aren't simply fixed-width two's-complement integers with leading zeros removed.
    

Varints are useful for compact formats, but they're not interchangeable with fixed-width fields. Use them only when your file format or protocol specifies them.

## 7. A practical example: building a binary packet

Let's put the concepts together. Imagine a simple packet format:

|Offset|Size|Field|
|---|---|---|
|0|1 byte|Message type|
|1|4 bytes|Sequence number|
|5|2 bytes|Payload length|
|7|Variable|Payload|

All multibyte fields use big-endian order.


```go
package main

import (
    "encoding/binary"
    "errors"
    "fmt"
)

func EncodePacket(msgType byte, seq uint32, payload []byte) ([]byte, error) {
    if len(payload) > 65535 {
        return nil, errors.New("payload too large")
    }

    packet := make([]byte, 7+len(payload))

    packet[0] = msgType
    binary.BigEndian.PutUint32(packet[1:5], seq)
    binary.BigEndian.PutUint16(packet[5:7], uint16(len(payload)))
    copy(packet[7:], payload)

    return packet, nil
}

func DecodePacket(packet []byte) (byte, uint32, []byte, error) {
    if len(packet) < 7 {
        return 0, 0, nil, errors.New("packet too short")
    }

    payloadLen := int(binary.BigEndian.Uint16(packet[5:7]))

    if len(packet) != 7+payloadLen {
        return 0, 0, nil, errors.New("invalid payload length")
    }

    msgType := packet[0]
    seq := binary.BigEndian.Uint32(packet[1:5])
    payload := packet[7:]

    return msgType, seq, payload, nil
}

func main() {
    packet, err := EncodePacket(1, 42, []byte("hello"))
    if err != nil {
        panic(err)
    }

    fmt.Printf("% x\n", packet)
    // 01 00 00 00 2a 00 05 68 65 6c 6c 6f

    typ, seq, payload, err := DecodePacket(packet)
    if err != nil {
        panic(err)
    }

    fmt.Println(typ, seq, string(payload))
    // 1 42 hello
}
```

This example demonstrates an important distinction: `encoding/binary` handles the numeric fields, while you define the packet format, including where each field lives and how variable-length data is represented.

For production code, also consider maximum packet sizes, resource limits, and whether trailing bytes are allowed by the protocol.

## 8. Common pitfalls

### Mixing up byte order

Decoding little-endian bytes as big-endian produces a different number, without necessarily producing an error. Always follow the format specification.

### Assuming Go struct memory layout is the wire format

`binary.Write` serializes fields in declaration order; it does not simply dump the struct's in-memory representation. Go memory alignment and padding are not a substitute for an explicit protocol specification.

### Forgetting that strings need a format

A string is a sequence of bytes, not a fixed-width binary field. Common approaches include:

- A fixed-width byte array.
    
- A length prefix followed by the string bytes.
    
- A delimiter, if the format defines escaping and termination rules.
    

### Ignoring errors

`binary.Read` can return `io.EOF` if no bytes were available, or `io.ErrUnexpectedEOF` if a record was only partially read. `binary.Write` can also return errors from the underlying writer.

### Using `int` for portable file formats

Prefer explicit-width types such as `uint16`, `uint32`, and `uint64` for serialized fields. The width of Go's `int` depends on the target architecture, so it is not a good choice for a portable wire format.

### Confusing binary encoding with serialization frameworks

`encoding/binary` is intentionally low-level. It doesn't automatically encode arbitrary Go objects, discover schemas, or handle versioning. For richer data models, consider JSON, Gob, or Protocol Buffers, depending on your requirements.

## 9. Quick reference

| Goal                                        | API                                   |
| ------------------------------------------- | ------------------------------------- |
| Read a 16-bit integer                       | `binary.BigEndian.Uint16(b)`          |
| Write a 32-bit integer                      | `binary.LittleEndian.PutUint32(b, n)` |
| Encode a fixed-size struct                  | `binary.Write(w, order, v)`           |
| Decode a fixed-size struct                  | `binary.Read(r, order, &v)`           |
| Encode into an existing slice (Go 1.23+)    | `binary.Encode(buf, order, v)`        |
| Append encoded data (Go 1.23+)              | `binary.Append(buf, order, v)`        |
| Encode a compact unsigned integer           | `binary.PutUvarint(buf, n)`           |
| Decode a compact unsigned integer           | `binary.Uvarint(buf)`                 |
| Determine a fixed-size value's encoded size | `binary.Size(v)`                      |

---


## Frequently Asked Questions

### Why does binary.Write fail on my struct?

It contains a variable-size field such as a slice, string or `int`. Use fixed-size types or a manual length prefix.

### Which byte order should I choose?

Big-endian for network formats; little-endian if you match an existing file format. Document it either way.

### How big is my struct on the wire?

Call `binary.Size(value)`. It returns the encoded size, or -1 for non-fixed types.

### Is encoding/binary safe for untrusted input?

Yes for fixed layouts, provided you validate lengths before you allocate.