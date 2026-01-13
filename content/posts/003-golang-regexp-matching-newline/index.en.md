+++
title = 'Golang regexp: matching newline'
date = 2020-03-30T09:00:00+03:00
draft = false
tags = ['regular expressions', 'sources']
featured_image = 'regular.svg'
url = '/en/post/golang-regexp-matching-newline.html'

[quiz]
  [[quiz.questions]]
    question = "In Go, does the dot (.) character in regular expressions match newline characters by default?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "No, unlike PHP and JavaScript"
      correct = true
    [[quiz.questions.answers]]
      text = "Yes, just like PHP and JavaScript"
      correct = false
    [[quiz.questions.answers]]
      text = "Only in POSIX mode"
      correct = false
  
  [[quiz.questions]]
    question = "What flag is needed to allow the dot (.) to match newline characters in Go?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "DotNL flag"
      correct = true
    [[quiz.questions.answers]]
      text = "ClassNL flag"
      correct = false
    [[quiz.questions.answers]]
      text = "OneLine flag"
      correct = false
  
  [[quiz.questions]]
    question = "How many regexp flag options are available in Go's `regexp.Compile` function?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Two: POSIX and Perl"
      correct = true
    [[quiz.questions.answers]]
      text = "One: POSIX only"
      correct = false
    [[quiz.questions.answers]]
      text = "Three: POSIX, Perl, and DotNL"
      correct = false
  
  [[quiz.questions]]
    question = "What is the recommended regexp pattern to match all characters including newlines in Go?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Using character classes like [[:graph:]\\s]"
      correct = true
    [[quiz.questions.answers]]
      text = "Using the dot (.) character"
      correct = false
    [[quiz.questions.answers]]
      text = "Using \\n explicitly"
      correct = false
+++

Why regular expressions with dot (".") work differently in Go compared to PHP and JavaScript.

<!--more-->

To enable code syntax highlighting on this website, I use regular expressions. The logic is simple — I put source code into special HTML tags. When a post loads, I process these tags — search for them using regular expressions, and replace the source code with highlighted versions.

I spent a lot of time trying to understand why some code examples were not matched by regular expressions. I used the dot "." special character to match any symbol inside my tag.
Look at the following regexp and text example and guess if it matches or not:

**Regexp:**
```regexp
<tag>(.*?)</tag>
```

**Text:**
```html
<tag>1
2
3</tag>
```

If you have PHP experience, your answer will probably be "yes".

However, a simple example from the Go Playground makes it clear that the answer is actually "no":
```go
match, _ := regexp.MatchString("<tag>(.*)</tag>", "<tag>1\n2\n3</tag>")
fmt.Println(match)

// false
```

Then I searched Go sources trying to understand how Go deals with character classes, and I found the following list of flags:

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

According to the sources, Go works in the following way:

- It compiles regexp using `syntax.Parse`
- `syntax.Parse` uses Flags to "plan" regular expression execution (to match regexp symbols to operations)
- `regexp.Regexp` (a public struct) is created using the results of `syntax.Parse`

So we need to compile the regexp with the `DotNL` flag.

When I searched all `regexp.Compile` function use cases, I found that there were only two regexp flag options available — POSIX and Perl. That means there is no option in Go to match newlines with dot.

So, the regexp that actually works is below:
```regexp
<tag>([[:graph:]\\s]*?)</tag>
```

There are also a lot of predefined character classes, documented [here](https://golang.org/pkg/regexp/syntax/). I used two of them to cover all characters in the `[]` brackets.