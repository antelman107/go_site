+++
title = 'GORM: фильтрация сущности по связанной сущности'
date = 2020-03-30T09:00:00+03:00
draft = false
tags = ['gorm', 'sql', 'postgresql']
url = '/ru/post/gorm-filter-entity-by-linked-entity.html'

[quiz]
  [[quiz.questions]]
    question = "Почему простой JOIN не работает для фильтрации родителей по связанным детям, когда у родителей несколько совпадающих детей?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Это приводит к повторению родителей несколько раз в результатах"
      correct = true
    [[quiz.questions.answers]]
      text = "Операции JOIN не поддерживаются в GORM"
      correct = false
    [[quiz.questions.answers]]
      text = "Это вызывает ошибку базы данных"
      correct = false
  
  [[quiz.questions]]
    question = "Какое решение рекомендуется для фильтрации сущностей по связанным сущностям в GORM?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Использование подзапроса EXISTS в условии WHERE"
      correct = true
    [[quiz.questions.answers]]
      text = "Использование LEFT JOIN"
      correct = false
    [[quiz.questions.answers]]
      text = "Использование INNER JOIN"
      correct = false
  
  [[quiz.questions]]
    question = "Что возвращает подзапрос EXISTS?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Он возвращает true, если подзапрос находит хотя бы одну совпадающую строку"
      correct = true
    [[quiz.questions.answers]]
      text = "Он возвращает совпадающие строки"
      correct = false
    [[quiz.questions.answers]]
      text = "Он возвращает количество совпадающих строк"
      correct = false
+++

Эта задача не так проста, особенно с GORM.

<!--more-->

Посмотрите на следующие сущности:
```go
type Parent struct {
gorm.Model
Name string
}

type Child struct {
gorm.Model
ParentID uint // связь child -> parent многие к одному
Name     string
}
```

В коде используется `gorm.Model` для базовой структуры.

Как отфильтровать родителей, фильтруя связанных детей?

Общее решение в SQL-базах данных — использовать простой JOIN и затем фильтровать. Это возможно с GORM:

```go
err := db.Model(&Parent{}).Joins(
"LEFT JOIN children ON parents.id = children.parent_id AND children.name = ?",
"somename",
).Find(&results).Error
```

Полный запрос:

```sql
SELECT * FROM "parents" LEFT JOIN children ON parents.id = children.parent_id AND children.name = 'somename'
```

Но это обычно не работает для нашей задачи — для родителей, у которых несколько совпадающих детей. В этом случае у нас будут такие родители повторяться несколько раз.

Таким образом, простой JOIN здесь, вероятно, не работает. Нам, вероятно, нужно решение на основе условия WHERE. Следующий код работает отлично:

```go
err := db.Model(&Parent{}).Where(
"EXISTS (SELECT 1 FROM children WHERE children.parent_id = parents.id AND children.name = ?)",
"somename",
).Find(&results).Error
```

Полный запрос выглядит следующим образом:

```sql
SELECT * FROM "parents" WHERE EXISTS (SELECT 1 FROM children WHERE children.parent_id = parents.id AND children.name = 'somename')
```
