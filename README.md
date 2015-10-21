dlug
===

Dlugosz based variable length integer encoding scheme.

Quick Setup
---

```go
$ go get github.com/abeconnelly/dlug
$ cat <<EOF > dlug_example.go
package main

import "fmt"
import "github.com/abeconnelly/dlug"

func main() {
  var x0, x1 uint64

  x0 = 120
  x1 = 250

  byte_slice0 := dlug.MarshalUint64(x0)
  byte_slice1 := dlug.MarshalUint64(x1)

  fmt.Printf("Value: %d, Bytes: %v (%d byte(s))\n", x0, byte_slice0, len(byte_slice0))
  fmt.Printf("Value: %d, Bytes: %v (%d byte(s))\n", x1, byte_slice1, len(byte_slice1))

}
EOF
$ go run dlug_example.go
Value: 120, Bytes: [120] (1 byte(s))
Value: 250, Bytes: [128 250] (2 byte(s))
```


Description
---

Dlugosz's variable length encoding (VLE)
has a linear number of prefix bits encode the number of
bytes until a cutoff at which time it switches over to the prefix
bits describing the length of the VLE integer.  The VLE integer encoding
is slightly modified from the one presented in the Dlugosz VLI post.

The following table gives a sense for how to encode:

| prefix | bytes | header bits | data bits | unsigned range |
|---|---|---|---|---|
| 0 | 1 | 1 | 7 | 127 |
| 10 | 2 | 2 | 14 | 16,383 |
| 110 | 3 | 3 | 21 | 2,097,151 |
| 111 00 | 4 | 5 | 27 | 134,217,727 |
| 111 01 | 5 | 5 | 35 | 34,359,738,367 |
| 111 10 | 6 | 5 | 43 | 8,796,093,022,207 |
| 111 11 000 | 8 | 8 | 56 | 72,057,594,037,927,935 |
| 111 11 001 |  9 | 8 | 64 | A full 64-bit value with one byte overhead |
| 111 11 010 | 17 | 8 | 128 | A GUID/UUID |
| 111 11 111 |  n | 8 | any | Any multi-precision integer |

This is a nice compromise between arbitrary length and efficient encoding
for small numbers.

The document by Dlugosz is unclear about what happens in the 0xff case for the prefix byte.
This will be resolved here by considering the next 8 bytes to encode the length
of the subsequent bytes.  So

    0xff | 0xgh 0xij 0xkl 0xmn 0xop 0xqr 0xst 0xuv [ ... ]

where [g-v] hold the number of bytes in [...].  The value (2^64)-1 in the length
field is reserved for future use.

References
---

  - [Dlugosz Variable Length Integer](http://www.dlugosz.com/ZIP2/VLI.html)
