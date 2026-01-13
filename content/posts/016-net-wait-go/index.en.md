+++
title = 'net-wait-go utility / library'
date = 2020-06-30T09:00:00+03:00
draft = false
tags = ['tcp','udp','docker']
featured_image = 'tube2.svg'
url = '/en/post/net-wait-go.html'

[quiz]
  [[quiz.questions]]
    question = "What is net-wait-go used for?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "To wait for ports to open (TCP, UDP)"
      correct = true
    [[quiz.questions.answers]]
      text = "To test network speed"
      correct = false
    [[quiz.questions.answers]]
      text = "To monitor network traffic"
      correct = false
  
  [[quiz.questions]]
    question = "Why is net-wait-go useful for Docker containers?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Minimal Docker images don't have bash or common utilities"
      correct = true
    [[quiz.questions.answers]]
      text = "Docker doesn't support networking"
      correct = false
    [[quiz.questions.answers]]
      text = "It's faster than bash commands"
      correct = false
  
  [[quiz.questions]]
    question = "What protocols does net-wait-go support?"
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

Both utility and Go package to wait for ports to open (TCP, UDP).

<!--more-->

## What is it?

Both utility and Go package to wait for ports to open (TCP, UDP).

## Why do we need it?

In dockerized applications, we usually deploy several containers along with the main program container.
We need to know if containers are started, so we can continue to execute our program or fail it after some deadline.

There are a lot of examples on the internet that advise us to use several bash commands like `nc`, `timeout`, `curl`. But what if we have a Go program with a minimal Docker image `FROM scratch` that does not have bash? We could use this package as a library in our program — just add a couple of lines of code to check if some ports are available.

This package can also be downloaded as a utility and used from the command line.

## Library usage

### Simple

```go
import "github.com/antelman107/net-wait-go/wait"

if !wait.New().Do([]string{"postgres:5432"}) {
    logger.Error("db is not available")
    return
}
```

### All optional settings definition

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

## Utility usage

```
net-wait-go

-addrs string
    address:port(,address:port,address:port,...)
-deadline uint
    deadline in milliseconds (default 10000)
-debug
    debug messages toggler
-delay uint
    break between requests in milliseconds (default 50)
-packet string
    UDP packet to be sent
-proto string
    tcp (default "tcp")
-wait uint
    delay of single request in milliseconds (default 100)
```

### 1 service check

```bash
net-wait-go -addrs ya.ru:443 -debug true
```

```
2020/06/30 18:07:38 ya.ru:443 is OK
```

Return code is 0

### 2 services check

```bash
net-wait-go -addrs ya.ru:443,yandex.ru:443 -debug true
```

```
2020/06/30 18:09:24 yandex.ru:443 is OK
2020/06/30 18:09:24 ya.ru:443 is OK
```

Return code is 0

### 2 services check (fail)

```bash
net-wait-go -addrs ya.ru:445,yandex.ru:445 -debug true
```

```
2020/06/30 18:09:24 yandex.ru:445 is FAILED
2020/06/30 18:09:24 ya.ru:445 is FAILED
...
```

Return code is 1

## UDP support

Since UDP as a protocol does not provide a connection between a server and clients, it is not supported in most popular utilities:

- wait-for-it ([issue](https://github.com/vishnubob/wait-for-it/issues/29))
- netcat (nc) has the following note in its manual page:

**CAVEATS**
> UDP port scans will always succeed (i.e. report the port as open)

net-wait-go provides UDP support, working in the following way:
- sends a meaningful packet to the server
- waits for a message back from the server (1 byte at least)

### UDP packet example

Counter Strike game server is accessible via UDP. Let's check a random Counter Strike server by sending an A2S_INFO packet ([documentation](https://developer.valvesoftware.com/wiki/Server_queries#A2S_INFO)):

```bash
net-wait-go -proto udp -addrs 46.174.53.245:27015,185.158.113.136:27015 -packet '/////1RTb3VyY2UgRW5naW5lIFF1ZXJ5AA==' -debug true
```

```
2020/07/12 15:13:25 udp 185.158.113.136:27015 is OK
2020/07/12 15:13:25 udp 46.174.53.245:27015 is OK
```

Return code is 0

The `-packet` value here is the base64-encoded A2S_INFO packet, which is documented [here](https://github.com/wriley/steamserverinfo/blob/master/steamserverinfo.go#L133).

## The sources

Available here — [github.com/antelman107/net-wait-go](https://github.com/antelman107/net-wait-go).