+++
title = 'Создание минимального Docker образа для Go приложений'
date = 2020-06-29T09:00:00+03:00
draft = false
tags = ['docker', 'compilation', 'upx', 'ldflags']
featured_image = 'docker.svg'
url = '/en/post/go-minimum-docker-image.html'

[quiz]
  [[quiz.questions]]
    question = "Какой базовый Docker образ самый маленький?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "scratch"
      correct = true
    [[quiz.questions.answers]]
      text = "alpine"
      correct = false
    [[quiz.questions.answers]]
      text = "ubuntu"
      correct = false
  
  [[quiz.questions]]
    question = "Какие флаги удаляют отладочную информацию из бинарного файла?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "-s -w"
      correct = true
    [[quiz.questions.answers]]
      text = "-d -v"
      correct = false
    [[quiz.questions.answers]]
      text = "-O -g"
      correct = false
  
  [[quiz.questions]]
    question = "Какой инструмент используется для сжатия бинарного файла?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "upx"
      correct = true
    [[quiz.questions.answers]]
      text = "gzip"
      correct = false
    [[quiz.questions.answers]]
      text = "zip"
      correct = false
  
  [[quiz.questions]]
    question = "Какова цель двухэтапной сборки Docker?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Уменьшить размер итогового образа"
      correct = true
    [[quiz.questions.answers]]
      text = "Ускорить компиляцию"
      correct = false
    [[quiz.questions.answers]]
      text = "Включить кэширование"
      correct = false
+++

Давайте обсудим, как создать минимальный Docker образ для Go программы.

<!--more-->

Docker поддерживается многими операционными системами — Linux, Unix, Windows и macOS. Docker контейнеры также поддерживаются многими популярными хостинг-платформами — Microsoft Azure, Amazon Web Services, Digital Ocean и другими.

## Сборка образа и Dockerfile

Сборка Docker образа выполняется с помощью команды `docker build`, а Dockerfile необходим для предоставления инструкций по сборке.

Давайте посмотрим на следующий Dockerfile:

```dockerfile
FROM rhaps1071/golang-1.14-alpine-git AS build
WORKDIR /build
COPY . .
RUN CGO_ENABLED=0 GOARCH=amd64 GOOS=linux go build -ldflags "-s -w -extldflags '-static'" -o ./app
RUN apk add upx
RUN upx ./app

FROM scratch
COPY --from=build /build/app /app

ENTRYPOINT ["/app"]
```

Поскольку мы используем Go модули, нам не нужно помещать исходный код в определенную папку или загружать зависимости вручную — это делается самой командой `go build`.

Теперь давайте обсудим все особенности этого Dockerfile.

## Двухэтапная сборка

Размер Docker образа важен, потому что образ будет загружаться и скачиваться во время сборки и развертывания. Чем больше размер образа, тем больше времени вы потратите на передачу по сети.

На первом этапе ("build") в Dockerfile я начинаю сборку с образа `rhaps1071/golang-1.14-alpine-git`, который представляет собой `golang:1.14-alpine` с установленным git.
Alpine — это самый маленький Linux образ, доступный в настоящее время. Размер его образа составляет около 3MB.

На втором этапе я использую `scratch` просто для размещения бинарного файла там.
`scratch` — это готовый Docker образ с нулевым размером. Это означает, что результирующий размер образа равен размеру бинарного файла.

## Размер бинарного файла

Мы можем уменьшить размер бинарного файла, используя два подхода:

- Флаги линковки (`ldflags`);
- Сжатие бинарного файла после компиляции;

Флаги `-s -w` удаляют отладочную информацию из бинарного файла. В моем случае размер был уменьшен с 15MB до 11MB благодаря этим флагам.

`-extldflags '-static'` используется для статической линковки бинарного файла даже при включенном CGO. Динамическая линковка делает наш бинарный файл зависимым от внешних библиотек (файлы `.so` в Linux).
Зависимости внешних библиотек можно проверить с помощью команды `ldd` для бинарного файла.
В `scratch` и Alpine у нас не установлены эти библиотеки. В этом случае нам приходится использовать статическую линковку — все библиотеки будут скомпилированы в наш бинарный файл программы.

Я также использовал инструмент `upx` для сжатия бинарного файла. Размер был уменьшен с 11MB до 4.1MB.

Исходный код этого примера можно найти в репозитории GitHub.

