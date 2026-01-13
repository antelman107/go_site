+++
title = 'Шаблоны GO: принципы и использование'
date = 2020-04-09T09:00:00+03:00
draft = false
tags = ['templates','html','text','sources']
featured_image = 'lis.svg'
url = '/ru/post/templates.html'

[quiz]
  [[quiz.questions]]
    question = "Какие символы используются для ограничения инструкций шаблона в Go?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "{{ и }}"
      correct = true
    [[quiz.questions.answers]]
      text = "{% и %}"
      correct = false
    [[quiz.questions.answers]]
      text = "<% и %>"
      correct = false
  
  [[quiz.questions]]
    question = "Что представляет точка (.) в шаблонах Go?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Текущий контекст шаблона (данные по умолчанию)"
      correct = true
    [[quiz.questions.answers]]
      text = "Символ подстановки"
      correct = false
    [[quiz.questions.answers]]
      text = "Вызов метода"
      correct = false
  
  [[quiz.questions]]
    question = "Какие типы можно итерировать с помощью инструкции range?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "Массивы"
      correct = true
    [[quiz.questions.answers]]
      text = "Срезы"
      correct = true
    [[quiz.questions.answers]]
      text = "Карты"
      correct = true
    [[quiz.questions.answers]]
      text = "Каналы"
      correct = true
  
  [[quiz.questions]]
    question = "Какой пакет использует html/template внутри?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "text/template"
      correct = true
    [[quiz.questions.answers]]
      text = "html/parser"
      correct = false
    [[quiz.questions.answers]]
      text = "net/html"
      correct = false
  
  [[quiz.questions]]
    question = "Какой пакет позволяет шаблонам Go работать с любыми типами данных?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Пакет reflect"
      correct = true
    [[quiz.questions.answers]]
      text = "Пакет types"
      correct = false
    [[quiz.questions.answers]]
      text = "Пакет interface"
      correct = false
  
  [[quiz.questions]]
    question = "Когда условие в инструкции if считается false в шаблонах Go?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "Когда значение равно 0"
      correct = true
    [[quiz.questions.answers]]
      text = "Когда значение равно пустой строке \"\""
      correct = true
    [[quiz.questions.answers]]
      text = "Когда значение равно nil"
      correct = true
    [[quiz.questions.answers]]
      text = "Когда значение равно 1"
      correct = false
  
  [[quiz.questions]]
    question = "Что делает инструкция with в шаблонах Go?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Устанавливает текущий контекст шаблона в pipeline, если он эквивалентен true"
      correct = true
    [[quiz.questions.answers]]
      text = "Создаёт новый шаблон"
      correct = false
    [[quiz.questions.answers]]
      text = "Выполняет цикл"
      correct = false
  
  [[quiz.questions]]
    question = "Какие инструкции используются для создания шаблонов?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "define"
      correct = true
    [[quiz.questions.answers]]
      text = "block"
      correct = true
    [[quiz.questions.answers]]
      text = "template"
      correct = false
  
  [[quiz.questions]]
    question = "Как вызывается созданный шаблон?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "{{template \"name\" pipeline}}"
      correct = true
    [[quiz.questions.answers]]
      text = "{{call \"name\" pipeline}}"
      correct = false
    [[quiz.questions.answers]]
      text = "{{execute \"name\" pipeline}}"
      correct = false
  
  [[quiz.questions]]
    question = "Какие стандартные функции доступны в шаблонах Go?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "len"
      correct = true
    [[quiz.questions.answers]]
      text = "index"
      correct = true
    [[quiz.questions.answers]]
      text = "printf"
      correct = true
    [[quiz.questions.answers]]
      text = "map"
      correct = false
  
  [[quiz.questions]]
    question = "Как передать значение в функцию через pipe оператор (|)?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Значение становится последним аргументом функции"
      correct = true
    [[quiz.questions.answers]]
      text = "Значение становится первым аргументом функции"
      correct = false
    [[quiz.questions.answers]]
      text = "Pipe оператор не поддерживается"
      correct = false
  
  [[quiz.questions]]
    question = "Какие требования к методам, вызываемым в шаблонах?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Должны возвращать одно значение или два значения, где последнее - ошибка"
      correct = true
    [[quiz.questions.answers]]
      text = "Должны возвращать только одно значение"
      correct = false
    [[quiz.questions.answers]]
      text = "Могут возвращать любое количество значений"
      correct = false
  
  [[quiz.questions]]
    question = "Как добавить пользовательские функции в шаблон?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Использовать метод Funcs с template.FuncMap"
      correct = true
    [[quiz.questions.answers]]
      text = "Использовать метод AddFunc"
      correct = false
    [[quiz.questions.answers]]
      text = "Функции добавляются автоматически"
      correct = false
  
  [[quiz.questions]]
    question = "В чём основное отличие html/template от text/template?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "html/template автоматически экранирует HTML контент для безопасности"
      correct = true
    [[quiz.questions.answers]]
      text = "html/template быстрее text/template"
      correct = false
    [[quiz.questions.answers]]
      text = "html/template не использует text/template"
      correct = false
  
  [[quiz.questions]]
    question = "Какие поля структуры можно использовать в шаблонах?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Только экспортированные поля (начинающиеся с заглавной буквы)"
      correct = true
    [[quiz.questions.answers]]
      text = "Все поля структуры"
      correct = false
    [[quiz.questions.answers]]
      text = "Только публичные методы"
      correct = false
  
  [[quiz.questions]]
    question = "Какие булевы операторы доступны в шаблонах Go?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "eq (равно)"
      correct = true
    [[quiz.questions.answers]]
      text = "ne (не равно)"
      correct = true
    [[quiz.questions.answers]]
      text = "lt (меньше)"
      correct = true
    [[quiz.questions.answers]]
      text = "and"
      correct = false
  
  [[quiz.questions]]
    question = "Как получить ключ и значение в цикле range для карты?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "{{range $key, $value := pipeline}}"
      correct = true
    [[quiz.questions.answers]]
      text = "{{range key, value := pipeline}}"
      correct = false
    [[quiz.questions.answers]]
      text = "{{range pipeline as $key, $value}}"
      correct = false
+++

Пакеты `text/template` и `html/template` являются частью стандартной библиотеки Go. Шаблоны Go используются во многих программах, написанных на Go — Docker, Kubernetes, Helm. Многие сторонние библиотеки интегрированы с шаблонами Go, например Echo. Знание синтаксиса шаблонов Go очень полезно.

Эта статья состоит из документации пакета `text/template` и нескольких решений автора. После описания синтаксиса шаблонов Go мы погрузимся в исходники `text/template` и `html/template`.

<!--more-->

Шаблоны Go являются активными, что означает, что здесь доступны инструкции управления потоком, такие как `if`, `else` и циклы `range`.

Go — строго типизированный язык, но шаблоны работают со всеми типами данных благодаря разработчикам пакета `reflect`.

Давайте кратко рассмотрим основы синтаксиса шаблонов Go.

## Вызовы шаблонов

Все инструкции шаблона задаются между символами `{{` и `}}`. Любой другой текст — это обычный текст, который просто выводится без изменений.

## Контекст шаблона

У нас есть функции Go `Execute` и `ExecuteTemplate` для запуска шаблонов. Обе они имеют параметр `data interface{}`.

```go
Execute(wr io.Writer, data interface{}) error
ExecuteTemplate(wr io.Writer, name string, data interface{}) error
```

Параметр `data` здесь — это данные шаблона по умолчанию. К ним можно получить доступ из шаблонов как `.`. Следующий код выводит данные по умолчанию:

```go
{{ . }}
```

Назовём данные по умолчанию текущим контекстом шаблона. Некоторые инструкции шаблона могут изменять контекст шаблона.

## Комментарии

```go
{{/* comment */}}
```

## Условный оператор

```go
{{if condition}} T1 {{end}}
```

Если условие равно `0`, `""`, `nil` или пустому массиву/срезу, условие обрабатывается как false, и T1 не выполнится. В противном случае T1 выполнится.

Вариации с `else`, `else if`:

```go
{{if condition}} T1 {{else}} T0 {{end}}
{{if condition1}} T1 {{else if condition2}} T0 {{end}}
```

## Циклы Range

Можно итерировать массивы, срезы, карты или каналы.

В следующем коде инструкция T1 будет выполняться на каждой итерации:

```go
{{range pipeline}} T1 {{end}}
{{range pipeline}} T1 {{else}} T2 {{end}}
```

Переменные ключ/значение для каждой итерации также можно получить:

```go
{{range $key, $value := pipeline}}
{{ $key }}: {{ $value }}
{{end}}
```

## With

```go
{{with pipeline}} T1 {{end}}
```

Если pipeline эквивалентен true (как описано в объяснении `if`), T1 выполняется, и текущий контекст шаблона устанавливается в pipeline.

## Шаблоны

Можно создать шаблон, используя одну из следующих инструкций:

```go
{{block "name" pipeline}} T1 {{end}}
{{define "name" pipeline}} T1 {{end}}
```

Для запуска шаблона:

```go
{{template "name" pipeline}}
```

## Элементы карты, поля структуры, вызовы методов

Шаблоны Go могут выводить данные. Например, можно вывести поле структуры или значение карты. Поля структуры, используемые в шаблонах, должны быть экспортированными (начинаться с заглавной буквы). Ключи карты могут начинаться со строчной буквы.

Все они могут быть объединены в цепочку:

```go
.Field1.Field2.key1
.key1.key2
```

Можно использовать вызовы методов в шаблонах. Методы шаблона должны возвращать одно значение или два значения, и последнее значение должно быть ошибкой. В исходниках есть такая проверка метода.

Код Go:

```go
type myType struct{}

func(m *myType) Method() string {
    return "123"
}
```

Код шаблона:

```go
.Method()
```

## Стандартные функции

В шаблонах Go есть два типа функций — встроенные и пользовательские.

Синтаксис вызова функции для каждой функции следующий:

```go
funcname arg1 arg2 arg3
```

Список стандартных функций:

- `call funcLocation arg1 arg2` — функция для вызова функции с аргументами;
- `index x 1 2 3` — получение элемента среза/массива/карты;
- `slice x 1 2` — срез среза/массива — `s[1:2]`;
- `len x` — получение длины среза/массива/карты;
- `print`, `printf`, `println` — явный вывод данных;

Булевы операторы также работают как функции:

- `eq arg1 arg2` — `arg1 == arg2`
- `ne arg1 arg2` — `arg1 != arg2`
- `lt arg1 arg2` — `arg1 < arg2`
- `le arg1 arg2` — `arg1 <= arg2`
- `gt arg1 arg2` — `arg1 > arg2`
- `ge arg1 arg2` — `arg1 >= arg2`

Значения можно объединять в цепочку с функцией с помощью оператора `|`. Такие значения становятся последним аргументом функции:

```go
{{"output" | printf "%q"}}
```

## Пользовательские функции

Можно определить любую функцию в Go для использования её позже в шаблонах.

Код Go функции `last`, которая помогает проверить, является ли итерируемый элемент последним в списке:

```go
tempTemplate := template.New("main").Funcs(
    template.FuncMap{
        "last": func(x int, a interface{}) bool {
            return x == reflect.ValueOf(a).Len()-1
        },
    },
)
```

Использование `last` в шаблоне:

```go
{{ $allKeywords := .Data.Keywords }}
{{ range $k,$v := .Data.Keywords}}
{{ $v }}{{ if ne (last $i $allKeywords) }},{{ end }}
{{ end }}
```

## Внутреннее устройство

Шаблоны Go используют пакет `reflect` для работы с любыми типами данных. Например, исходники `range` в `text/template`:

```go
func (s *state) walkRange(dot reflect.Value, r *parse.RangeNode) {
    // ...
    switch val.Kind() {
    case reflect.Array, reflect.Slice:
        // ...
    case reflect.Map:
        // ...
    case reflect.Chan:
        // ...
    }
}
```

Для каждого итерируемого типа есть логическая ветвь. Тот же подход с `reflect` используется в исходниках инструкции `evalField`.

`html/template` использует `text/template`. `html/template` специально разработан для точного определения того, какой тип HTML-контента обрабатывается шаблоном (имя HTML-тега, атрибут, содержимое тега, CSS-контент, URL). На основе этой идентификации контента существуют различные решения экранирования.

