+++
title = 'Деплой docker swarm из Gitlab CI'
date = 2020-07-01T09:00:00+03:00
draft = false
tags = ['docker', 'deploy', 'gitlab', 'ci/cd']
featured_image = 'docker.svg'
url = '/ru/post/deploy-docker-swarm-gitlab.html'

[quiz]
  [[quiz.questions]]
    question = "Какая команда инициализирует Docker Swarm?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "docker swarm init"
      correct = true
    [[quiz.questions.answers]]
      text = "docker swarm start"
      correct = false
    [[quiz.questions.answers]]
      text = "docker init swarm"
      correct = false
  
  [[quiz.questions]]
    question = "Каковы преимущества Docker перед деплоем бинарного файла?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "Не требуется systemd или supervisor"
      correct = true
    [[quiz.questions.answers]]
      text = "Не нужно устанавливать ПО на сервер"
      correct = true
    [[quiz.questions.answers]]
      text = "Упрощенный деплой на хостинг"
      correct = true
    [[quiz.questions.answers]]
      text = "Быстрее компиляция"
      correct = false
  
  [[quiz.questions]]
    question = "Какая команда используется для деплоя стека в Docker Swarm?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "docker stack deploy"
      correct = true
    [[quiz.questions.answers]]
      text = "docker deploy stack"
      correct = false
    [[quiz.questions.answers]]
      text = "docker swarm deploy"
      correct = false
  
  [[quiz.questions]]
    question = "Какой формат использует docker stack deploy для конфигурации?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Формат docker-compose"
      correct = true
    [[quiz.questions.answers]]
      text = "Формат YAML"
      correct = false
    [[quiz.questions.answers]]
      text = "Формат JSON"
      correct = false
+++

Ранее мы рассматривали деплой бинарного файла программы через SSH на сервер

Сейчас мы рассмотрим как реализовать деплой Docker контейнера из Gitlab CI.
<!--more-->

## Плюсы Docker

При использовании Docker существует множество плюсов сравнительно с деплоем бинарного файла:

- Для запуска/перезапуска в случае падения сервиса не потребуется ПО, кроме утилиты docker. В то время, как для запуска бинарного файла потребуется программы типа systemd или supervisor.
- Нет необходимости устанавливать или обновлять на сервере, где запускается docker-контейнер. Все необходимые дополнительные программы могут быть установлены внутрь контейнера нашей программы.
- Ограниченный срок жизни контейнера позволяет нам легко менять этот контейнер — добавлять или убирать софт, менять базовую ОС. При этом нам нет необходимости поддерживать старый вариант контейнера - мы меняем процедуру сборки и выкатываем новый контейнер вместо старого.
- Упрощенный деплой на сторонний хостинг — множество современных хостингов (AWS, Microsoft Azure, Google Cloud) работают с docker-контейнерами.
- В docker существуют механизмы healthcheck для оценивания работоспособности и перезапуска контейнеров; горизонтальное и вертикальное масштабирование; балансирование нагрузки между контейнерами; DNS (сервисы могут обращаться друг к другу по простым, заранее известным именам).

## Сборка образа

Ранее мы рассмотрели сборку минимально возможного образа с программой GO. Для сборки образа далее будем использовать данный подход.

### Деплой

Будем пользоваться только базовыми возможностями консольной команды docker, намеренно не будем применять никакие оркестраторы вроде kubernetes,nomad; не будем использовать облачные хостинги вроде Google Cloud — все это потребует очень много дополнительных объяснений и будет рассмотрено отдельно.

### Docker swarm

Для деплоя будем использовать docker swarm, который входит в базовую функциональность docker.
Swarm позволяет управлять запуском docker-контейнеров на удаленном сервере, на котором заранее была выполнена команда docker swarm init (создание нового swarm) или docker swarm join (подключение к существующему swarm). После выполнения этих команд docker на текущем сервере переходит в swarm-режим.

Для деплоя в docker swarm нам не требуется SSH-доступ на сервер. Команда docker stack deploy позволяет выполнить обновление docker-контейнеров на сервере по конфигурационному файлу в формате docker-compose, так что даже дополнительную утилиту docker-compose устанавливать не требуется.

Также именно при вызове docker stack deploy мы можем указать не только список контейнеров, но также объединить несколько контейнеры в сервисы с масштабированием нагрузки по контейнерам внутри сервиса; можем указать политику обновления контейнеров, те обеспечить production-ready деплой с резервированием и без простоев. Данный способ деплоя является наилучшей альтернативой при использовании лишь утилиты docker.

## Подготовка сервера

Необходимо зайти по SSH по сервер, на котором предполагается запуск контейнеров, и перевести docker-сервер в swarm режим:

```shell
docker swarm init
```

Данная команда также выведет токены для подключения других серверов к данному swarm с помощью команды docker swarm join --token xxx Однако, для такой работы нам потребуются другие серверы. Мы же будем работать всего с одним сервером. Мы не будем использовать docker swarm join.

Изменим запуск docker-сервиса таким образом, чтобы сервер был доступен удаленно (не только с localhost). Создадим или изменим файл службы docker для systemd:

```shell
sudo -s
mkdir -p /etc/systemd/system/docker.service.d
touch /etc/systemd/system/docker.service.d/startup_options.conf
```

Содержимое файла `startup_options.conf`:

```ini
[Service]
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2376
```

Перезапустим службу:

```shell
systemctl daemon-reload
```

Теперь Docker-сервис на данном сервере может принимать удаленные подключения.

## Подготовка .gitlab-ci.yml

Для деплоя docker стэка на удаленный сервер, работающий в swarm режиме, мы реализовали следующий шаг в .gitlab-ci.yml:

```yaml
deploy_docker_stage:
  environment:
    name: stage
    url: https://someurl
  image: docker:19.03.1
  services:
    - docker:19.03.1-dind
  stage: deploy
  variables:
    DOCKER_HOST: "tcp://${DOCKER_HOST}:2376"
  before_script:
    - apk add jq
    - docker login -u $USER -p $PASSWORD $DOCKER_REPO
  script:
    - export CONFIG=$(echo $CONFIG1_STAGE | jq -c) && docker stack deploy --with-registry-auth -c ./docker/stage/app.yml app
```

Переопределяем переменную DOCKER_HOST для того, чтобы команда docker в Gitlab CI взаимодействовала с удаленным сервером, который мы настроили в предыдущем параграфе.

Наш образ хранится в приватном репозитории, для доступа к которому используем docker login. Чтобы наша команда docker stack deploy использовала полученную авторизации в репозитории, указываем параметр `--with-registry-auth`.

YAML-файл конфигурации `./docker/stage/app.yml`, имеет следующий вид:

```yaml
version: '3.8'
services:
  app:
    image: ${DOCKER_REPO}/${DOCKER_IMAGE}
    environment:
      - CONFIG
    ports:
      - "9091:9091"
      - "8444:8444"
    depends_on:
      - postgres
      - redis
    deploy:
      replicas: 3
      update_config:
        parallelism: 2
        delay: 3s
        order: start-first
        failure_action: rollback
        monitor: 1m
      restart_policy:
        max_attempts: 3
```

`docker stack deploy` использует переменные окружения для формирования окончательного вида конфигурации. Поэтому в файле используются переменные `${DOCKER_REPO}`, `${DOCKER_IMAGE}` и `CONFIG`.

Вся структура deploy в yaml файле указывает на то, как выполняется деплой:

- `order: start-first` отвечает за то, чтобы сначала запустился контейнер новой версии контейнера, а затем уже остановился старый — та самая бесперебойность.
- `parallelism: 2` определяет сколько пар контейнеров могут быть одновременно обновлены. Чем больше данный параметр, тем больше будет нагрузка на сервер во время деплоя.
- `monitor: 1m` указывает сколько времени после создания контейнера docker-сервис следит за его состоянием, чтобы понять, был ли деплой успешен.
- `failure_action: rollback` определяет что делать с деплоем в случае проблемы и в данном случае выбран откат до предыдущей версии.