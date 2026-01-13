+++
title = 'Building Minimal Docker Images for Go Applications'
date = 2020-06-29T09:00:00+03:00
draft = false
tags = ['docker', 'compilation', 'upx', 'ldflags']
featured_image = 'docker.svg'
url = '/ru/post/go-minimum-docker-image.html'

[quiz]
  [[quiz.questions]]
    question = "What is the smallest base Docker image?"
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
    question = "What flags remove debug information from the binary?"
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
    question = "What tool is used to compress the binary file?"
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
    question = "What is the purpose of two-stage Docker builds?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "To reduce final image size"
      correct = true
    [[quiz.questions.answers]]
      text = "To speed up compilation"
      correct = false
    [[quiz.questions.answers]]
      text = "To enable caching"
      correct = false
+++

Let's discuss how to build a minimal Docker image for a Go program.

<!--more-->

Docker is supported by many operating systems — Linux, Unix, Windows, and macOS. Docker containers are also supported by many popular hosting platforms — Microsoft Azure, Amazon Web Services, Digital Ocean, and others.

## Image Building and Dockerfile

Docker image building is performed using the `docker build` command, and a Dockerfile is required to provide building instructions.

Let's take a look at the following Dockerfile:

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

Because we are using Go modules, we don't need to put the source code in a particular folder or download dependencies manually — this is done by the `go build` command itself.

Let's now discuss all the features of this Dockerfile.

## Two-Stage Building

Docker image size is important because the image will be uploaded and downloaded during build and deployment. The larger the image size, the more time you will spend on network transmission.

In the first stage ("build") of the Dockerfile, I start building from the `rhaps1071/golang-1.14-alpine-git` image, which is `golang:1.14-alpine` with git installed.
Alpine is the smallest Linux image currently available. Its image size is about 3MB.

In the second stage, I use `scratch` just to place the binary file there.
`scratch` is an out-of-the-box Docker image with zero size. This means the resulting image size equals the binary size.

## Binary Size

We can reduce binary size by using two approaches:

- Linking flags (`ldflags`);
- Compressing the binary file after compilation;

Flags `-s -w` remove debug information from the binary. In my case, the size was reduced from 15MB to 11MB using these flags.

`-extldflags '-static'` is used to statically link the binary even with CGO enabled. Dynamic linking makes our binary require external libraries (`.so` files in Linux).
External library dependencies can be reviewed by using the `ldd` command against the binary file.
In `scratch` and Alpine, we don't have these libraries installed. In this case, we have to use static linking — all libraries will be compiled into our program binary.

I also used the `upx` tool to compress the binary. The size was reduced from 11MB to 4.1MB.

The source code for this example can be found in the GitHub repository.
