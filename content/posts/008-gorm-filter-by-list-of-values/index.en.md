+++
title = 'Filtering by a list of values with GORM'
date = 2020-04-07T09:00:00+03:00
draft = false
tags = ['gorm', 'sql','postgresql']
url = '/en/post/gorm-filter-by-list-of-values.html'

[quiz]
  [[quiz.questions]]
    question = "What PostgreSQL operator should be used to filter by a list of values?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "ANY operator"
      correct = true
    [[quiz.questions.answers]]
      text = "IN operator"
      correct = false
    [[quiz.questions.answers]]
      text = "EXISTS operator"
      correct = false
  
  [[quiz.questions]]
    question = "What function converts a Go slice to PostgreSQL array format?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "pq.Array()"
      correct = true
    [[quiz.questions.answers]]
      text = "pq.Slice()"
      correct = false
    [[quiz.questions.answers]]
      text = "pq.Convert()"
      correct = false
  
  [[quiz.questions]]
    question = "Why is the ANY operator with pq.Array() approach efficient?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "It works well with GORM's query builder and is efficient"
      correct = true
    [[quiz.questions.answers]]
      text = "It's faster than SQL"
      correct = false
    [[quiz.questions.answers]]
      text = "It doesn't require a database connection"
      correct = false
+++

When you need to filter data by a list of values (for example, IDs: 1, 2, 3), you should use the `ANY` operator combined with `pq.Array` from the PostgreSQL driver.

<!--more-->

## Solution

Here's how to filter records by a list of IDs:

```go
var priceIDs = []uint{1, 2, 3}
if err := db.Model(prices).Where("id = ANY(?)", pq.Array(priceIDs)).Find(&prices).Error; err != nil {
    return err
}
```

The `pq.Array()` function converts a Go slice into a PostgreSQL array format that can be used with the `ANY` operator. This approach is efficient and works well with GORM's query builder.