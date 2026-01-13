+++
title = 'Compilation and deploy via SSH in Gitlab CI'
date = 2020-05-08T09:00:00+03:00
draft = false
tags = ['gitlab', 'compilation', 'deploy', 'ssh', 'modules', 'vendor', 'ci/cd']
featured_image = 'gitlab_logo.svg'
url = '/en/post/go-build-and-deploy-ssh-gitlab.html'

[quiz]
  [[quiz.questions]]
    question = "What file is needed in the repository root to enable GitLab CI?"
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
    question = "What file is required for Go commands to work outside $GOPATH?"
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
    question = "What is the vendor approach used for?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "To store dependency source code with project code"
      correct = true
    [[quiz.questions.answers]]
      text = "To download dependencies automatically"
      correct = false
    [[quiz.questions.answers]]
      text = "To compile faster"
      correct = false
  
  [[quiz.questions]]
    question = "What section in .gitlab-ci.yml allows saving files for download or passing to following stages?"
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

Let's take a look at how Go compilation works and how to use GitLab CI for that.

<!--more-->

## Compilation

To compile a Go program, we should run `go build -o binary_name`.

If there are imports of third-party code, that means there are external dependencies in the project. To build a binary, all source code is required (including the source code of third-party libraries).

## Vendor or Loading

Currently, the latest Go version is 1.14.2. Starting from Go 1.11, there is Go modules functionality in Go.

Go modules make Go download every dependency when `go run` or `go build` is called. Also, we can explicitly download dependencies by calling `go get ./...`.

Downloaded dependency code is cached in `$GOPATH/pkg/mod` or in `$HOME/go/pkg/mod`, if the `$GOPATH` variable is not set.

There is also a vendor approach that can be used for dependencies. If the `-mod=vendor` parameter is set, dependencies will be downloaded to the `vendor` folder. That folder can be located inside the project repository and stored in Git.

If dependency source code is stored with the project code, there is no need to download dependencies.

## GitLab

GitLab allows us to run some actions after a code push.

To make GitLab do something with your code, we need to put a `.gitlab-ci.yml` file with instructions into the root of the repository.

## Compilation with GitLab

We need to set particular stages and actions to make GitLab perform them. For every stage and action, we can specify a Docker image which will be downloaded.

The source code is downloaded into `/builds/{project_group}/{project_name}` folder, which means there is no need to do anything to get the source code.

I created a build stage and an action with the same name:

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

I used a custom Docker image `rhaps1071/golang-1.14-alpine-git` to perform the build. I added the `git` command to my custom Docker image, which is absent in `golang:1.14-alpine`. That is critical to make `go get ./...` work, which downloads dependencies by using `git clone`.

`golang:1.14-alpine` is based on Alpine Linux, which is very small (about 3MB). But `golang:1.14-alpine` size is 370MB because of Go in it. However, there is a 2x image size economy compared to `golang:1.14` (809MB), which is based on Ubuntu.

There is no vendor approach in the project, so I have to download the dependencies. That's why `go get ./...` is used.

As mentioned before, the source code is placed into `/builds/{project_group}/{project_name}` folder by GitLab. That path is outside `$GOPATH`. To make Go commands work outside `$GOPATH`, a `go.mod` file is required to be placed in the root of project files. That file can be created with the `go mod init` command. If there is no `go.mod` file, `go get ./...` (also `go run` and `go build`) will fail to run outside `$GOPATH`.

If there are no Go modules in your project, your source code needs to be placed inside `$GOPATH`. In GitLab CI, we can do that by copying in the following way:

```bash
cp /builds/* $GOPATH/src/
```

The `artifacts` section in `.gitlab-ci.yml` allows us to save any file from CI for manual download via the GitLab interface or to pass that file to following stages/actions.

## Deploy via SSH

Let's take a look at a simple binary deploy via SSH:

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

Here we are using a custom Docker image based on Alpine Linux again — `kroniak/ssh-client` size is 12.1MB. This time there is an SSH client installed, which allows us to use `ssh` and `scp` commands.

The deploy logic is as follows:

- The following GitLab variables are set:
  - `$SSH_PRIVATE_KEY` — private SSH key to access the server
  - `$SSH_USER`, `$SSH_HOST`, `$SSH_PORT` — SSH credentials
  - `$SSH_KNOWN_HOSTS` — data for `.ssh/known_hosts` file, which helps us validate that we are deploying to the exact server
  - `$CONFIG` — contents of the service config file in JSON
- On deploy, we put data from some GitLab variables into files
- All files are copied with the `scp` command. We could use `rsync` here because it only copies changed files. But since there are only 2 files that need to be copied, it doesn't matter
- Finally, we change our binary and restart the service

Full contents of `.gitlab-ci.yml` is below:

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