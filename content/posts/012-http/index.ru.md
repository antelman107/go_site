+++
title = 'HTTP'
date = 2020-05-02T09:00:00+03:00
draft = false
tags = ['http', 'client', 'server', 'middleware']
url = '/ru/post/http.html'

[quiz]
  [[quiz.questions]]
    question = "Какой интерфейс должен быть реализован для создания HTTP-обработчика в Go?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "`http.Handler` с методом `ServeHTTP`"
      correct = true
    [[quiz.questions.answers]]
      text = "Интерфейс http.Server"
      correct = false
    [[quiz.questions.answers]]
      text = "Интерфейс `http.Request`"
      correct = false
  
  [[quiz.questions]]
    question = "Что поддерживает стандартный роутер Go?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Только простое сравнение URL"
      correct = true
    [[quiz.questions.answers]]
      text = "Сопоставление шаблонов с параметрами"
      correct = false
    [[quiz.questions.answers]]
      text = "Регулярные выражения"
      correct = false
  
  [[quiz.questions]]
    question = "Для чего используется middleware?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "Логирование"
      correct = true
    [[quiz.questions.answers]]
      text = "Сбор статистики"
      correct = true
    [[quiz.questions.answers]]
      text = "Проверка авторизации"
      correct = true
    [[quiz.questions.answers]]
      text = "Запросы к базе данных"
      correct = false
  
  [[quiz.questions]]
    question = "Как обычно реализуется middleware в Go?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Используя вложенные обработчики"
      correct = true
    [[quiz.questions.answers]]
      text = "Используя специальный пакет middleware"
      correct = false
    [[quiz.questions.answers]]
      text = "Используя аннотации"
      correct = false
+++

Все примеры из статьи находятся в репозитории на github.

Давайте рассмотрим инструменты HTTP в GO.

<!--more-->

Среди других стандартных пакетов GO есть пакет `net/http`, который реализует функциональность как HTTP-клиента, так и сервера.

## HTTP Сервер

Самый простой способ запустить HTTP-сервер — вызвать `http.ListenAndServe`:

```go
package main

import (
	"log"
	"net/http"
)

type Handler struct {}

func (h *Handler) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	_, err := w.Write([]byte("hello"))
	if err != nil {
		log.Println(err)
	}
}

func main() {
	log.Println(http.ListenAndServe("127.0.0.1:9090", &Handler{}))
}
```

Мы только что запустили сервер на порту 9090.

Второй параметр `http.ListenAndServe` — это реализатор интерфейса `http.Handler`. Интерфейс требует функцию `ServeHTTP(w http.ResponseWriter, req *http.Request)`. Эта функция будет обрабатывать все запросы на нашем сервере:

```bash
curl localhost:9090/someurl
hello

curl localhost:9090/secondurl
hello
```

Мы также можем настроить сервер для обработки конкретного URL, например `/hello`:

```go
package main

import (
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/hello", func(w http.ResponseWriter, req *http.Request) {
		_, err := w.Write([]byte("hello"))
		if err != nil {
			log.Println(err)
		}
	})

	log.Println(http.ListenAndServe(":9090", nil))
}
```

Как это работает? Когда мы вызываем `http.HandleFunc`, путь и функция-обработчик добавляются в структуру `net/http.DefaultServeMux`, которая хранит все из них. Когда мы вызываем конкретный URL, `net/http.DefaultServeMux` используется для поиска конкретного обработчика. Если обработчик не найден, генерируется стандартный ответ 404.

Стандартный роутер реализует только простое сравнение URL с каждым из зарегистрированных URL. Сопоставление шаблонов, такое как сопоставление выражения типа `/hello/{page}`, где параметр `page` может быть разным между несколькими вызовами, невозможно со стандартным роутером. Чтобы реализовать это, нам нужно либо реализовать сопоставление шаблонов самостоятельно, либо использовать одну из библиотек роутеров.

## Пользовательский роутер

Мы могли бы использовать любой другой роутер, например `gorilla/mux`. Он поддерживает сопоставление шаблонов.

```go
func main() {
	r := mux.NewRouter()
	r.HandleFunc("/products/{key}", ProductHandler)
	r.HandleFunc("/articles/{category}/", ArticlesCategoryHandler)
	r.HandleFunc("/articles/{category}/{id:[0-9]+}", ArticleHandler)
	http.Handle("/", r)
}
```

Затем в обработчике мы можем получить переменные из URL:

```go
func ArticlesCategoryHandler(w http.ResponseWriter, r *http.Request) {
	vars := mux.Vars(r)
	w.WriteHeader(http.StatusOK)
	fmt.Fprintf(w, "Category: %v\n", vars["category"])
}
```

## Middleware

В стандартном веб-сервисе есть не только код обработчика, который выполняется при каждом запросе. Также есть некоторый общий код, который выполняет общую работу — логирование, сбор статистики, проверку авторизации и т.д. Этот общий код называется middleware.

В стандартной библиотеке GO нет явной реализации middleware. Но любой может реализовать что-то подобное, используя вложенные обработчики. Давайте рассмотрим middleware для логирования, который выводит каждый запрошенный URL в консоль:

```go
package main

import (
	"log"
	"net/http"
)

func loggingMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		log.Printf("%s requested", r.URL.Path)
		next.ServeHTTP(w, r)
	})
}

func main() {
	handler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("hello"))
	})

	http.Handle("/", loggingMiddleware(handler))
	log.Println(http.ListenAndServe(":9090", nil))
}
```

При каждом запросе к нашему серверу мы теперь можем видеть следующие данные в консоли:

```
2020/05/01 18:09:42 / requested
2020/05/01 18:09:58 /someurl requested
```

Библиотеки, такие как `gorilla/mux`, `echo` или `gin`, могут значительно помочь с реализацией middleware. Кроме того, самые популярные задачи для middleware уже покрыты во многих библиотеках.

## HTTP Клиент

В пакете `net/http` есть структура `http.Client`, которая отвечает за HTTP-запросы клиента:

```go
client := http.Client{}

resp, err := client.Get("https://golangforall.com/en/")
if err != nil {
	log.Fatal(err)
}

buf, err := ioutil.ReadAll(resp.Body)
if err != nil {
	log.Fatal(err)
}

fmt.Println(string(buf))

// some html is printed
```

Стандартный HTTP-клиент в GO выглядит простым, но он покрывает все возможные задачи.

Некоторую конфигурацию покрывает поле `Transport` в структуре `Client`.

