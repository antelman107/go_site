+++
title = 'GORM: filter entity by linked entity'
date = 2020-03-30T09:00:00+03:00
draft = false
tags = ['gorm', 'sql', 'postgresql']
url = '/en/post/gorm-filter-entity-by-linked-entity.html'

[quiz]
  [[quiz.questions]]
    question = "Why doesn't a simple JOIN work for filtering parents by linked children when parents have multiple matching children?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "It causes parents to be repeated multiple times in the results"
      correct = true
    [[quiz.questions.answers]]
      text = "JOIN operations are not supported in GORM"
      correct = false
    [[quiz.questions.answers]]
      text = "It causes a database error"
      correct = false
  
  [[quiz.questions]]
    question = "What is the recommended solution for filtering entities by linked entities in GORM?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Using EXISTS subquery in WHERE clause"
      correct = true
    [[quiz.questions.answers]]
      text = "Using LEFT JOIN"
      correct = false
    [[quiz.questions.answers]]
      text = "Using INNER JOIN"
      correct = false
  
  [[quiz.questions]]
    question = "What does the EXISTS subquery return?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "It returns true if the subquery finds at least one matching row"
      correct = true
    [[quiz.questions.answers]]
      text = "It returns the matching rows"
      correct = false
    [[quiz.questions.answers]]
      text = "It returns the count of matching rows"
      correct = false
+++

This task is not that simple, especially with GORM.

<!--more-->

Take a look at the following entities:
```go
type Parent struct {
gorm.Model
Name string
}

type Child struct {
gorm.Model
ParentID uint // child -> parent many to one link
Name     string
}
```

How do I filter parents by filtering linked children?

The general solution in SQL-related databases is to use a simple JOIN and then filter. This is possible with GORM:

```go
err := db.Model(&Parent{}).Joins(
"LEFT JOIN children ON parents.id = children.parent_id AND children.name = ?",
"somename",
).Find(&results).Error
```

The full query:

```sql
SELECT * FROM "parents" LEFT JOIN children ON parents.id = children.parent_id AND children.name = 'somename'
```

But this does not generally work for our task — for parents who have several matching children. In that case we'll have such parents repeated multiple times.

So, a simple JOIN probably does not work here. We probably need a solution based on a WHERE condition. The following code works perfectly:

```go
err := db.Model(&Parent{}).Where(
"EXISTS (SELECT 1 FROM children WHERE children.parent_id = parents.id AND children.name = ?)",
"somename",
).Find(&results).Error
```

The full query looks as follows:

```sql
SELECT * FROM "parents" WHERE EXISTS (SELECT 1 FROM children WHERE children.parent_id = parents.id AND children.name = 'somename')
```
