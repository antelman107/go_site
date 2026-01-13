+++
title = 'Удалённая отладка с Delve'
date = 2020-07-01T09:00:00-07:00
draft = false
tags = ['docker', 'debugger', 'delve', 'vscode', 'goland']
url = '/ru/post/go-docker-delve-remote-debug.html'

[quiz]
  [[quiz.questions]]
    question = "Что такое Delve?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Инструмент отладки для Go"
      correct = true
    [[quiz.questions.answers]]
      text = "Фреймворк для тестирования"
      correct = false
    [[quiz.questions.answers]]
      text = "Форматтер кода"
      correct = false
  
  [[quiz.questions]]
    question = "Когда полезна удалённая отладка?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "Когда программу нельзя протестировать локально"
      correct = true
    [[quiz.questions.answers]]
      text = "Когда ошибки воспроизводятся только в удалённой среде"
      correct = true
    [[quiz.questions.answers]]
      text = "Когда нужна более быстрая отладка"
      correct = false
  
  [[quiz.questions]]
    question = "Какие флаги компиляции нужны для правильной отладки с Delve?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "-gcflags 'all=-N -l'"
      correct = true
    [[quiz.questions.answers]]
      text = "-ldflags '-s -w'"
      correct = false
    [[quiz.questions.answers]]
      text = "-tags debug"
      correct = false
  
  [[quiz.questions]]
    question = "Как работает Delve с IDE?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Delve запускается как сервер, и IDE подключается к нему"
      correct = true
    [[quiz.questions.answers]]
      text = "IDE запускает Delve напрямую"
      correct = false
    [[quiz.questions.answers]]
      text = "Delve встроен в IDE"
      correct = false
+++

Ранее мы обсуждали локальную отладку с помощью IDE GoLand. Теперь мы обсудим, как удалённо отлаживать программу, работающую внутри Docker-контейнера, используя Visual Studio Code и GoLand IDE.

<!--more-->

При локальной отладке IDE управляет всем — компилирует программу, запускает её и подключается к ней.

Однако иногда может потребоваться выполнить удалённую отладку. В этом случае ваша программа должна запускаться независимо от IDE. Подготовка программы к удалённой отладке будет задачей разработчика или DevOps-инженера.

## Зачем нужна удалённая отладка?

Это может показаться редким случаем использования, но вот ситуации, когда она может понадобиться:

- Программу нельзя протестировать локально, только в определённой удалённой среде
- Необходимо проанализировать ошибку, которая воспроизводится только в удалённой среде — например, во время интеграционного тестирования

Перед началом реализации удалённой отладки необходимо понимать, что ошибки легче исправлять на ранних этапах разработки. При локальной отладке вы можете запустить программу с помощью `go run`, `delve debug` или любой IDE. При удалённой отладке вам придётся ждать развёртывания программы в удалённой среде.

## Delve

Инструмент отладки, используемый в IDE GoLand или Visual Studio Code, — это Delve.

Отладка Delve + IDE всегда работает следующим образом:

1. Delve запускается как серверное приложение, прослушивая сетевые подключения на определённом порту
2. Delve запускает нашу программу (скомпилированный бинарный файл или исходный код Go)
3. Delve позволяет IDE GoLand или Visual Studio Code подключиться к нему и получает данные о точках останова
4. В точке останова Delve приостанавливает программу и отправляет данные о переменных в IDE

Delve — это инструмент командной строки. Все его параметры CLI определены [здесь](https://github.com/go-delve/delve/tree/master/Documentation/cli).

Помимо сетевого API, Delve также имеет опции отладки из командной строки для отладки напрямую из командной строки (без IDE). Однако мы не будем использовать эту опцию.

Итак, нам нужно запустить Delve и обеспечить удалённое подключение к нему из нашей IDE.

## Исходный код

Для лучшего понимания этой статьи я создал репозиторий GitHub со всем связанным кодом.

## Docker

Docker-контейнеры — популярный тип развёртывания. Недавно мы обсуждали создание минимального Docker-образа и развёртывание Docker Swarm. Docker-контейнеры можно использовать здесь для демонстрации удалённого подключения.

Давайте настроим Dockerfile:

```dockerfile
FROM rhaps1071/golang-1.14-alpine-git AS build
WORKDIR /
COPY . .
RUN CGO_ENABLED=0 go get -ldflags "-s -w -extldflags '-static'" github.com/go-delve/delve/cmd/dlv
RUN CGO_ENABLED=0 go build -gcflags "all=-N -l" -o ./app

FROM scratch
COPY --from=build /go/bin/dlv /dlv
COPY --from=build /app /app
ENTRYPOINT [ "/dlv" ]
```

Здесь я использовал двухэтапную сборку и статическую компиляцию для создания минимального Docker-образа `FROM scratch`. В результирующем Docker-образе есть два файла: `/dlv` — Delve; `/app` — наша программа.

Флаги компиляции Go поддерживаются многими инструментами Go: `build`, `clean`, `get`, `install`, `list`, `run`, `test`. Благодаря этому бинарный файл Delve, полученный с помощью `go get`, также статически скомпилирован. По умолчанию бинарный файл Delve, скомпилированный в среде Alpine Linux, зависит от двух библиотек. Это можно проверить с помощью инструмента `ldd`:

```bash
ldd /go/bin/dlv
/lib/ld-musl-x86_64.so.1 (0x7f761f8b7000)
libc.musl-x86_64.so.1 => /lib/ld-musl-x86_64.so.1 (0x7f761f8b7000)
```

Флаги `-gcflags "all=-N -l"`, которые используются для компиляции основной программы, необходимы для правильной отладки нашей программы с помощью Delve.

Для сборки контейнера можно использовать `docker build -f ./docker/debug/Dockerfile -t debug .`, что реализовано как команда Makefile `docker-build-debug`, поэтому та же команда будет работать как `make docker-build-debug`.

## Запуск контейнера

Давайте запустим наш образ как контейнер, используя docker-compose.

Содержимое `docker-compose.yml` ниже:

```yaml
version: "3"

services:
  debug:
    build:
      dockerfile: docker/debug/Dockerfile
      context: ../../
    ports:
      - 2345:2345
    command: "--listen=:2345 --headless=true --log=true --log-output=debugger,debuglineerr,gdbwire,lldbout,rpc --accept-multiclient --api-version=2 exec ./app"
```

Ранее мы настроили `ENTRYPOINT` в нашем образе как `/dlv`, поэтому теперь в параметре `command` мы передаём только аргументы Delve. Указанные выше параметры передаются, чтобы Delve работал как сетевой сервер и включал подробное логирование.

Давайте вызовем `docker-compose -f ./docker/debug/docker-compose.yml up` и проверим логи. Эта команда также доступна как `make docker-run-debug`.

```
Starting debug_debug_1 ... done
Attaching to debug_debug_1
debug_1  | API server listening at: [::]:2345
debug_1  | 2020-07-10T13:36:06Z debug layer=rpc API server pid = 1
debug_1  | 2020-07-10T13:36:06Z info layer=debugger launching process with args: [./app]
```

Логи показывают, что сервер отладки запущен на порту 2345 и готов принимать подключения.

Давайте подключимся к нему из IDE.

## Visual Studio Code

Нам нужно открыть наш проект в IDE. Исходный код представляет программу в нашем контейнере.

Давайте создадим/изменим файл `.vscode/launch.json` со следующей конфигурацией:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Attach",
      "type": "go",
      "request": "attach",
      "mode": "remote",
      "remotePath": "",
      "port": 2345,
      "host": "127.0.0.1",
      "showLog": true,
      "trace": "log",
      "logOutput": "rpc"
    }
  ]
}
```

- `"request": "attach"` — позволяет VS Code подключиться к запущенному экземпляру Delve вместо запуска новой сессии отладки локально
- `port`, `host` — сетевой хост и порт нашего Delve. Используя docker-compose, мы запустили Docker-контейнер локально и опубликовали его порт 2345 на хост (localhost)
- `remotePath` — один из критических параметров, который влияет на успешную установку точек останова. Это путь к папке с исходным кодом, которая использовалась на этапе компиляции. Мы скомпилировали наш бинарный файл в Dockerfile, используя `WORKDIR /`, поэтому наша директория — это корневая директория — оставим `remotePath` пустым

Перед выполнением каких-либо действий в IDE давайте проверим, что наш Docker-контейнер сейчас работает.

Затем запустите задачу отладки "Attach". Мы должны установить точки останова и убедиться, что они остаются на месте (визуально в IDE). Каждая установка точки останова отражается в логах Docker-контейнера — ошибок быть не должно.

## GoLand IDE

Все настройки в этой IDE можно изменить в графическом интерфейсе, без редактирования файлов конфигурации. Важная вещь — включить интеграцию модулей:

Нажмите **Run → Edit configurations**, добавьте новую конфигурацию отладки и выберите "Go Remote":

Так же, как и в VS Code, мы должны установить точки останова и убедиться, что они на месте. Мы должны проверить логи Docker-контейнера. Если есть какие-либо ошибки, например `could not find file somedir/main.go`, вам нужно включить интеграцию модулей Go.

