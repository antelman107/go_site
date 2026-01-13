+++
title = 'Golang regexp: сопоставление символа новой строки'
date = 2020-03-30T09:00:00+03:00
draft = false
tags = ['regular expressions', 'sources']
featured_image = 'regular.svg'
url = '/ru/post/golang-regexp-matching-newline.html'

[quiz]
  [[quiz.questions]]
    question = "В Go символ точки (.) в регулярных выражениях по умолчанию сопоставляется с символами новой строки?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Нет, в отличие от PHP и JavaScript"
      correct = true
    [[quiz.questions.answers]]
      text = "Да, так же как в PHP и JavaScript"
      correct = false
    [[quiz.questions.answers]]
      text = "Только в режиме POSIX"
      correct = false
  
  [[quiz.questions]]
    question = "Какой флаг нужен, чтобы позволить точке (.) сопоставляться с символами новой строки в Go?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Флаг DotNL"
      correct = true
    [[quiz.questions.answers]]
      text = "Флаг ClassNL"
      correct = false
    [[quiz.questions.answers]]
      text = "Флаг OneLine"
      correct = false
  
  [[quiz.questions]]
    question = "Сколько вариантов флагов регулярных выражений доступно в функции `regexp.Compile` в Go?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Два: POSIX и Perl"
      correct = true
    [[quiz.questions.answers]]
      text = "Один: только POSIX"
      correct = false
    [[quiz.questions.answers]]
      text = "Три: POSIX, Perl и DotNL"
      correct = false
  
  [[quiz.questions]]
    question = "Какое регулярное выражение рекомендуется для сопоставления всех символов, включая символы новой строки, в Go?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Использование классов символов, таких как [[:graph:]\\s]"
      correct = true
    [[quiz.questions.answers]]
      text = "Использование символа точки (.)"
      correct = false
    [[quiz.questions.answers]]
      text = "Явное использование \\n"
      correct = false
+++

Почему регулярные выражения с точкой (".") работают по-другому в Go по сравнению с PHP и JavaScript.

<!--more-->

Для включения подсветки синтаксиса кода на этом сайте я использую регулярные выражения. Логика проста — я помещаю исходный код в специальные HTML-теги. Когда пост загружается, я обрабатываю эти теги — ищу их с помощью регулярных выражений и заменяю исходный код на подсвеченные версии.

Я потратил много времени, пытаясь понять, почему некоторые примеры кода не совпадали с регулярными выражениями. Я использовал специальный символ точки "." для сопоставления любого символа внутри моего тега.
Посмотрите на следующий пример регулярного выражения и текста и угадайте, совпадает ли он или нет:

**Регулярное выражение:**
```regexp
<tag>(.*?)</tag>
```

**Текст:**
```html
<tag>1
2
3</tag>
```

Если у вас есть опыт работы с PHP, ваш ответ, вероятно, будет "да".

Однако простой пример из Go Playground ясно показывает, что ответ на самом деле "нет":
```go
match, _ := regexp.MatchString("<tag>(.*)</tag>", "<tag>1\n2\n3</tag>")
fmt.Println(match)

// false
```

Затем я искал в исходниках Go, пытаясь понять, как Go работает с классами символов, и нашел следующий список флагов:

```go
const (
FoldCase      Flags = 1 << iota // case-insensitive match
Literal                         // treat pattern as literal string
ClassNL                         // allow character classes like [^a-z] and [[:space:]] to match newline
DotNL                           // allow . to match newline
OneLine                         // treat ^ and $ as only matching at beginning and end of text
NonGreedy                       // make repetition operators default to non-greedy
PerlX                           // allow Perl extensions
UnicodeGroups                   // allow \p{Han}, \P{Han} for Unicode group and negation
WasDollar                       // regexp OpEndText was $, not \z
Simple                          // regexp contains no counted repetition

	MatchNL = ClassNL | DotNL

	Perl        = ClassNL | OneLine | PerlX | UnicodeGroups // as close to Perl as possible
	POSIX Flags = 0                                         // POSIX syntax
)
```

Согласно исходникам, Go работает следующим образом:

- Он компилирует регулярное выражение с помощью `syntax.Parse`
- `syntax.Parse` использует Flags для "планирования" выполнения регулярного выражения (для сопоставления символов регулярного выражения с операциями)
- `regexp.Regexp` (публичная структура) создается с использованием результатов `syntax.Parse`

Таким образом, нам нужно скомпилировать регулярное выражение с флагом `DotNL`.

Когда я искал все случаи использования функции `regexp.Compile`, я обнаружил, что доступны только два варианта флагов регулярных выражений — POSIX и Perl. Это означает, что в Go нет возможности сопоставлять символы новой строки с точкой.

Итак, регулярное выражение, которое на самом деле работает, приведено ниже:
```regexp
<tag>([[:graph:]\\s]*?)</tag>
```

Также существует множество предопределенных классов символов, документированных [здесь](https://golang.org/pkg/regexp/syntax/). Я использовал два из них, чтобы покрыть все символы в скобках `[]`.