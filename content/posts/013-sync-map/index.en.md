+++
title = 'sync.Map'
date = 2020-05-02T09:00:00+03:00
draft = false
tags = ['sync.map', 'map', 'concurrency']
featured_image = 'dvoe.svg'
url = '/en/post/sync-map.html'

[quiz]
  [[quiz.questions]]
    question = "What are the two internal maps in `sync.Map`?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "read - for reading data"
      correct = true
    [[quiz.questions.answers]]
      text = "dirty - for new elements and writing"
      correct = true
    [[quiz.questions.answers]]
      text = "cache - for caching"
      correct = false
  
  [[quiz.questions]]
    question = "Why is `sync.Map` most effective for read-only scenarios?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Reading from 'read' doesn't require a mutex"
      correct = true
    [[quiz.questions.answers]]
      text = "It uses less memory"
      correct = false
    [[quiz.questions.answers]]
      text = "It's faster than regular maps"
      correct = false
  
  [[quiz.questions]]
    question = "What happens when the 'misses' counter exceeds the number of elements in 'dirty'?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Elements from 'dirty' are copied to 'read' and counter is reset"
      correct = true
    [[quiz.questions.answers]]
      text = "The map is cleared"
      correct = false
    [[quiz.questions.answers]]
      text = "An error is thrown"
      correct = false
  
  [[quiz.questions]]
    question = "When does `sync.Map` perform worse than `map+sync.RWMutex`?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "When there is both reading and updating"
      correct = true
    [[quiz.questions.answers]]
      text = "When there is only reading"
      correct = false
    [[quiz.questions.answers]]
      text = "When the map is empty"
      correct = false
+++

Let's take a look at `sync.Map` usage and its source code.

<!--more-->

## Interface

The following methods are available to work with `sync.Map`:

- `Load(key interface{}) (value interface{}, ok bool)`
- `Store(key, value interface{})`
- `Delete(key interface{})`
- `Range(f func(key, value interface{}) bool)`
- `LoadOrStore(key, value interface{}) (actual interface{}, loaded bool)`

This covers every map use case — insert, read, delete, and iteration. There is also the `LoadOrStore` bonus method, which allows setting a value by key if the value is not set before.

To perform iteration, there is `Range`, which receives a function that is called on every map element. If that function returns `false`, iteration will be stopped.

When seeing the `sync.Map` name, one might imagine that `sync.Map` is a simple map with some `sync` package features. But actually `sync.Map` is much more complex. Let's dive into the source code and find out how it works.

## Sources

The `sync.Map` struct looks as below:

```go
type Map struct {
	mu sync.Mutex

	read atomic.Value
	dirty map[interface{}]*entry
	misses int
}

type readOnly struct {
	m       map[interface{}]*entry
	amended bool
}
```

`read` — a pointer to a `readOnly` struct. This struct stores part of the data that is used to read data by key or to check that there is data behind that key. There are no inserts into `read`, that's why no mutex is used when accessing `read`.

`dirty` — a map that stores the other part of elements. It is generally used to place new elements and for reading too. In that case, the `mu` mutex is used when `dirty` is accessed.

So `sync.Map` has two internal maps (`read` and `dirty`). By using one map only for reading and a second for writing, `sync.Map` tries to avoid using a mutex when possible. Let's take a look at `sync.Map` operations.

## Reading

First, `read` is accessed to check if there is data behind the key. If it is — the data is returned. This is the simplest scenario — to return data from `read`.

Then `dirty` is searched — the `mu` mutex is activated beforehand. `misses` — a counter of accesses to `dirty` — is incremented. An important thing — if that counter's value is more than the number of elements in `dirty`, elements in `dirty` will be copied to `read`, and the counter will be reset. The copying will happen right after `dirty` access — `mu` is activated at that moment.

To avoid extra `dirty` access, there is an `amended` field in the `readOnly` struct (which is `read`). `amended = true` means there is a non-empty `dirty` map in `sync.Map`.

So, reading from `sync.Map` is most effective when read-only. In this case, this reading is equal to simple map (without mutex) reading. If there is only reading from a large map set up beforehand, `sync.Map` is good in that case.

## Updating

First, there is a check against `read` — if there is data behind the key. If yes — the value is updated by `atomic.CompareAndSwap`.

If there is no data read by the key, the same reading is performed against `dirty`. If `dirty` is not initialized — it will be.

So, updating an already existent key is simple, but adding data with a new key is complex.

## Iterating

Before iterating, `dirty` is copied into `read`, if `dirty` stores anything. This is done with the `mu` mutex activated.

Next, there is `read` iteration — simple map iteration.

## Summary

`sync.Map` is a complex struct, generally consisting of two maps — one for reading and one for new elements. `sync.Map` is not only close to `map+sync.RWMutex` on speed metrics, but can also be the best in the read-only case. When there is both reading and updating, `sync.Map` will have elements in both `read` and `dirty`. In that case, `sync.Map` performs worse than `map+sync.RWMutex` because of reading from two internal maps. Also, there is a counter that needs to be updated when accessing `dirty`.

