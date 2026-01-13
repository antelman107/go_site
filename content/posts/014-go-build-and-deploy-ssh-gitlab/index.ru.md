+++
title = 'Компиляция и деплой через SSH в GitLab CI'
date = 2020-05-08T09:00:00+03:00
draft = false
tags = ['gitlab', 'compilation', 'deploy', 'ssh', 'modules', 'vendor', 'ci/cd']
featured_image = 'gitlab_logo.svg'
url = '/ru/post/go-build-and-deploy-ssh-gitlab.html'

[quiz]
  [[quiz.questions]]
    question = "Какой файл нужен в корне репозитория для включения GitLab CI?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = ".gitlab-ci.yml"
      correct = true
    [[quiz.questions.answers]]
      text = "gitlab.yml"
      correct = false
    [[quiz.questions.answers]]
      text = "ci.yml"
      correct = false
  
  [[quiz.questions]]
    question = "Какой файл требуется для работы команд Go вне $GOPATH?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "go.mod"
      correct = true
    [[quiz.questions.answers]]
      text = "go.sum"
      correct = false
    [[quiz.questions.answers]]
      text = "vendor.json"
      correct = false
  
  [[quiz.questions]]
    question = "Для чего используется подход vendor?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Для хранения исходного кода зависимостей вместе с кодом проекта"
      correct = true
    [[quiz.questions.answers]]
      text = "Для автоматической загрузки зависимостей"
      correct = false
    [[quiz.questions.answers]]
      text = "Для более быстрой компиляции"
      correct = false
  
  [[quiz.questions]]
    question = "Какая секция в .gitlab-ci.yml позволяет сохранять файлы для загрузки или передачи следующим этапам?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "artifacts"
      correct = true
    [[quiz.questions.answers]]
      text = "files"
      correct = false
    [[quiz.questions.answers]]
      text = "output"
      correct = false
+++

Давайте рассмотрим, как работает компиляция Go и как использовать GitLab CI для этого.

<!--more-->

## Компиляция

Для компиляции программы Go мы должны выполнить `go build -o binary_name`.

Если есть импорты стороннего кода, это означает, что в проекте есть внешние зависимости. Для сборки бинарного файла требуется весь исходный код (включая исходный код сторонних библиотек).

## Vendor или загрузка

В настоящее время последняя версия Go — 1.14.2. Начиная с Go 1.11, в Go есть функциональность Go modules.

Go modules заставляют Go загружать каждую зависимость при вызове `go run` или `go build`. Также мы можем явно загрузить зависимости, вызвав `go get ./...`.

Загруженный код зависимостей кэшируется в `$GOPATH/pkg/mod` или в `$HOME/go/pkg/mod`, если переменная `$GOPATH` не установлена.

Также есть подход vendor, который можно использовать для зависимостей. Если установлен параметр `-mod=vendor`, зависимости будут загружены в папку `vendor`. Эта папка может находиться внутри репозитория проекта и храниться в Git.

Если исходный код зависимостей хранится вместе с кодом проекта, нет необходимости загружать зависимости.

## GitLab

GitLab позволяет нам выполнять некоторые действия после отправки кода.

Чтобы заставить GitLab что-то делать с вашим кодом, нам нужно поместить файл `.gitlab-ci.yml` с инструкциями в корень репозитория.

## Компиляция с GitLab

Нам нужно установить определенные этапы и действия, чтобы GitLab их выполнял. Для каждого этапа и действия мы можем указать Docker-образ, который будет загружен.

Исходный код загружается в папку `/builds/{project_group}/{project_name}`, что означает, что нет необходимости что-либо делать для получения исходного кода.

Я создал этап сборки и действие с тем же именем:

```yaml
stages:
  - build
build:
  image: rhaps1071/golang-1.14-alpine-git
  stage: build
  script:
    - go get ./...
    - GOARCH=amd64 GOOS=linux go build -ldflags "-extldflags '-static'" -o $CI_PROJECT_DIR/binary
  artifacts:
    paths:
      - binary
```

Я использовал пользовательский Docker-образ `rhaps1071/golang-1.14-alpine-git` для выполнения сборки. Я добавил команду `git` в свой пользовательский Docker-образ, которой нет в `golang:1.14-alpine`. Это критично для работы `go get ./...`, который загружает зависимости, используя `git clone`.

`golang:1.14-alpine` основан на Alpine Linux, который очень мал (около 3MB). Но размер `golang:1.14-alpine` составляет 370MB из-за Go в нем. Однако есть экономия размера образа в 2 раза по сравнению с `golang:1.14` (809MB), который основан на Ubuntu.

В проекте нет подхода vendor, поэтому мне приходится загружать зависимости. Вот почему используется `go get ./...`.

Как упоминалось ранее, исходный код размещается GitLab в папку `/builds/{project_group}/{project_name}`. Этот путь находится вне `$GOPATH`. Чтобы команды Go работали вне `$GOPATH`, требуется файл `go.mod`, который должен быть помещен в корень файлов проекта. Этот файл можно создать командой `go mod init`. Если нет файла `go.mod`, `go get ./...` (также `go run` и `go build`) не будут работать вне `$GOPATH`.

Если в вашем проекте нет Go modules, ваш исходный код должен быть помещен внутрь `$GOPATH`. В GitLab CI мы можем сделать это, скопировав следующим образом:

```bash
cp /builds/* $GOPATH/src/
```

Секция `artifacts` в `.gitlab-ci.yml` позволяет нам сохранить любой файл из CI для ручной загрузки через интерфейс GitLab или передать этот файл следующим этапам/действиям.

## Деплой через SSH

Давайте рассмотрим простой деплой бинарного файла через SSH:

```yaml
deploy_stage:
  image: kroniak/ssh-client
  stage: deploy
  environment:
    name: stage
    url: http://stage.project.com
  when: manual
  script:
    - mkdir -p ~/.ssh
    - echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
    - chmod -R 700 ~/.ssh
    - echo "$SSH_KNOWN_HOSTS" >> ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts
    - echo "$CONFIG" > ./config.json
    - scp -P$SSH_PORT -r ./config.json $SSH_USER@$SSH_HOST:/var/www/project/config/
    - scp -P$SSH_PORT -r ./binary $SSH_USER@$SSH_HOST:~/binary_tmp
    - ssh -p$SSH_PORT $SSH_USER@$SSH_HOST 'sudo service project stop && cp ~/binary_tmp /var/www/project/binary && sudo service project restart'
```

Здесь мы снова используем пользовательский Docker-образ на основе Alpine Linux — размер `kroniak/ssh-client` составляет 12.1MB. На этот раз установлен SSH-клиент, который позволяет нам использовать команды `ssh` и `scp`.

Логика деплоя следующая:

- Установлены следующие переменные GitLab:
  - `$SSH_PRIVATE_KEY` — приватный SSH-ключ для доступа к серверу
  - `$SSH_USER`, `$SSH_HOST`, `$SSH_PORT` — SSH-учетные данные
  - `$SSH_KNOWN_HOSTS` — данные для файла `.ssh/known_hosts`, которые помогают нам проверить, что мы деплоим на нужный сервер
  - `$CONFIG` — содержимое файла конфигурации сервиса в формате JSON
- При деплое мы помещаем данные из некоторых переменных GitLab в файлы
- Все файлы копируются командой `scp`. Мы могли бы использовать `rsync` здесь, потому что он копирует только измененные файлы. Но поскольку нужно скопировать только 2 файла, это не имеет значения
- Наконец, мы меняем наш бинарный файл и перезапускаем сервис

Полное содержимое `.gitlab-ci.yml` ниже:

```yaml
stages:
  - build
  - deploy

build:
  image: rhaps1071/golang-1.14-alpine-git
  stage: build
  script:
    - go get ./...
    - GOARCH=amd64 GOOS=linux go build -ldflags "-extldflags '-static'" -o $CI_PROJECT_DIR/binary
  artifacts:
    paths:
      - binary

deploy_stage:
  image: kroniak/ssh-client
  stage: deploy
  environment:
    name: stage
    url: http://stage.project.com
  when: manual
  script:
    - mkdir -p ~/.ssh
    - echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
    - chmod -R 700 ~/.ssh
    - echo "$SSH_KNOWN_HOSTS" >> ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts
    - echo "$CONFIG" > ./config.json
    - scp -P$SSH_PORT -r ./config.json $SSH_USER@$SSH_HOST:/var/www/project/config/
    - scp -P$SSH_PORT -r ./binary $SSH_USER@$SSH_HOST:~/binary_tmp
    - ssh -p$SSH_PORT $SSH_USER@$SSH_HOST 'sudo service project stop && cp ~/binary_tmp /var/www/project/binary && sudo service project restart'
```

