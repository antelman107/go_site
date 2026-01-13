+++
title = 'Remote debugging with Delve'
date = 2020-07-01T09:00:00-07:00
draft = false
tags = ['docker', 'debugger', 'delve', 'vscode', 'goland']
url = '/en/post/go-docker-delve-remote-debug.html'

[quiz]
  [[quiz.questions]]
    question = "What is Delve?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "A debugger tool for Go"
      correct = true
    [[quiz.questions.answers]]
      text = "A testing framework"
      correct = false
    [[quiz.questions.answers]]
      text = "A code formatter"
      correct = false
  
  [[quiz.questions]]
    question = "When is remote debugging useful?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "When the program can't be tested locally"
      correct = true
    [[quiz.questions.answers]]
      text = "When bugs only reproduce in remote environment"
      correct = true
    [[quiz.questions.answers]]
      text = "When you want faster debugging"
      correct = false
  
  [[quiz.questions]]
    question = "What compilation flags are needed for Delve to debug properly?"
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
    question = "How does Delve work with IDEs?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Delve runs as a server and IDE connects to it"
      correct = true
    [[quiz.questions.answers]]
      text = "IDE runs Delve directly"
      correct = false
    [[quiz.questions.answers]]
      text = "Delve is embedded in IDE"
      correct = false
+++

Previously we discussed local debugging with GoLand IDE. Now we'll discuss how to remotely debug a program running inside a Docker container using Visual Studio Code and GoLand IDE.

<!--more-->

In local debugging, the IDE manages everything — it compiles the program, starts it, and connects to it.

However, sometimes you may need to perform remote debugging. In this case, your program should be started independently of the IDE. It will be the developer's or DevOps engineer's responsibility to prepare the program for remote debugging.

## Why do we need remote debugging?

It may seem like a rare use case, but here are situations where you might need it:

- The program can't be tested locally, only in a specific remote environment
- You need to analyze a bug that is only reproducible in a remote environment — during integration testing, for example

Before starting to implement remote debugging, you must understand that bugs can be fixed more easily in the early stages of development. When debugging locally, you can run the program with `go run`, `delve debug`, or any IDE. With remote debugging, you have to wait for the program to be deployed to the remote environment.

## Delve

The debug tool used in GoLand IDE or Visual Studio Code is Delve.

Delve + IDE debugging always works like this:

1. Delve is started as a server application, listening for network connections on a specific port
2. Delve runs our program (compiled binary or Go source code)
3. Delve allows GoLand IDE or Visual Studio Code to connect to it and receives breakpoint data
4. At a breakpoint, Delve pauses the program and sends variable data to the IDE

Delve is a command-line tool. All its CLI parameters are defined [here](https://github.com/go-delve/delve/tree/master/Documentation/cli).

Besides having a network API, Delve also has command-line debugging options to debug directly from the command line (without an IDE). However, we are not going to use this option.

So we need to start Delve and provide a remote connection to it from our IDE.

## The sources

For better understanding of this article, I've created a GitHub repository with all related code.

## Docker

Docker containers are a popular type of deployment. Recently we discussed minimal Docker image building and Docker Swarm deployment. Docker containers can be used here to demonstrate remote connection.

Let's set up the Dockerfile:

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

Here I used a two-stage build and static compilation to create a minimal Docker image `FROM scratch`. There are two files in the resulting Docker image: `/dlv` — Delve; `/app` — our program.

Go compilation flags are supported by many Go tools: `build`, `clean`, `get`, `install`, `list`, `run`, `test`. Due to this, the Delve binary obtained with `go get` is also statically compiled. By default, a Delve binary compiled in an Alpine Linux environment depends on two libraries. You can check this with the `ldd` tool:

```bash
ldd /go/bin/dlv
/lib/ld-musl-x86_64.so.1 (0x7f761f8b7000)
libc.musl-x86_64.so.1 => /lib/ld-musl-x86_64.so.1 (0x7f761f8b7000)
```

The `-gcflags "all=-N -l"` flags, which are used to compile the main program, are required to make Delve debug our program properly.

To build the container, you may use `docker build -f ./docker/debug/Dockerfile -t debug .`, which is implemented as a Makefile command `docker-build-debug`, so the same command will work as `make docker-build-debug`.

## Container run

Let's start our image as a container using docker-compose.

The `docker-compose.yml` contents are below:

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

Previously we set up the `ENTRYPOINT` in our image as `/dlv`, so now in the `command` parameter we pass Delve arguments only. The parameters above are passed to make Delve work as a network server and to enable rich logging.

Let's call `docker-compose -f ./docker/debug/docker-compose.yml up` and check the logs. This command is also available as `make docker-run-debug`.

```
Starting debug_debug_1 ... done
Attaching to debug_debug_1
debug_1  | API server listening at: [::]:2345
debug_1  | 2020-07-10T13:36:06Z debug layer=rpc API server pid = 1
debug_1  | 2020-07-10T13:36:06Z info layer=debugger launching process with args: [./app]
```

The logs indicate that the debug server is started on port 2345 and is ready to accept connections.

Let's connect to it from IDEs.

## Visual Studio Code

We need to open our project in the IDE. The source code represents the program in our container.

Let's create/modify the `.vscode/launch.json` file with the following configuration:

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

- `"request": "attach"` — allows VS Code to attach to a running Delve instance, instead of starting a new debug session locally
- `port`, `host` — the network host and port of our Delve. By using docker-compose, we started the Docker container locally and published its port 2345 to the host (localhost)
- `remotePath` — one of the critical parameters that affects successful breakpoint setting. It is the path to the sources folder that was used during the compile stage. We compiled our binary in the Dockerfile using `WORKDIR /`, so our directory is the root directory — let's leave `remotePath` blank

Before doing anything in the IDE, let's check that our Docker container is currently running.

Next, run the "Attach" debug task. We should set breakpoints and ensure they stay in place (visually in the IDE). Each breakpoint setting is reflected in the Docker container logs — there should be no errors.

## GoLand IDE

All settings in this IDE can be changed in the graphical interface, without editing configuration files. An important thing — enable modules integration:

Click **Run → Edit configurations**, add a new debug configuration, and select "Go Remote":

Same as in VS Code, we should set breakpoints and ensure they are in place. We should check the Docker container logs. If there are any errors like `could not find file somedir/main.go`, you need to enable Go modules integration.
