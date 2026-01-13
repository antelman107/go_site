+++
title = 'Dependency injection in GO'
date = 2020-04-21T09:00:00+03:00
draft = false
tags = ['dependency', 'injection', 'container', 'singleton','multiton']
featured_image = 'vlog.svg'
url = '/en/post/dependency-injection.html'

[quiz]
  [[quiz.questions]]
    question = "What is dependency injection?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "A pattern where a parent entity stores a dependency entity in its state"
      correct = true
    [[quiz.questions.answers]]
      text = "A function call pattern"
      correct = false
    [[quiz.questions.answers]]
      text = "A database connection pattern"
      correct = false
  
  [[quiz.questions]]
    question = "What pattern is used when you need a single instance of a dependency?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Singleton pattern"
      correct = true
    [[quiz.questions.answers]]
      text = "Factory pattern"
      correct = false
    [[quiz.questions.answers]]
      text = "Builder pattern"
      correct = false
  
  [[quiz.questions]]
    question = "What libraries are mentioned for dependency injection in Go?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "uber-go/dig"
      correct = true
    [[quiz.questions.answers]]
      text = "google/wire"
      correct = true
    [[quiz.questions.answers]]
      text = "sarulabs/di"
      correct = true
    [[quiz.questions.answers]]
      text = "golang/di"
      correct = false
  
  [[quiz.questions]]
    question = "What package does dig use to analyze function parameters and return types?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "reflect package"
      correct = true
    [[quiz.questions.answers]]
      text = "types package"
      correct = false
    [[quiz.questions.answers]]
      text = "inject package"
      correct = false
  
  [[quiz.questions]]
    question = "What is the multiton pattern?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Pool of singletons - multiple separate singletons for different instances of the same type"
      correct = true
    [[quiz.questions.answers]]
      text = "Multiple inheritance"
      correct = false
    [[quiz.questions.answers]]
      text = "Pattern for creating multiple objects"
      correct = false
  
  [[quiz.questions]]
    question = "What problems arise with a large number of dependencies in a program?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "A lot of initialization code"
      correct = true
    [[quiz.questions.answers]]
      text = "Service maintenance difficulty"
      correct = true
    [[quiz.questions.answers]]
      text = "Duplication of initialization code"
      correct = true
    [[quiz.questions.answers]]
      text = "Increased program execution speed"
      correct = false
  
  [[quiz.questions]]
    question = "What method is used in dig to get an entity from the container?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Invoke"
      correct = true
    [[quiz.questions.answers]]
      text = "Get"
      correct = false
    [[quiz.questions.answers]]
      text = "Retrieve"
      correct = false
  
  [[quiz.questions]]
    question = "What method is used in dig to add an entity initialization function to the container?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Provide"
      correct = true
    [[quiz.questions.answers]]
      text = "Add"
      correct = false
    [[quiz.questions.answers]]
      text = "Register"
      correct = false
  
  [[quiz.questions]]
    question = "How does lazy loading of entities work in dependency containers?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Entities are created only when Invoke is called"
      correct = true
    [[quiz.questions.answers]]
      text = "Entities are created immediately when added to container"
      correct = false
    [[quiz.questions.answers]]
      text = "Entities are created at program startup"
      correct = false
  
  [[quiz.questions]]
    question = "Which library uses GO code generation from YAML configuration?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "elliotchance/dingo"
      correct = true
    [[quiz.questions.answers]]
      text = "uber-go/dig"
      correct = false
    [[quiz.questions.answers]]
      text = "sarulabs/di"
      correct = false
  
  [[quiz.questions]]
    question = "Which library uses GO code generation from constructor function templates?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "google/wire"
      correct = true
    [[quiz.questions.answers]]
      text = "uber-go/dig"
      correct = false
    [[quiz.questions.answers]]
      text = "sarulabs/di"
      correct = false
  
  [[quiz.questions]]
    question = "What is the difference between sarulabs/di and uber-go/dig?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "di doesn't use reflect"
      correct = true
    [[quiz.questions.answers]]
      text = "di requires manual dependency retrieval from container"
      correct = true
    [[quiz.questions.answers]]
      text = "di supports destroy hook functions"
      correct = true
    [[quiz.questions.answers]]
      text = "di uses reflect for type analysis"
      correct = false
  
  [[quiz.questions]]
    question = "Why is changing the parent entity state important for defining dependency injection?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "It distinguishes dependency injection from external function call"
      correct = true
    [[quiz.questions.answers]]
      text = "It increases performance"
      correct = false
    [[quiz.questions.answers]]
      text = "It's required for container to work"
      correct = false
  
  [[quiz.questions]]
    question = "How are identical entities of the same type created in dig?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Name parameter is passed when calling Provide"
      correct = true
    [[quiz.questions.answers]]
      text = "Separate CreateNamed method is used"
      correct = false
    [[quiz.questions.answers]]
      text = "It's impossible in dig"
      correct = false
  
  [[quiz.questions]]
    question = "What advantages does using a dependency injection container provide?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "Simplification of initialization code"
      correct = true
    [[quiz.questions.answers]]
      text = "Centralized dependency management"
      correct = true
    [[quiz.questions.answers]]
      text = "Automatic dependency resolution"
      correct = true
    [[quiz.questions.answers]]
      text = "Increased program execution time"
      correct = false
+++

Let's talk about dependency injection pattern and dependency management in large programs.

<!--more-->

## Logger example

In any program there is `main.go` which manages to initialize and start some service(s).

We may say that every service in GO doesn't implement all its logic. Sometimes it requires other services and relies on them in particular parts of logic.

For example, logging is often delegated to some logger entity, for example zap:

```go
type Server struct {
	logger *zap.Logger
}

func NewServer(logger *zap.Logger) *Server {
	return &Server{logger: logger}
}

func (s *Server) Handle() {
	// do some work
	s.logger.Info("request processed")
}

logger := //... logger initializing
NewServer(logger).Run() // service with logger initializing
```

It is good to reuse code and rely on an entity that does its work well instead of writing your own code.

Currently our Server is not logging by itself, Server relies on logger. In other words, logger became the dependency of Server.
We saved logger as a property of Server. By doing it we injected logger as a dependency.

## Definition

Dependency injection — pattern of composing entities, as a result of which the first (parent) entity is saved to the state of second (dependency) entity. Parent entity can call dependency entity when it is necessary.

Parent state change is important to distinguish dependency injection and external function call.

Without state change the basic "hello world" program can be mistakenly recognized as dependency injection:

```go
func main() {
	fmt.Println("hello world")
}
```

There is no state in main function, so it is not dependency injection.

## Issues

Why do I discuss dependency injection and which issues can be behind this topic?

Issues can appear in programs that have a large amount of entities having a lot of links between them.
If there are a lot of linked entities, there is a lot of their initialization code. Such code with proper logic structure makes service difficult to support.

### Service with more dependencies

Let's imagine that we are developing a service that has to do the following:

- database interaction;
- perform external service calls;
- logging;
- loading and using config;

The service constructor should look like:

```go
func NewService(
	db *sql.DB,
	bankClient *client.Bank,
	cfg *config.Config,
	logger *zap.Logger,
)
```

Also, every Service dependency requires its own initialization, that can require other entities. For example:

```go
// Getting db connection
db, err := sql.Open("postgres", fmt.Sprintf(
	"host=%s port=%s user=%s dbname=%s sslmode=disable password=%s",
	configStruct.DbHost,
	configStruct.DbPort,
	configStruct.DbUser,
	configStruct.DbName,
	configStruct.DbPass)
)
if err != nil {
	log.Fatal(err)
}
```

To create bankClient we need cfg and logger.

Now let's imagine there is a second service needed to be implemented in the same program, that also requires db, cfg, logger as dependencies. Let's visualize the dependencies scheme:

![deps2.svg](deps2.svg)

There is a lot of code to initialize the first service, but also we need to initialize the second.

### Copy init code

We could just copy-paste db, cfg, logger init code for service2.

It will work, but copying code is a bad idea. More code to support, more mistake probability.

Let's check other options.

### Implement init code for each dep

For example we can implement db init function:

```go
func GetDB(cfg *config.Config) (*sql.DB, error) {
	db, err := sql.Open("postgres", fmt.Sprintf(
		"host=%s port=%s user=%s dbname=%s sslmode=disable password=%s",
		configStruct.DbHost, configStruct.DbPort, configStruct.DbUser, configStruct.DbName, configStruct.DbPass)
	)
	if err != nil {
		return nil, err
	}

	return db, nil
}
```

It looks good and there will be no duplicate db init code. But we still need to implement that code for each reusable dep.

We still haven't finished with GetDB - it will create a new connection for each call.

### Singleton

In case of db we need a single instance.

Let's implement it with singleton pattern:

```go
package db

var db *sql.DB

func GetDB(cfg *config.Config) (*sql.DB, error) {
	if db != nil {
		return db, nil
	}

	var err error
	db, err = sql.Open("postgres", fmt.Sprintf(
		"host=%s port=%s user=%s dbname=%s sslmode=disable password=%s",
		configStruct.DbHost, configStruct.DbPort, configStruct.DbUser, configStruct.DbName, configStruct.DbPass)
	)
	if err != nil {
		return nil, err
	}

	return db, nil
}
```

### Pool of singletons (multiton)

We can have connections to different database servers, it should be separate connections. But we still need each of them to be singleton. Let's implement pool of singletons — multiton.

On a small number of entities these patterns work well.
But if there are dozens of entity types even that simple code like singleton and multiton are hard to implement. In that case we could use some centralized logic that helps to build entities — dependency injector.

## Dependency injection container (injector)

Using a separate entity to build and store other entities (injector) is pretty common in many programming languages.
Container implements logic about creating each entity, storing and getting.

The focus in the program that uses container is moved from entity and its links to the container that helps to simplify the code.

![container.svg](container.svg)

Sometimes container work is so predictable that one can specify dependencies in declarative format — XML, YAML.

In Symfony (PHP) service container is one of the central parts in the framework - even Symfony core components are designed to work with container.
Symfony supports XML and YAML to declare dependencies.

In Spring (JAVA) dependency container can be configured by XML or annotations.

There are several libraries in GO implementing injector differently.

I used some of them and prepared a review about each of them below. There is source code about di libraries interaction in a separate github repository.

### uber-go/dig

dig allows us to configure container by passing anonymous functions and uses reflect package.

One should use Provide method to add entity init function into container.
The function should return the desired entity, or both entity and error.

Let's see how we can create logger that depends on config. (It is almost the original example from dig readme).

```go
c := dig.New()
err := c.Provide(func() (*Config, error) {
	// In real program there should be reading from the file, for example
	var cfg Config
	err := json.Unmarshal([]byte(`{"prefix": "[foo] "}`), &cfg)
	return &cfg, err
})
if err != nil {
	panic(err)
}

// Function to create logger by using config
err = c.Provide(func(cfg *Config) *log.Logger {
	return log.New(os.Stdout, cfg.Prefix, 0)
})
if err != nil {
	panic(err)
}
```

By using reflect package dig analyzes the types of returning value and the types of parameters.
Using that data the links between entities are resolved.

To get entity from container there is Invoke method:

```go
err = c.Invoke(func(l *log.Logger) {
	l.Print("You've been invoked")
})
if err != nil {
	panic(err)
}
```

On identical entity creation one should pass name parameter when calling Provide. Otherwise Provide will return error.

```go
// creating another logger
err = c.Provide(
	func(cfg *Config) *log.Logger {
		return log.New(os.Stdout, cfg.Prefix, 0)
	},
	dig.Name("logger2"), // passing name option
)
if err != nil {
	panic(err)
}
```

Unfortunately getting named entity is not so simple — there is no name parameter in Invoke function.
In the related github issue developers say that the issue is fixed, but not released yet.
Currently one should use structure with tagged fields to invoke named entities:

```go
c := dig.New()
c.Provide(username, dig.Name("username"))
c.Provide(password, dig.Name("password"))

err := c.Invoke(func(p struct {
	dig.In

	U string `name:"username"`
	P string `name:"password"`
}) {
	fmt.Println("user >>>", p.U)
	fmt.Println("pwd  >>>", p.P)
})
```

dig (and every injector library here) implements lazy loading of entities. Required entities are created only on Invoke call.

We could speak about that reflect is slow, but for container it doesn't matter, because typically container is used once on program start.

As a result: named entities issue should be documented in dig main readme. In other cases it works perfectly as injector.

### elliotchance/dingo

elliotchance/dingo works in a completely different way.
One should specify YAML config in order to generate container's GO-code. Let's continue with logger-config example. Our YAML should look like:

```yaml
services:
  Config:
    type: '*Config'
    error: return nil
    returns: NewConfig()
  Logger:
    type: '*log.Logger'
    import:
      - 'os'
    returns: log.New(os.Stdout, @{Config}.Prefix, 0)
```

To me YAML is not very comfortable to use here. You will see below, that some parts of YAML could be actually the parts of GO code. But to me the GO code is comfortable to be in *.go files — at least the IDE will check go syntax.

For every entity in YAML probably need to specify following:

- imports — the list of imported libraries;
- error — GO code, that should be called on error check;
- returns — the part of GO code which will init and return the entity;

With returns I couldn't decide: should I add big portion of GO code into the YAML, or should I create constructor function for each entity. Finally I moved all config construction logic to NewConfig function:

```go
func NewConfig() (*Config, error) {
	var cfg Config
	err := json.Unmarshal([]byte(`{"prefix": "[foo] "}`), &cfg)
	return &cfg, err
}
```

When the YAML is ready, one should install dingo binary and call it in the project directory — `go get -u github.com/elliotchance/dingo; dingo`.

Code generation works fast. To me it looks like that the most settings from YAML are just directly copied into generated *.go file. So, generated file could be invalid.
Generated code is placed in file dingo.go. Container is simple structure with fields for every entity with singleton logic:

```go
type Container struct {
	Config	*Config
	Logger	*log.Logger
}

func (container *Container) GetLogger() *log.Logger {
	if container.Logger == nil {
		service := log.New(os.Stdout, container.GetConfig().Prefix, 0)
		container.Logger = service
	}
	return container.Logger
}
```

As a result: elliotchance/dingo helps to generate simple typed container from YAML, but putting GO code to YAML makes me feel a little bit uncomfortable.

### sarulabs/di

sarulabs/di looks like dig, but doesn't use reflect. All deps in di must have unique names.

The main difference is that in dig we don't have to init dependencies of our entity, even from container — they just come as function parameters.
In di we have to pull dependencies from container:

```go
err = builder.Add(di.Def{
	Name: "logger",
	Build: func(ctn di.Container) (interface{}, error) {
		// Getting config from container to init logger
		var cfg *Config
		err = ctn.Fill("config", &cfg)
		if err != nil {
			return nil, err
		}

		// Init logger
		return log.New(os.Stdout, cfg.Prefix, 0), nil
	}
})
```

GO code that gets dependency from container is not big, but it will be copied between entities with similar dependencies.

But also sarulabs/di has a bonus — one can specify not only creation function, but also a container destroy hook function. di container destroy starts with DeleteWithSubContainers call and can be performed on program shutdown.

```go
Close: func(obj interface{}) error {
	if _, ok := obj.(*log.Logger); ok {
		fmt.Println("logger close") // this code is called on logger destroy
	}
	return nil
}
```

As I mentioned before di doesn't use reflect and also doesn't store any information about entities types, that's why we should use type assertion in Close function to get logger to original type.

There is also a bonus functionality sarulabs/dingo, from the same developer, that also provides strictly typed container and code generation.

As a result: di is great injector, but there is some code copying logic — to get dependency from container.

dig is better here.

### google/wire

With wire we have to put construction function template code for each entity. We should place `//+build wireinject` comment to the beginning of such template files.

Then we should run `go get github.com/google/wire/cmd/wire; wire` which generates `*_gen.go` files for each template file. Generated code will contain real constructor functions that are generated from templates.

For our logger-config example the template of logger constructor will look like:

```go
//+build wireinject

package main

import (
	"log"

	"github.com/google/wire"
)

// Template for generation
func GetLogger() (*log.Logger, error) {
	panic(wire.Build(NewLogger, NewConfig))
}
```

Generated code is put into `*_gen.go` and looks like:

```go
import (
	"log"
)

// Injectors from wire.go:

func GetLogger() (*log.Logger, error) {
	config, err := NewConfig()
	if err != nil {
		return nil, err
	}
	logger := NewLogger(config)
	return logger, nil
}
```

As in elliotchance/dingo there is code generation in wire. But I didn't manage to generate invalid GO code. In every invalid template situation wire outputs the errors and code is not generated.

There is one minus in wire — we have to implement constructor template by using wire package calls. And these calls are not so expressive as GO code. So I also move all constructor logic to the constructor functions to just call these constructor functions from templates.

## Comparison table

| Library | Dependencies format | GO code generation | Typing | Code reduction |
|---------|---------------------|-------------------|--------|----------------|
| uber-go/dig | GO code, anonymous functions with parameters | No | Strict, but also reflect is used | Maximum |
| elliotchance/dingo | YAML | Yes | Strict | Maximum, but there is mixing of GO code in YAML |
| sarulabs/di | GO code, declaration of Build functions with manual parameters getting | No, but sarulabs/dingo allows that | All deps are stored as interface{}. But sarulabs/dingo offers strictly typed container | Good, but we have to get deps from container |
| google/wire | GO code — templates of constructor functions | Yes | Strict | Maximum |
