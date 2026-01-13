+++
title = 'Principles of slice type in GO'
date = 2020-04-04T09:00:00+03:00
draft = false
tags = ['slice', 'allocation','sources']
featured_image = 'gophslice.svg'
url = '/en/post/golang-slice.html'

[quiz]
  [[quiz.questions]]
    question = "What is the main difference between a slice and an array in Go?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "A slice is a pointer to an array"
      correct = true
    [[quiz.questions.answers]]
      text = "A slice's size can be changed"
      correct = true
    [[quiz.questions.answers]]
      text = "Arrays are faster than slices"
      correct = false
  
  [[quiz.questions]]
    question = "What happens to the underlying array when a slice's capacity changes during append?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "The data is copied to a new array"
      correct = true
    [[quiz.questions.answers]]
      text = "The array is extended in place"
      correct = false
    [[quiz.questions.answers]]
      text = "Nothing happens"
      correct = false
  
  [[quiz.questions]]
    question = "How does slice capacity grow when length is less than 1024?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Memory size doubles"
      correct = true
    [[quiz.questions.answers]]
      text = "Memory grows by a quarter"
      correct = false
    [[quiz.questions.answers]]
      text = "Memory grows by a fixed amount"
      correct = false
  
  [[quiz.questions]]
    question = "What happens when you create a new slice from an old slice and the old slice's capacity changes?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "The old underlying array remains in memory"
      correct = true
    [[quiz.questions.answers]]
      text = "The old array is garbage collected immediately"
      correct = false
    [[quiz.questions.answers]]
      text = "Both slices share the same array"
      correct = false
+++

The Go blog describes how to use slices. Let's take a look at slice internals.

<!--more-->

A slice is a data type that wraps an array.

The differences between a slice and an array are:

- A slice is a pointer to an array
- A slice's size can be changed

In the Go sources, a slice is represented by the following structure:

```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

`len` (length) is the current slice length, `cap` (capacity) is the number of elements in the underlying array.

Both fields can be passed as parameters to the `make` function:

```go
s := make(
	[]int,
	10, // len
	10, // cap
)
```

Capacity is the main parameter responsible for memory allocations in slices. It is also responsible for append performance.

## Slice Behavior on Append

Let's take a look at slice behavior when appending:

```go
a := []int{1}
b := a[0:1]
b[0] = 0

fmt.Println(a)
// [0]

a[0] = 1

fmt.Println(b)
// [1]
```

We created slice `b` from slice `a`. As we can see, both slices point to the same underlying array.

Let's add an append operation to one of the slices:

```go
a := []int{1}
b := a[0:1]

a = append(a, 2)
a[0] = 10

fmt.Println(a, b)
// [10 2] [0]
```

After the append, slices have different values in the first element. Now the slices point to different arrays.

One can understand this situation by examining the `growslice` function sources.

On capacity change, the underlying array data is always copied:

```go
memmove(p, old.array, lenmem)
```

Now let's take a look at the actual capacity changes:

```go
newcap := old.cap
doublecap := newcap + newcap
if cap > doublecap {
	newcap = cap
} else {
	if old.len < 1024 {
		newcap = doublecap
	} else {
		// Check 0 < newcap to detect overflow
		// and prevent an infinite loop.
		for 0 < newcap && newcap < cap {
			newcap += newcap / 4
		}
		// Set newcap to the requested cap when
		// the newcap calculation overflowed.
		if newcap <= 0 {
			newcap = cap
		}
	}
}
```

When slice length < 1024, memory size doubles.

Otherwise, the slice grows by a quarter.

## Memory Impact of Appending

Appending to a slice has a significant impact on memory:

- On capacity change, the array will be copied
- Allocated memory will grow according to internal Go logic
- To avoid allocations, one must set the initial slice capacity large enough

In the next example, the only change is the slice capacity. Now after appending to the slice, there is no capacity change. Both slices still point to the same array:

```go
a := make([]int, 1, 2)
a[0] = 1

b := a[0:1]
b[0] = 0

fmt.Println(a, b)
// [0] [0]

a = append(a, 2)
a[0] = 10

fmt.Println(a, b)
// [10 2] [10]
```

The Go blog describes an option to use a different append function. However, the only thing we can do is make even more aggressive allocations than Go does:

```go
func AppendByte(slice []byte, data ...byte) []byte {
	m := len(slice)
	n := m + len(data)
	if n > cap(slice) { // if necessary, reallocate
		// allocate double what's needed, for future growth.
		newSlice := make([]byte, (n+1)*2)
		copy(newSlice, slice)
		slice = newSlice
	}
	slice = slice[0:n]
	copy(slice[m:n], data)
	return slice
}
```

One should be careful when a new slice is created as a part of an old one. The full old underlying array will remain in memory.
