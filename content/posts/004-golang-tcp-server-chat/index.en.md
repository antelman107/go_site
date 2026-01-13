+++
title = 'Coding a Simple TCP Chat in Go'
date = 2020-03-30T09:00:00+03:00
draft = false
tags = ['tcp', 'server', 'chat']
url = '/en/post/golang-tcp-server-chat.html'

[quiz]
  [[quiz.questions]]
    question = "Why do we need goroutines for handling multiple client connections in a TCP chat server?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Reading from connections is a blocking operation"
      correct = true
    [[quiz.questions.answers]]
      text = "Goroutines make the code faster"
      correct = false
    [[quiz.questions.answers]]
      text = "TCP requires goroutines"
      correct = false
  
  [[quiz.questions]]
    question = "What data structure is recommended for storing client connections in a concurrent TCP chat?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "sync.Map"
      correct = true
    [[quiz.questions.answers]]
      text = "Regular map with mutex"
      correct = false
    [[quiz.questions.answers]]
      text = "Slice"
      correct = false
  
  [[quiz.questions]]
    question = "What pattern is used when sending a message to all clients in the chat?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Fan out pattern"
      correct = true
    [[quiz.questions.answers]]
      text = "Round-robin pattern"
      correct = false
    [[quiz.questions.answers]]
      text = "Broadcast pattern"
      correct = false
  
  [[quiz.questions]]
    question = "Why is it important to work with a pointer to sync.Map rather than a value?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "sync.Map contains locks that shouldn't be copied"
      correct = true
    [[quiz.questions.answers]]
      text = "Pointers are faster"
      correct = false
    [[quiz.questions.answers]]
      text = "sync.Map only works with pointers"
      correct = false
  
  [[quiz.questions]]
    question = "What function is used to create a TCP server in Go?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "net.Listen"
      correct = true
    [[quiz.questions.answers]]
      text = "net.Server"
      correct = false
    [[quiz.questions.answers]]
      text = "net.TCPListen"
      correct = false
  
  [[quiz.questions]]
    question = "What method is used to accept new connections?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Accept"
      correct = true
    [[quiz.questions.answers]]
      text = "Connect"
      correct = false
    [[quiz.questions.answers]]
      text = "Receive"
      correct = false
  
  [[quiz.questions]]
    question = "What package is used to read strings from a connection?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "bufio"
      correct = true
    [[quiz.questions.answers]]
      text = "io"
      correct = false
    [[quiz.questions.answers]]
      text = "strings"
      correct = false
  
  [[quiz.questions]]
    question = "Why is UUID used to identify clients?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Guarantees unique keys with extremely low collision probability"
      correct = true
    [[quiz.questions.answers]]
      text = "UUID is faster than other identifiers"
      correct = false
    [[quiz.questions.answers]]
      text = "UUID is required for TCP"
      correct = false
  
  [[quiz.questions]]
    question = "What sync.Map method is used to iterate over all elements?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Range"
      correct = true
    [[quiz.questions.answers]]
      text = "Iterate"
      correct = false
    [[quiz.questions.answers]]
      text = "ForEach"
      correct = false
  
  [[quiz.questions]]
    question = "What happens when returning false from the function in Range?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Map iteration stops"
      correct = true
    [[quiz.questions.answers]]
      text = "An error occurs"
      correct = false
    [[quiz.questions.answers]]
      text = "Nothing happens"
      correct = false
  
  [[quiz.questions]]
    question = "How is client disconnection handled in the chat?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "On read error the function returns"
      correct = true
    [[quiz.questions.answers]]
      text = "Client is removed from connMap via Delete"
      correct = true
    [[quiz.questions.answers]]
      text = "Connection is closed via defer"
      correct = true
    [[quiz.questions.answers]]
      text = "Client remains in the map"
      correct = false
  
  [[quiz.questions]]
    question = "Why is defer used to close the connection?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "defer is called on any function exit, guaranteeing closure"
      correct = true
    [[quiz.questions.answers]]
      text = "defer is faster than normal closure"
      correct = false
    [[quiz.questions.answers]]
      text = "defer is required for TCP"
      correct = false
  
  [[quiz.questions]]
    question = "What pattern is used to send a message to all connected clients?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Iterate over all connections via Range and write to each"
      correct = true
    [[quiz.questions.answers]]
      text = "Send only to the first client"
      correct = false
    [[quiz.questions.answers]]
      text = "Use a channel for all clients"
      correct = false
  
  [[quiz.questions]]
    question = "What happens when there's an error writing to one connection during broadcast?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Error is logged but broadcast continues to other clients"
      correct = true
    [[quiz.questions.answers]]
      text = "Broadcast stops"
      correct = false
    [[quiz.questions.answers]]
      text = "Program exits"
      correct = false
  
  [[quiz.questions]]
    question = "Why is reading from connection done in an infinite loop?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "To handle multiple messages from the client"
      correct = true
    [[quiz.questions.answers]]
      text = "To increase performance"
      correct = false
    [[quiz.questions.answers]]
      text = "It's a TCP requirement"
      correct = false
+++

An example of a simple TCP chat in Go with logic explanation.

<!--more-->

Let's start the server:

```go
l, err := net.Listen("tcp", "localhost:9090")
if err != nil {
	return
}

defer l.Close()
```

The server listens on TCP port 9090 on localhost. It is normal practice to launch services bound to localhost.

Next, let's accept connections:

```go
conn, err := l.Accept()
if err != nil {
	return
}
```

No error handling yet. I'll focus on it later.

In a chat, we need to read messages sent by clients.

The number of clients is generally more than one, and reading is a blocking operation. Having a single program thread doesn't allow us to read from all connections. Let's fix this issue by adding a goroutine for each connected client:

```go
l, err := net.Listen("tcp", "localhost:9090")
if err != nil {
	return
}

defer l.Close()

for {
	conn, err := l.Accept()
	if err != nil {
		return
	}

	go handleUserConnection(conn)
}

func handleUserConnection(c net.Conn) {
	defer c.Close()

	for {
		userInput, err := bufio.NewReader(c).ReadString('\n')
		if err != nil {
			return
		}
	}
}
```

Here we return from the handling function on read error, which will happen when a client disconnects. After returning from the function, we also close the connection. The defer helps here — no matter how the function finishes — defer is called after return.

We don't need to handle connection closing errors, because errors here don't change the program flow.

Now we have a working server, but there is no logic besides reading data from clients. Let's add chat logic. The simplest chat will send any message to all clients. We may call this the "fan out" pattern and it is illustrated below:

To implement "fan out", we have to iterate over all client connections and write messages to them.

In order to do that, we need to store connections in some way so we can iterate them. I chose `sync.Map` here because it solves all concurrent access issues.

For our task, a slice would be enough. But in our concurrent program, we would have to add/remove data with that slice, and it would be impossible without `sync.Lock`. Since we're coding a simple TCP chat, let's just use a predefined type that solves concurrency issues.

```go
// Using sync.Map to not deal with concurrency slice/map issues
var connMap = &sync.Map{}
```

It is important to work with a pointer to `sync.Map`, not with a value.

`sync.Map` is a struct that contains `sync.Lock`. There is a known issue called "locks passed by value".

For our map, we need keys. I use UUID here because it guarantees that the keys will be different. (Extremely low probability that UUID generates two equal values).

When a client disconnects, we have to remove that client from the map. Let's pass the client's ID to the handling function:

```go
id := uuid.New().String()
connMap.Store(id, conn)
```

Finally, let's implement the message fan out pattern:

```go
for {
	userInput, err := bufio.NewReader(c).ReadString('\n')
	if err != nil {
		return
	}

	connMap.Range(func(key, value interface{}) bool {
		if conn, ok := value.(net.Conn); ok {
			conn.Write([]byte(userInput))
		}

		return true
	})
}
```

We can stop iterating the map if we return false from the range function.

Finally, I added error handling and use of the zap logger. The following code is the complete solution. Full source code is also available on GitHub.

```go
package main

import (
	"bufio"
	"net"
	"sync"

	"github.com/google/uuid"
	"go.uber.org/zap"
)

func main() {
	var loggerConfig = zap.NewProductionConfig()
	loggerConfig.Level.SetLevel(zap.DebugLevel)

	logger, err := loggerConfig.Build()
	if err != nil {
		panic(err)
	}

	l, err := net.Listen("tcp", "localhost:9090")
	if err != nil {
		return
	}

	defer l.Close()

	// Using sync.Map to not deal with concurrency slice/map issues
	var connMap = &sync.Map{}

	for {
		conn, err := l.Accept()
		if err != nil {
			logger.Error("error accepting connection", zap.Error(err))
			return
		}

		id := uuid.New().String()
		connMap.Store(id, conn)

		go handleUserConnection(id, conn, connMap, logger)
	}
}

func handleUserConnection(id string, c net.Conn, connMap *sync.Map, logger *zap.Logger) {
	defer func() {
		c.Close()
		connMap.Delete(id)
	}()

	for {
		userInput, err := bufio.NewReader(c).ReadString('\n')
		if err != nil {
			logger.Error("error reading from client", zap.Error(err))
			return
		}

		connMap.Range(func(key, value interface{}) bool {
			if conn, ok := value.(net.Conn); ok {
				if _, err := conn.Write([]byte(userInput)); err != nil {
					logger.Error("error on writing to connection", zap.Error(err))
				}
			}

			return true
		})
	}
}
```
