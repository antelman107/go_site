+++
title = 'Создание простого TCP чата на Go'
date = 2020-03-30T09:00:00+03:00
draft = false
tags = ['tcp', 'server', 'chat']
url = '/ru/post/golang-tcp-server-chat.html'

[quiz]
  [[quiz.questions]]
    question = "Почему нам нужны горутины для обработки нескольких клиентских подключений в TCP чат-сервере?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Чтение из подключений — это блокирующая операция"
      correct = true
    [[quiz.questions.answers]]
      text = "Горутины делают код быстрее"
      correct = false
    [[quiz.questions.answers]]
      text = "TCP требует горутины"
      correct = false
  
  [[quiz.questions]]
    question = "Какая структура данных рекомендуется для хранения клиентских подключений в конкурентном TCP чате?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "sync.Map"
      correct = true
    [[quiz.questions.answers]]
      text = "Обычная карта с мьютексом"
      correct = false
    [[quiz.questions.answers]]
      text = "Срез"
      correct = false
  
  [[quiz.questions]]
    question = "Какой паттерн используется при отправке сообщения всем клиентам в чате?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Паттерн веерной рассылки (fan out)"
      correct = true
    [[quiz.questions.answers]]
      text = "Паттерн round-robin"
      correct = false
    [[quiz.questions.answers]]
      text = "Паттерн broadcast"
      correct = false
  
  [[quiz.questions]]
    question = "Почему важно работать с указателем на sync.Map, а не со значением?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "sync.Map содержит блокировки, которые не должны копироваться"
      correct = true
    [[quiz.questions.answers]]
      text = "Указатели быстрее"
      correct = false
    [[quiz.questions.answers]]
      text = "sync.Map работает только с указателями"
      correct = false
  
  [[quiz.questions]]
    question = "Какая функция используется для создания TCP сервера в Go?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "net.Listen"
      correct = true
    [[quiz.questions.answers]]
      text = "net.Server"
      correct = false
    [[quiz.questions.answers]]
      text = "net.TCPListen"
      correct = false
  
  [[quiz.questions]]
    question = "Какой метод используется для принятия новых подключений?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Accept"
      correct = true
    [[quiz.questions.answers]]
      text = "Connect"
      correct = false
    [[quiz.questions.answers]]
      text = "Receive"
      correct = false
  
  [[quiz.questions]]
    question = "Какой пакет используется для чтения строк из подключения?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "bufio"
      correct = true
    [[quiz.questions.answers]]
      text = "io"
      correct = false
    [[quiz.questions.answers]]
      text = "strings"
      correct = false
  
  [[quiz.questions]]
    question = "Почему используется UUID для идентификации клиентов?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Гарантирует уникальность ключей с крайне низкой вероятностью коллизий"
      correct = true
    [[quiz.questions.answers]]
      text = "UUID быстрее других идентификаторов"
      correct = false
    [[quiz.questions.answers]]
      text = "UUID обязателен для TCP"
      correct = false
  
  [[quiz.questions]]
    question = "Какой метод sync.Map используется для перебора всех элементов?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Range"
      correct = true
    [[quiz.questions.answers]]
      text = "Iterate"
      correct = false
    [[quiz.questions.answers]]
      text = "ForEach"
      correct = false
  
  [[quiz.questions]]
    question = "Что происходит при возврате false из функции в Range?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Перебор карты останавливается"
      correct = true
    [[quiz.questions.answers]]
      text = "Возникает ошибка"
      correct = false
    [[quiz.questions.answers]]
      text = "Ничего не происходит"
      correct = false
  
  [[quiz.questions]]
    question = "Как обрабатывается отключение клиента в чате?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "При ошибке чтения функция возвращается"
      correct = true
    [[quiz.questions.answers]]
      text = "Клиент удаляется из connMap через Delete"
      correct = true
    [[quiz.questions.answers]]
      text = "Подключение закрывается через defer"
      correct = true
    [[quiz.questions.answers]]
      text = "Клиент остаётся в карте"
      correct = false
  
  [[quiz.questions]]
    question = "Почему используется defer для закрытия подключения?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "defer вызывается при любом завершении функции, гарантируя закрытие"
      correct = true
    [[quiz.questions.answers]]
      text = "defer быстрее обычного закрытия"
      correct = false
    [[quiz.questions.answers]]
      text = "defer обязателен для TCP"
      correct = false
  
  [[quiz.questions]]
    question = "Какой паттерн используется для отправки сообщения всем подключённым клиентам?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Перебор всех подключений через Range и запись в каждое"
      correct = true
    [[quiz.questions.answers]]
      text = "Отправка только первому клиенту"
      correct = false
    [[quiz.questions.answers]]
      text = "Использование канала для всех клиентов"
      correct = false
  
  [[quiz.questions]]
    question = "Что происходит при ошибке записи в одно из подключений при рассылке?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Ошибка логируется, но рассылка продолжается другим клиентам"
      correct = true
    [[quiz.questions.answers]]
      text = "Рассылка останавливается"
      correct = false
    [[quiz.questions.answers]]
      text = "Программа завершается"
      correct = false
  
  [[quiz.questions]]
    question = "Почему чтение из подключения выполняется в бесконечном цикле?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Чтобы обрабатывать множественные сообщения от клиента"
      correct = true
    [[quiz.questions.answers]]
      text = "Чтобы увеличить производительность"
      correct = false
    [[quiz.questions.answers]]
      text = "Это требование TCP"
      correct = false
+++

Пример простого TCP чата на Go с объяснением логики.

<!--more-->

Начнём с запуска сервера:

```go
l, err := net.Listen("tcp", "localhost:9090")
if err != nil {
	return
}

defer l.Close()
```

Сервер слушает TCP порт 9090 на localhost. Это обычная практика — запускать сервисы, привязанные к localhost.

Теперь давайте принимаем подключения:

```go
conn, err := l.Accept()
if err != nil {
	return
}
```

Пока без обработки ошибок. На этом остановимся позже.

В чате нам нужно читать сообщения, отправляемые клиентами.

Количество клиентов обычно больше одного, а чтение — это блокирующая операция. Один поток программы не позволяет нам читать из всех подключений. Исправим это, добавив горутину для каждого подключённого клиента:

```go
l, err := net.Listen("tcp", "localhost:9090")
if err != nil {
	return
}

defer l.Close()

for {
	conn, err := l.Accept()
	if err != nil {
		return
	}

	go handleUserConnection(conn)
}

func handleUserConnection(c net.Conn) {
	defer c.Close()

	for {
		userInput, err := bufio.NewReader(c).ReadString('\n')
		if err != nil {
			return
		}
	}
}
```

Здесь мы возвращаемся из функции обработки при ошибке чтения, что произойдёт при отключении клиента. После возврата из функции мы также закрываем подключение. Здесь помогает defer — независимо от того, как завершается функция — defer вызывается после return.

Нам не нужно обрабатывать ошибки закрытия подключения, потому что ошибки здесь не меняют поток выполнения программы.

Теперь у нас есть работающий сервер, но нет логики, кроме чтения данных от клиентов. Добавим логику чата. Самый простой чат будет отправлять любое сообщение всем клиентам. Это можно назвать паттерном "fan out" (веерная рассылка), он проиллюстрирован ниже:

Чтобы реализовать "fan out", нам нужно перебрать все подключения клиентов и записать в них сообщения.

Для этого нам нужно каким-то образом хранить подключения, чтобы мы могли их перебирать. Я выбрал `sync.Map` здесь, потому что он решает все проблемы конкурентного доступа.

Для нашей задачи хватило бы среза. Но в нашей конкурентной программе нам пришлось бы добавлять/удалять данные из этого среза, и это было бы невозможно без `sync.Lock`. Поскольку мы пишем простой TCP чат, давайте просто используем предопределённый тип, который решает проблемы конкурентности.

```go
// Используем sync.Map, чтобы не иметь дело с проблемами конкурентности срезов/карт
var connMap = &sync.Map{}
```

Важно работать с указателем на `sync.Map`, а не со значением.

`sync.Map` — это структура, которая содержит `sync.Lock`. Есть известная проблема под названием "блокировки, передаваемые по значению".

Для нашей карты нам нужны ключи. Я использую UUID здесь, потому что он гарантирует, что ключи будут разными. (Крайне низкая вероятность того, что UUID сгенерирует два одинаковых значения).

Когда клиент отключается, нам нужно удалить этого клиента из карты. Давайте передадим ID клиента в функцию обработки:

```go
id := uuid.New().String()
connMap.Store(id, conn)
```

Наконец, давайте реализуем паттерн веерной рассылки сообщений:

```go
for {
	userInput, err := bufio.NewReader(c).ReadString('\n')
	if err != nil {
		return
	}

	connMap.Range(func(key, value interface{}) bool {
		if conn, ok := value.(net.Conn); ok {
			conn.Write([]byte(userInput))
		}

		return true
	})
}
```

Мы можем остановить перебор карты, если вернём false из функции range.

Наконец, я добавил обработку ошибок и использование логгера zap. Следующий код — это полное решение. Полный исходный код также доступен на GitHub.

```go
package main

import (
	"bufio"
	"net"
	"sync"

	"github.com/google/uuid"
	"go.uber.org/zap"
)

func main() {
	var loggerConfig = zap.NewProductionConfig()
	loggerConfig.Level.SetLevel(zap.DebugLevel)

	logger, err := loggerConfig.Build()
	if err != nil {
		panic(err)
	}

	l, err := net.Listen("tcp", "localhost:9090")
	if err != nil {
		return
	}

	defer l.Close()

	// Используем sync.Map, чтобы не иметь дело с проблемами конкурентности срезов/карт
	var connMap = &sync.Map{}

	for {
		conn, err := l.Accept()
		if err != nil {
			logger.Error("error accepting connection", zap.Error(err))
			return
		}

		id := uuid.New().String()
		connMap.Store(id, conn)

		go handleUserConnection(id, conn, connMap, logger)
	}
}

func handleUserConnection(id string, c net.Conn, connMap *sync.Map, logger *zap.Logger) {
	defer func() {
		c.Close()
		connMap.Delete(id)
	}()

	for {
		userInput, err := bufio.NewReader(c).ReadString('\n')
		if err != nil {
			logger.Error("error reading from client", zap.Error(err))
			return
		}

		connMap.Range(func(key, value interface{}) bool {
			if conn, ok := value.(net.Conn); ok {
				if _, err := conn.Write([]byte(userInput)); err != nil {
					logger.Error("error on writing to connection", zap.Error(err))
				}
			}

			return true
		})
	}
}
```
