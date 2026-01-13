+++
title = 'GO Templates: Principles and Usage'
date = 2020-04-09T09:00:00+03:00
draft = false
tags = ['templates','html','text','sources']
featured_image = 'lis.svg'
url = '/en/post/templates.html'

[quiz]
  [[quiz.questions]]
    question = "What symbols are used to delimit template instructions in Go?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "{{ and }}"
      correct = true
    [[quiz.questions.answers]]
      text = "{% and %}"
      correct = false
    [[quiz.questions.answers]]
      text = "<% and %>"
      correct = false
  
  [[quiz.questions]]
    question = "What does the dot (.) represent in Go templates?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "The current template context (default data)"
      correct = true
    [[quiz.questions.answers]]
      text = "A wildcard character"
      correct = false
    [[quiz.questions.answers]]
      text = "A method call"
      correct = false
  
  [[quiz.questions]]
    question = "What types can be iterated using the range instruction?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "Arrays"
      correct = true
    [[quiz.questions.answers]]
      text = "Slices"
      correct = true
    [[quiz.questions.answers]]
      text = "Maps"
      correct = true
    [[quiz.questions.answers]]
      text = "Channels"
      correct = true
  
  [[quiz.questions]]
    question = "What package does html/template use internally?"
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
    question = "What package allows Go templates to work with any data type?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "reflect package"
      correct = true
    [[quiz.questions.answers]]
      text = "types package"
      correct = false
    [[quiz.questions.answers]]
      text = "interface package"
      correct = false
  
  [[quiz.questions]]
    question = "When is a condition considered false in Go template if instruction?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "When value equals 0"
      correct = true
    [[quiz.questions.answers]]
      text = "When value equals empty string \"\""
      correct = true
    [[quiz.questions.answers]]
      text = "When value equals nil"
      correct = true
    [[quiz.questions.answers]]
      text = "When value equals 1"
      correct = false
  
  [[quiz.questions]]
    question = "What does the with instruction do in Go templates?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Sets current template context to pipeline if it's equivalent to true"
      correct = true
    [[quiz.questions.answers]]
      text = "Creates a new template"
      correct = false
    [[quiz.questions.answers]]
      text = "Executes a loop"
      correct = false
  
  [[quiz.questions]]
    question = "What instructions are used to create templates?"
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
    question = "How is a created template called?"
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
    question = "What standard functions are available in Go templates?"
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
    question = "How to pass a value to a function using pipe operator (|)?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Value becomes the last function argument"
      correct = true
    [[quiz.questions.answers]]
      text = "Value becomes the first function argument"
      correct = false
    [[quiz.questions.answers]]
      text = "Pipe operator is not supported"
      correct = false
  
  [[quiz.questions]]
    question = "What are the requirements for methods called in templates?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Must return one value or two values where the last is an error"
      correct = true
    [[quiz.questions.answers]]
      text = "Must return only one value"
      correct = false
    [[quiz.questions.answers]]
      text = "Can return any number of values"
      correct = false
  
  [[quiz.questions]]
    question = "How to add custom functions to a template?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Use Funcs method with template.FuncMap"
      correct = true
    [[quiz.questions.answers]]
      text = "Use AddFunc method"
      correct = false
    [[quiz.questions.answers]]
      text = "Functions are added automatically"
      correct = false
  
  [[quiz.questions]]
    question = "What is the main difference between html/template and text/template?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "html/template automatically escapes HTML content for security"
      correct = true
    [[quiz.questions.answers]]
      text = "html/template is faster than text/template"
      correct = false
    [[quiz.questions.answers]]
      text = "html/template doesn't use text/template"
      correct = false
  
  [[quiz.questions]]
    question = "What struct fields can be used in templates?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Only exported fields (starting with capital letter)"
      correct = true
    [[quiz.questions.answers]]
      text = "All struct fields"
      correct = false
    [[quiz.questions.answers]]
      text = "Only public methods"
      correct = false
  
  [[quiz.questions]]
    question = "What boolean operators are available in Go templates?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "eq (equals)"
      correct = true
    [[quiz.questions.answers]]
      text = "ne (not equals)"
      correct = true
    [[quiz.questions.answers]]
      text = "lt (less than)"
      correct = true
    [[quiz.questions.answers]]
      text = "and"
      correct = false
  
  [[quiz.questions]]
    question = "How to get key and value in range loop for a map?"
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

Packages `text/template` and `html/template` are part of the Go standard library. Go templates are used in many Go-programmed software — Docker, Kubernetes, Helm. Many third-party libraries are integrated with Go templates, for example Echo. Knowing Go template syntax is very useful.

This article consists of `text/template` package documentation and a couple of author's solutions. After describing Go template syntax, we'll dive into `text/template` and `html/template` sources.

<!--more-->

Go templates are active, which means flow-control instructions such as `if`, `else`, and `range` cycles are available.

Go is a strictly typed language, but templates work with all data types, thanks to the reflect package developers.

Let's take a brief look at Go template syntax basics.

## Template Calls

All template instructions are set between `{{` and `}}` symbols. Any other text is plain text that is simply printed to output without any changes.

## Template Context

We have `Execute` and `ExecuteTemplate` Go functions to run templates. Both of them have a `data interface{}` parameter.

```go
Execute(wr io.Writer, data interface{}) error
ExecuteTemplate(wr io.Writer, name string, data interface{}) error
```

The `data` parameter here is the default template data. It can be accessed from templates as `.`. The following code prints the default data:

```go
{{ . }}
```

Let's call the default data the current template context. Some template instructions can change the template context.

## Comments

```go
{{/* comment */}}
```

## Condition Operator

```go
{{if condition}} T1 {{end}}
```

If condition is `0`, `""`, `nil`, or an empty array/slice, the condition is processed as false, and T1 won't execute. Otherwise, T1 will execute.

Variations with `else`, `else if`:

```go
{{if condition}} T1 {{else}} T0 {{end}}
{{if condition1}} T1 {{else if condition2}} T0 {{end}}
```

## Range Cycles

One can iterate arrays, slices, maps, or channels.

In the following code, the T1 instruction will be executed on each iteration:

```go
{{range pipeline}} T1 {{end}}
{{range pipeline}} T1 {{else}} T2 {{end}}
```

Key/value variables for each iteration can also be obtained:

```go
{{range $key, $value := pipeline}}
{{ $key }}: {{ $value }}
{{end}}
```

## With

```go
{{with pipeline}} T1 {{end}}
```

If pipeline is equivalent to true (as described in the `if` explanation), T1 is executed, and the current template context is set to pipeline.

## Templates

One can create a template using one of the following instructions:

```go
{{block "name" pipeline}} T1 {{end}}
{{define "name" pipeline}} T1 {{end}}
```

To run a template:

```go
{{template "name" pipeline}}
```

## Map Elements, Struct Fields, Method Calls

Go templates can print data. For example, one can print a struct field or map value. Struct fields to be used in templates should be exported (started with a capital letter). Map keys can start with a lowercase letter.

All of them can be chained:

```go
.Field1.Field2.key1
.key1.key2
```

One can use method calls in templates. Template methods should return one value or two values, and the last value should be an error. There is such a method check in the sources.

Go code:

```go
type myType struct{}

func(m *myType) Method() string {
    return "123"
}
```

Template code:

```go
.Method()
```

## Standard Functions

There are two types of functions in Go templates — built-in and user-defined.

Function call syntax for every function is as follows:

```go
funcname arg1 arg2 arg3
```

Standard functions list:

- `call funcLocation arg1 arg2` — function to call a function with arguments;
- `index x 1 2 3` — obtaining a slice/array/map element;
- `slice x 1 2` — slicing slice/array — `s[1:2]`;
- `len x` — obtaining length of slice/array/map;
- `print`, `printf`, `println` — explicit printing of data;

Boolean operators also work as functions:

- `eq arg1 arg2` — `arg1 == arg2`
- `ne arg1 arg2` — `arg1 != arg2`
- `lt arg1 arg2` — `arg1 < arg2`
- `le arg1 arg2` — `arg1 <= arg2`
- `gt arg1 arg2` — `arg1 > arg2`
- `ge arg1 arg2` — `arg1 >= arg2`

Values can be chained to a function with the `|` operator. Such values become the last function argument:

```go
{{"output" | printf "%q"}}
```

## User-Defined Functions

One can define any function in Go to use it later in templates.

Go code of a `last` function which helps to check if the iterated element is the last in a list:

```go
tempTemplate := template.New("main").Funcs(
    template.FuncMap{
        "last": func(x int, a interface{}) bool {
            return x == reflect.ValueOf(a).Len()-1
        },
    },
)
```

Using `last` in a template:

```go
{{ $allKeywords := .Data.Keywords }}
{{ range $k,$v := .Data.Keywords}}
{{ $v }}{{ if ne (last $i $allKeywords) }},{{ end }}
{{ end }}
```

## Internals

Go templates use the `reflect` package to work with any data type. For example, `range` sources in `text/template`:

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

There is a logical branch for every iterated type. The same reflect approach is used in the `evalField` instruction sources.

`html/template` uses `text/template`. `html/template` is specifically designed to identify exactly which type of HTML content is processed by the template (HTML tag's name, attribute, tag's content, CSS content, URL). Based on this content identification, there are different escape solutions.