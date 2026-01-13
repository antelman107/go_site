+++
title = 'Principles of map type in GO'
date = 2020-04-02T09:00:00+03:00
draft = false
tags = ['map', 'sources']
featured_image = 'map.svg'
url = '/en/post/map-principles-golang.html'

[quiz]
  [[quiz.questions]]
    question = "What types can be used as map keys in Go?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "Comparable types like simple scalar types"
      correct = true
    [[quiz.questions.answers]]
      text = "Channels"
      correct = true
    [[quiz.questions.answers]]
      text = "Arrays"
      correct = true
    [[quiz.questions.answers]]
      text = "Slices"
      correct = false
    [[quiz.questions.answers]]
      text = "Maps"
      correct = false
  
  [[quiz.questions]]
    question = "What are the requirements for a hash function used in Go maps?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "Deterministic"
      correct = true
    [[quiz.questions.answers]]
      text = "Uniform distribution"
      correct = true
    [[quiz.questions.answers]]
      text = "Fast execution"
      correct = true
    [[quiz.questions.answers]]
      text = "Cryptographically secure"
      correct = false
  
  [[quiz.questions]]
    question = "How many buckets are added when the first key is inserted into an empty map?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "8 buckets"
      correct = true
    [[quiz.questions.answers]]
      text = "1 bucket"
      correct = false
    [[quiz.questions.answers]]
      text = "16 buckets"
      correct = false
  
  [[quiz.questions]]
    question = "Why doesn't Go allow taking the address of a map value?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "During map growth, data addresses can change"
      correct = true
    [[quiz.questions.answers]]
      text = "Map values are immutable"
      correct = false
    [[quiz.questions.answers]]
      text = "It's a language design limitation"
      correct = false
+++

The map programming interface in Go is described in the Go blog. We just need to recall that a map is a key-value storage and it should retrieve values by key as fast as possible.

<!--more-->

Any comparable type can be a key — all simple scalar types, channels, and arrays.
Non-comparable types include slices, maps, and functions.

Map keys and values should be stored in memory allocated for the map, consecutively. To deal with memory addresses, we should use a hash function.

We have the following requirements for a hash function:

- **Deterministic** — should return the same value for the same key
- **Uniform** — values should be somehow equally related to all buckets
- **Fast** — should run quickly to be used many times

## Implementation

Now to the implementation. The simplified map structure is below (from sources):

```go
// A header for a Go map.
type hmap struct {
	count      int // # live cells == size of map.  Must be first (used by len() builtin)
	B          uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
	buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
	oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
}
```

Data addresses are stored divided into parts — in the buckets array.

`unsafe.Pointer` — can be a pointer of any variable type — a generics issue solution (Go developers use it to make keys and values of any type).

Buckets is nil if there is no data in the map. On the first key, 8 buckets are added.

## Getting Data from Map

We need to find the memory address of the key and value.

First, to find the bucket. It is selected by a comparison of the first 8 bits of the key hash with corresponding data stored in the bucket struct.

Next, to find the key value by its address:

```go
k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
if t.indirectkey() {
	k = *((*unsafe.Pointer)(k))
}
```

Next, to find the value the same way:

```go
if t.key.equal(key, k) {
	e := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
	if t.indirectelem() {
		e = *((*unsafe.Pointer)(e))
	}
	return e
}
```

## Map Growing

We work with the `oldbuckets` map struct property during data migration.

Migration starts when there is too much data in buckets. We save the current value of buckets to oldbuckets, then create a doubled amount of buckets in the buckets property. Map data is copied from oldbuckets to buckets.

The amount of buckets is always a power of 2.
That's why there is a B property — it is the power of 2 that shows the current number of buckets.

During migration, the map is accessible. That's why there is a lot of work with both buckets and oldbuckets in the same functions of the source code. After migration, oldbuckets is set to nil.

During migration, the address of data can change, so Go doesn't allow getting a pointer to a map value:

```go
mymap := map[string]string{"1": "1"}
fmt.Println(&mymap["1"])

// cannot take the address of mymap["1"]
```

The great GopherCon 2016 related lecture:
