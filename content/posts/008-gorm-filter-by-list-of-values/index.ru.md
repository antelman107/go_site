+++
title = 'Фильтрация по списку значений с помощью GORM'
date = 2020-04-07T09:00:00+03:00
draft = false
tags = ['gorm', 'sql','postgresql']
url = '/ru/post/gorm-filter-by-list-of-values.html'

[quiz]
  [[quiz.questions]]
    question = "Какой оператор PostgreSQL следует использовать для фильтрации по списку значений?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Оператор ANY"
      correct = true
    [[quiz.questions.answers]]
      text = "Оператор IN"
      correct = false
    [[quiz.questions.answers]]
      text = "Оператор EXISTS"
      correct = false
  
  [[quiz.questions]]
    question = "Какая функция преобразует Go-срез в формат массива PostgreSQL?"
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
    question = "Почему подход с оператором ANY и pq.Array() эффективен?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Он хорошо работает с построителем запросов GORM и эффективен"
      correct = true
    [[quiz.questions.answers]]
      text = "Он быстрее SQL"
      correct = false
    [[quiz.questions.answers]]
      text = "Он не требует подключения к базе данных"
      correct = false
+++

Когда необходимо отфильтровать данные по списку значений (например, по ID: 1, 2, 3), следует использовать оператор `ANY` в сочетании с `pq.Array` из драйвера PostgreSQL.

<!--more-->

## Решение

Вот как можно отфильтровать записи по списку ID:

```go
var priceIDs = []uint{1, 2, 3}
if err := db.Model(prices).Where("id = ANY(?)", pq.Array(priceIDs)).Find(&prices).Error; err != nil {
    return err
}
```

Функция `pq.Array()` преобразует Go-срез в формат массива PostgreSQL, который можно использовать с оператором `ANY`. Этот подход эффективен и хорошо работает с построителем запросов GORM.

