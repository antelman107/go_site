+++
title = 'Deploy docker swarm from Gitlab CI'
date = 2020-07-01T09:00:00+03:00
draft = false
tags = ['docker', 'deploy', 'gitlab', 'ci/cd']
featured_image = 'docker.svg'
url = '/en/post/deploy-docker-swarm-gitlab.html'

[quiz]
  [[quiz.questions]]
    question = "What command initializes Docker Swarm?"
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
    question = "What are the advantages of Docker over binary deployment?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "No need for systemd or supervisor"
      correct = true
    [[quiz.questions.answers]]
      text = "No need to install software on the server"
      correct = true
    [[quiz.questions.answers]]
      text = "Simplified hosting deployment"
      correct = true
    [[quiz.questions.answers]]
      text = "Faster compilation"
      correct = false
  
  [[quiz.questions]]
    question = "What command is used to deploy a stack in Docker Swarm?"
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
    question = "What format does docker stack deploy use for configuration?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "docker-compose format"
      correct = true
    [[quiz.questions.answers]]
      text = "YAML format"
      correct = false
    [[quiz.questions.answers]]
      text = "JSON format"
      correct = false
+++

Docker container deployment has many advantages over binary deployment. Let's dive into advantages and see how we can implement docker container deployment from Gitlab CI.
Let's talk about how standard docker can help us. Any docker orchestration tools (Kubernetes, for example) will be observed in future articles.

<!--more-->

Recently we discussed a binary file deployment via SSH.

Now we are discussing Docker container deployment with Gitlab CI.

## Docker advantages

There are some of them against binary file deployment:

- We don't need a software to start/restart our service besides docker utility. When using binary file deployment we need an additional software like systemd or supervisor.
- No need to install any additional software directly on the server where docker container is going to be started (again, besides docker utility.) All necessary additional programs could be installed inside our docker container during docker image build.
- Docker container's short lifetime allows us to change the docker image how and when we want to. We could install/update software in docker image, even change base OS of docker image. We don't have to care about old container — we just rebuild new container and deploy it. The old container will be replaced by the new one.
- Simplified hosting deployment — docker is a standard tool and many modern hosting providers (AWS, Microsoft Azure, Google Cloud) provide us with docker hosting options.
- Docker has health checking logic to monitor, start and restart containers; horizontal and vertical scaling; load balancing between containers in single service; DNS (services could access each other by short names) — all of these make docker suitable tool for any kind of modern applications.

## Docker image building

Recently we discussed minimal docker image building for GO program. Let's continue to use that approach.

### Deployment
We are going to use only standard docker utility options. We intentionally ignore any external orchestration tools like kubernetes or nomad; We won't use any cloud docker provider, for example, Google Cloud — all of it we'll discuss in future articles.

### Docker swarm

We choose docker swarm, it is a part of standard docker utility.
Swarm allows us to manage containers on a remote server that is previously configured with docker swarm init or docker swarm join commands.
These commands change docker service mode to swarm.

SSH access to the target server is not required, which is an advantage. docker stack deploy updates containers using file in docker-compose format, so we even don't need to install docker-compose.

When using docker stack deploy, which is accessible when docker service runs in swarm mode, we can pass the exact order in which we want our containers to be updated and many other deployment options. By this we can provides smooth update without downtime. Also, features like scaling, load balancing and DNS, which i highlighted before, are accessible only in swarm mode.

## Target server preparation

We have to access target server via SSH and change it's mode to swarm:

```shell
docker swarm init
```

This command also prints tokens which can be used later from other servers to join the swarm by using docker swarm join --token xxx command. But in this article, we only have one server and we are not going to use docker swarm join.

We are going to use remote connection from Gitlab CI to our docker service on target server. So we are changing docker service configuration to make docker service remotely accessible. To do this create/change docker service configuration for systemd:

```shell
sudo -s
mkdir -p /etc/systemd/system/docker.service.d
touch /etc/systemd/system/docker.service.d/startup_options.conf
```

`startup_options.conf` contents:

```ini
[Service]
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2376
```

Restart systemd and docker service:

```shell
systemctl daemon-reload
```

Now docker service is remotely accessible.

## .gitlab-ci.yml preparation

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

We changed the DOCKER_HOST variable to make docker command access our remote server which we configured previously.

Our image is stored in private repository, to access which we used docker login command. To make docker stack deploy command use authorization granted by docker login, `--with-registry-auth` parameter is set.

YAML configuration file `./docker/stage/app.yml` contents:
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

`docker stack deploy` is very helpful because it uses environment variables to fill the configuration. We use `${DOCKER_REPO}`, `${DOCKER_IMAGE}` and `CONFIG`.

Deploy structure in yaml sets up deployment:

- `order: start-first` makes new container start before old container is stopped which makes our deploy downtime-less.
- `parallelism: 2` sets how many container pairs are going to be updated in one time, which affect on server load during the update.
- `monitor: 1m` sets the time which docker service will monitor new containers to make decision if update is successful.
- `failure_action: rollback` sets behaviour if one of new containers is failed — rollback is chosen.

