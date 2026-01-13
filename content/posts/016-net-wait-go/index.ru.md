+++
title = 'Утилита и библиотека net-wait-go'
date = 2020-06-30T09:00:00+03:00
draft = false
tags = ['tcp','udp','docker']
featured_image = 'tube2.svg'
url = '/ru/post/net-wait-go.html'

[quiz]
  [[quiz.questions]]
    question = "Для чего используется net-wait-go?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Для ожидания открытия портов (TCP, UDP)"
      correct = true
    [[quiz.questions.answers]]
      text = "Для тестирования скорости сети"
      correct = false
    [[quiz.questions.answers]]
      text = "Для мониторинга сетевого трафика"
      correct = false
  
  [[quiz.questions]]
    question = "Почему net-wait-go полезен для Docker-контейнеров?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Минимальные Docker-образы не имеют bash или обычных утилит"
      correct = true
    [[quiz.questions.answers]]
      text = "Docker не поддерживает сеть"
      correct = false
    [[quiz.questions.answers]]
      text = "Он быстрее bash-команд"
      correct = false
  
  [[quiz.questions]]
    question = "Какие протоколы поддерживает net-wait-go?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "TCP"
      correct = true
    [[quiz.questions.answers]]
      text = "UDP"
      correct = true
    [[quiz.questions.answers]]
      text = "HTTP"
      correct = false
+++

Утилита и пакет Go для ожидания открытия портов (TCP, UDP).

<!--more-->

## Что это?

Утилита и пакет Go для ожидания открытия портов (TCP, UDP).

## Зачем это нужно?

В докеризированных приложениях обычно развертывается несколько контейнеров вместе с основным контейнером программы.
Нам нужно знать, запущены ли контейнеры, чтобы мы могли продолжить выполнение нашей программы или завершить её с ошибкой после некоторого дедлайна.

В интернете есть множество примеров, которые советуют использовать несколько bash-команд, таких как `nc`, `timeout`, `curl`. Но что если у нас есть Go программа с минимальным Docker образом `FROM scratch`, который не имеет bash? Мы могли бы использовать этот пакет как библиотеку в нашей программе — просто добавить пару строк кода для проверки доступности портов.

Этот пакет также можно скачать как утилиту и использовать из командной строки.

## Использование библиотеки

### Простой пример

```go
import "github.com/antelman107/net-wait-go/wait"

if !wait.New().Do([]string{"postgres:5432"}) {
    logger.Error("db is not available")
    return
}
```

### Определение всех опциональных настроек

```go
import "github.com/antelman107/net-wait-go/wait"

if !wait.New(
    wait.WithProto("tcp"),
    wait.WithWait(200*time.Millisecond),
    wait.WithBreak(50*time.Millisecond),
    wait.WithDeadline(15*time.Second),
    wait.WithDebug(true),
).Do([]string{"postgres:5432"}) {
    logger.Error("db is not available")
    return
}
```

## Использование утилиты

```
net-wait-go

-addrs string
    address:port(,address:port,address:port,...)
-deadline uint
    дедлайн в миллисекундах (по умолчанию 10000)
-debug
    переключатель отладочных сообщений
-delay uint
    пауза между запросами в миллисекундах (по умолчанию 50)
-packet string
    UDP пакет для отправки
-proto string
    tcp (по умолчанию "tcp")
-wait uint
    задержка одного запроса в миллисекундах (по умолчанию 100)
```

### Проверка 1 сервиса

```bash
net-wait-go -addrs ya.ru:443 -debug true
```

```
2020/06/30 18:07:38 ya.ru:443 is OK
```

Код возврата 0

### Проверка 2 сервисов

```bash
net-wait-go -addrs ya.ru:443,yandex.ru:443 -debug true
```

```
2020/06/30 18:09:24 yandex.ru:443 is OK
2020/06/30 18:09:24 ya.ru:443 is OK
```

Код возврата 0

### Проверка 2 сервисов (ошибка)

```bash
net-wait-go -addrs ya.ru:445,yandex.ru:445 -debug true
```

```
2020/06/30 18:09:24 yandex.ru:445 is FAILED
2020/06/30 18:09:24 ya.ru:445 is FAILED
...
```

Код возврата 1

## Поддержка UDP

Поскольку UDP как протокол не обеспечивает соединение между сервером и клиентами, он не поддерживается в большинстве популярных утилит:

- wait-for-it ([issue](https://github.com/vishnubob/wait-for-it/issues/29))
- netcat (nc) имеет следующую заметку на странице руководства:

**CAVEATS**
> UDP port scans will always succeed (i.e. report the port as open)

net-wait-go предоставляет поддержку UDP, работая следующим образом:
- отправляет осмысленный пакет на сервер
- ждёт ответного сообщения от сервера (минимум 1 байт)

### Пример UDP пакета

Игровой сервер Counter Strike доступен через UDP. Давайте проверим случайный сервер Counter Strike, отправив пакет A2S_INFO ([документация](https://developer.valvesoftware.com/wiki/Server_queries#A2S_INFO)):

```bash
net-wait-go -proto udp -addrs 46.174.53.245:27015,185.158.113.136:27015 -packet '/////1RTb3VyY2UgRW5naW5lIFF1ZXJ5AA==' -debug true
```

```
2020/07/12 15:13:25 udp 185.158.113.136:27015 is OK
2020/07/12 15:13:25 udp 46.174.53.245:27015 is OK
```

Код возврата 0

Значение `-packet` здесь — это пакет A2S_INFO, закодированный в base64, который документирован [здесь](https://github.com/wriley/steamserverinfo/blob/master/steamserverinfo.go#L133).

## Исходный код

Доступен здесь — [github.com/antelman107/net-wait-go](https://github.com/antelman107/net-wait-go).

