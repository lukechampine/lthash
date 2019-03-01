lthash
------

[![GoDoc](https://godoc.org/lukechampine.com/lthash?status.svg)](https://godoc.org/lukechampine.com/lthash)
[![Go Report Card](http://goreportcard.com/badge/lukechampine.com/lthash)](https://goreportcard.com/report/lukechampine.com/lthash)

```
go get lukechampine.com/lthash
```

This repo contains an implementation of LtHash, as
[defined](https://cseweb.ucsd.edu/~daniele/papers/IncHash.pdf) by
Bellare and Micciancio and later [specified more concretely](https://eprint.iacr.org/2019/227.pdf) 
by researchers at Facebook.

LtHash is a homomorphic hash function based on BLAKE2x. A homomorphic hash
function provides a solution to the following problem: Given the hash of an
input, along with a small update to the input, how can we compute the hash of
the new input with its update applied, without having to recompute the entire
hash from scratch?

For example, say you have a database that contains three items: `"Apple"`,
`"Banana"`, and `"Orange"`. We compute the LtHash of this set by summing the
hashes of the individual elements. Later, we replace `"Banana"` with `"Grape"`.
To compute the new hash, we take our original hash, subtract the hash of
`"Banana"`, and add the hash of `"Grape"`. The resulting hash is the same as if
the database originally contained `"Grape"` instead of `"Banana"`.

This repo currently contains an implementation of `lthash16`. Facebook's paper
further specifies `lthash20` and `lthash32`; these may be added in the future.

Be aware that LtHash is vulnerable to multiset input collisions. A multiset is a
set containing more than one instance of a particular element. In particular, it
is trivial to produce a collision in `lthash16` by adding the same input to the
hash 2^16 times. One way to prevent this is to concatenate each input with a
unique piece of metadata, such as an index.

## Usage

```go
import "lukechampine.com/lthash"

func main() {
    h := lthash.New16()
    
    h.Add([]byte("Apple"))
    h.Add([]byte("Banana"))
    h.Add([]byte("Orange"))
    oldSum := h.Sum(nil)

    h.Remove([]byte("Banana"))
    h.Add([]byte("Grape"))
    newSum := h.Sum(nil)
}
```

## Benchmarks

Tested on an i5-7600K @ 3.80GHz. Results will likely be slower on non-amd64
architectures.

```
BenchmarkLtHash/16-4    200000    9651 ns/op    424.38 MB/s    0 allocs/op
```
