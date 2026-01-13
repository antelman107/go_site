+++
title = 'Внедрение зависимостей в GO'
date = 2020-04-21T09:00:00+03:00
draft = false
tags = ['dependency', 'injection', 'container', 'singleton','multiton']
featured_image = 'vlog.svg'
url = '/ru/post/dependency-injection.html'

[quiz]
  [[quiz.questions]]
    question = "Что такое внедрение зависимостей?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Паттерн, при котором родительская сущность сохраняет зависимую сущность в своем состоянии"
      correct = true
    [[quiz.questions.answers]]
      text = "Паттерн вызова функции"
      correct = false
    [[quiz.questions.answers]]
      text = "Паттерн подключения к базе данных"
      correct = false
  
  [[quiz.questions]]
    question = "Какой паттерн используется, когда нужен единственный экземпляр зависимости?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Паттерн Singleton"
      correct = true
    [[quiz.questions.answers]]
      text = "Паттерн Factory"
      correct = false
    [[quiz.questions.answers]]
      text = "Паттерн Builder"
      correct = false
  
  [[quiz.questions]]
    question = "Какие библиотеки упоминаются для внедрения зависимостей в Go?"
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
    question = "Какой пакет использует dig для анализа параметров функций и типов возвращаемых значений?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Пакет reflect"
      correct = true
    [[quiz.questions.answers]]
      text = "Пакет types"
      correct = false
    [[quiz.questions.answers]]
      text = "Пакет inject"
      correct = false
  
  [[quiz.questions]]
    question = "Что такое паттерн multiton?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Пул синглтонов - несколько отдельных синглтонов для разных экземпляров одного типа"
      correct = true
    [[quiz.questions.answers]]
      text = "Множественное наследование"
      correct = false
    [[quiz.questions.answers]]
      text = "Паттерн для создания множества объектов"
      correct = false
  
  [[quiz.questions]]
    question = "Какие проблемы возникают при большом количестве зависимостей в программе?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "Много кода инициализации"
      correct = true
    [[quiz.questions.answers]]
      text = "Сложность поддержки сервиса"
      correct = true
    [[quiz.questions.answers]]
      text = "Дублирование кода инициализации"
      correct = true
    [[quiz.questions.answers]]
      text = "Увеличение скорости выполнения программы"
      correct = false
  
  [[quiz.questions]]
    question = "Какой метод используется в dig для получения сущности из контейнера?"
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
    question = "Какой метод используется в dig для добавления функции инициализации сущности в контейнер?"
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
    question = "Как работает ленивая загрузка сущностей в контейнерах зависимостей?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Сущности создаются только при вызове Invoke"
      correct = true
    [[quiz.questions.answers]]
      text = "Сущности создаются сразу при добавлении в контейнер"
      correct = false
    [[quiz.questions.answers]]
      text = "Сущности создаются при старте программы"
      correct = false
  
  [[quiz.questions]]
    question = "Какая библиотека использует генерацию GO кода из YAML конфигурации?"
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
    question = "Какая библиотека использует генерацию GO кода из шаблонов функций-конструкторов?"
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
    question = "Какое отличие sarulabs/di от uber-go/dig?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "di не использует reflect"
      correct = true
    [[quiz.questions.answers]]
      text = "di требует ручного получения зависимостей из контейнера"
      correct = true
    [[quiz.questions.answers]]
      text = "di поддерживает функции-хуки уничтожения"
      correct = true
    [[quiz.questions.answers]]
      text = "di использует reflect для анализа типов"
      correct = false
  
  [[quiz.questions]]
    question = "Почему изменение состояния родительской сущности важно для определения внедрения зависимостей?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Это отличает внедрение зависимостей от внешнего вызова функции"
      correct = true
    [[quiz.questions.answers]]
      text = "Это увеличивает производительность"
      correct = false
    [[quiz.questions.answers]]
      text = "Это требуется для работы контейнера"
      correct = false
  
  [[quiz.questions]]
    question = "Как в dig создаются идентичные сущности одного типа?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Передаётся параметр name при вызове Provide"
      correct = true
    [[quiz.questions.answers]]
      text = "Используется отдельный метод CreateNamed"
      correct = false
    [[quiz.questions.answers]]
      text = "Это невозможно в dig"
      correct = false
  
  [[quiz.questions]]
    question = "Какие преимущества даёт использование контейнера внедрения зависимостей?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "Упрощение кода инициализации"
      correct = true
    [[quiz.questions.answers]]
      text = "Централизованное управление зависимостями"
      correct = true
    [[quiz.questions.answers]]
      text = "Автоматическое разрешение зависимостей"
      correct = true
    [[quiz.questions.answers]]
      text = "Увеличение времени выполнения программы"
      correct = false
+++

Давайте поговорим о паттерне внедрения зависимостей и управлении зависимостями в больших программах.

<!--more-->

## Пример с логгером

В любой программе есть `main.go`, который управляет инициализацией и запуском сервисов.

Можно сказать, что каждый сервис в GO не реализует всю свою логику. Иногда ему требуются другие сервисы, и он полагается на них в определенных частях логики.

Например, логирование часто делегируется некоторой сущности-логгеру, например zap:

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

Хорошо переиспользовать код и полагаться на сущность, которая хорошо выполняет свою работу, вместо написания собственного кода.

В настоящее время наш Server не логирует сам по себе, Server полагается на logger. Другими словами, logger стал зависимостью Server.
Мы сохранили logger как свойство Server. Делая это, мы внедрили logger как зависимость.

## Определение

Внедрение зависимостей — паттерн композиции сущностей, в результате которого первая (родительская) сущность сохраняется в состоянии второй (зависимой) сущности. Родительская сущность может вызывать зависимую сущность, когда это необходимо.

Изменение состояния родителя важно для различения внедрения зависимостей и внешнего вызова функции.

Без изменения состояния базовую программу "hello world" можно ошибочно принять за внедрение зависимостей:

```go
func main() {
	fmt.Println("hello world")
}
```

В функции main нет состояния, поэтому это не внедрение зависимостей.

## Проблемы

Почему я обсуждаю внедрение зависимостей и какие проблемы могут быть за этой темой?

Проблемы могут возникать в программах, которые имеют большое количество сущностей со множеством связей между ними.
Если есть много связанных сущностей, то есть много кода их инициализации. Такой код с правильной структурой логики затрудняет поддержку сервиса.

### Сервис с большим количеством зависимостей

Давайте представим, что мы разрабатываем сервис, который должен выполнять следующее:

- взаимодействие с базой данных;
- выполнение внешних вызовов сервисов;
- логирование;
- загрузка и использование конфигурации;

Конструктор сервиса должен выглядеть так:

```go
func NewService(
	db *sql.DB,
	bankClient *client.Bank,
	cfg *config.Config,
	logger *zap.Logger,
)
```

Здесь `*sql.DB` используется как тип параметра.

Также каждая зависимость Service требует своей собственной инициализации, которая может требовать других сущностей. Например:

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

Для создания bankClient нам нужны cfg и logger.

Теперь давайте представим, что есть второй сервис, который нужно реализовать в той же программе, который также требует db, cfg, logger в качестве зависимостей. Давайте визуализируем схему зависимостей:

![deps2.svg](deps2.svg)

Есть много кода для инициализации первого сервиса, но также нам нужно инициализировать второй.

### Копирование кода инициализации

Мы могли бы просто скопировать код инициализации db, cfg, logger для service2.

Это будет работать, но копирование кода — плохая идея. Больше кода для поддержки, больше вероятность ошибок.

Давайте проверим другие варианты.

### Реализация кода инициализации для каждой зависимости

Например, мы можем реализовать функцию инициализации db:

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

Это выглядит хорошо и не будет дублирующегося кода инициализации db. Но нам все еще нужно реализовать этот код для каждой переиспользуемой зависимости.

Мы все еще не закончили с GetDB - она будет создавать новое соединение при каждом вызове.

### Singleton

В случае с db нам нужен единственный экземпляр.

Давайте реализуем это с помощью паттерна singleton:

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

### Пул синглтонов (multiton)

У нас могут быть соединения с разными серверами баз данных, это должны быть отдельные соединения. Но нам все еще нужно, чтобы каждое из них было синглтоном. Давайте реализуем пул синглтонов — multiton.

При небольшом количестве сущностей эти паттерны работают хорошо.
Но если есть десятки типов сущностей, даже такой простой код, как singleton и multiton, трудно реализовать. В этом случае мы могли бы использовать некоторую централизованную логику, которая помогает строить сущности — инжектор зависимостей.

## Контейнер внедрения зависимостей (инжектор)

Использование отдельной сущности для построения и хранения других сущностей (инжектор) довольно распространено во многих языках программирования.
Контейнер реализует логику создания каждой сущности, хранения и получения.

Фокус в программе, использующей контейнер, перемещается с сущности и ее связей на контейнер, который помогает упростить код.

![container.svg](container.svg)

Иногда работа контейнера настолько предсказуема, что можно указать зависимости в декларативном формате — XML, YAML.

В Symfony (PHP) контейнер сервисов является одной из центральных частей фреймворка - даже основные компоненты Symfony разработаны для работы с контейнером.
Symfony поддерживает XML и YAML для объявления зависимостей.

В Spring (JAVA) контейнер зависимостей может быть настроен через XML или аннотации.

В GO есть несколько библиотек, реализующих инжектор по-разному.

Я использовал некоторые из них и подготовил обзор каждой из них ниже. Есть исходный код о взаимодействии библиотек di в отдельном репозитории github.

### uber-go/dig

dig позволяет нам настроить контейнер, передавая анонимные функции и используя пакет `reflect`.

Следует использовать метод Provide для добавления функции инициализации сущности в контейнер.
Функция должна возвращать желаемую сущность или и сущность, и ошибку.

Давайте посмотрим, как мы можем создать logger, который зависит от config. (Это почти оригинальный пример из readme dig).

```go
c := dig.New()
err := c.Provide(func() (*Config, error) {
	// В реальной программе здесь должно быть чтение из файла, например
	var cfg Config
	err := json.Unmarshal([]byte(`{"prefix": "[foo] "}`), &cfg)
	return &cfg, err
})
if err != nil {
	panic(err)
}

// Функция для создания logger с использованием config
err = c.Provide(func(cfg *Config) *log.Logger {
	return log.New(os.Stdout, cfg.Prefix, 0)
})
if err != nil {
	panic(err)
}
```

Используя пакет `reflect`, dig анализирует типы возвращаемого значения и типы параметров.
Используя эти данные, связи между сущностями разрешаются.

Для получения сущности из контейнера есть метод Invoke:

```go
err = c.Invoke(func(l *log.Logger) {
	l.Print("You've been invoked")
})
if err != nil {
	panic(err)
}
```

При создании идентичной сущности следует передать параметр name при вызове Provide. В противном случае Provide вернет ошибку.

```go
// создаем еще один логгер
err = c.Provide(
	func(cfg *Config) *log.Logger {
		return log.New(os.Stdout, cfg.Prefix, 0)
	},
	dig.Name("logger2"), // передаем опцию имени
)
if err != nil {
	panic(err)
}
```

К сожалению, получение именованной сущности не так просто — в функции Invoke нет параметра name.
В связанном issue на github разработчики говорят, что проблема исправлена, но еще не выпущена.
В настоящее время следует использовать структуру с помеченными полями для вызова именованных сущностей:

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

dig (и каждая библиотека инжектора здесь) реализует ленивую загрузку сущностей. Требуемые сущности создаются только при вызове Invoke.

Мы могли бы говорить о том, что `reflect` медленный, но для контейнера это не имеет значения, потому что обычно контейнер используется один раз при запуске программы.

В результате: проблема с именованными сущностями должна быть задокументирована в основном readme dig. В остальных случаях он работает отлично как инжектор.

### elliotchance/dingo

elliotchance/dingo работает совершенно по-другому.
Следует указать YAML конфигурацию для генерации GO-кода контейнера. Давайте продолжим с примером logger-config. Наш YAML должен выглядеть так:

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

Для меня YAML не очень удобен в использовании здесь. Вы увидите ниже, что некоторые части YAML могут быть фактически частями GO кода. Но для меня GO код удобнее быть в *.go файлах — по крайней мере IDE проверит синтаксис go.

Для каждой сущности в YAML, вероятно, нужно указать следующее:

- imports — список импортированных библиотек;
- error — GO код, который должен быть вызван при проверке ошибки;
- returns — часть GO кода, которая инициализирует и вернет сущность;

С returns я не мог решить: должен ли я добавить большую часть GO кода в YAML, или должен ли я создать функцию-конструктор для каждой сущности. В итоге я переместил всю логику построения конфигурации в функцию NewConfig:

```go
func NewConfig() (*Config, error) {
	var cfg Config
	err := json.Unmarshal([]byte(`{"prefix": "[foo] "}`), &cfg)
	return &cfg, err
}
```

Когда YAML готов, следует установить бинарный файл dingo и вызвать его в директории проекта — `go get -u github.com/elliotchance/dingo; dingo`.

Генерация кода работает быстро. Мне кажется, что большинство настроек из YAML просто напрямую копируются в сгенерированный *.go файл. Поэтому сгенерированный файл может быть невалидным.
Сгенерированный код помещается в файл dingo.go. Контейнер — это простая структура с полями для каждой сущности с логикой singleton:

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

В результате: elliotchance/dingo помогает генерировать простой типизированный контейнер из YAML, но помещение GO кода в YAML заставляет меня чувствовать себя немного некомфортно.

### sarulabs/di

sarulabs/di похож на dig, но не использует `reflect`. Все зависимости в di должны иметь уникальные имена.

Основное отличие в том, что в dig нам не нужно инициализировать зависимости нашей сущности, даже из контейнера — они просто приходят как параметры функции.
В di нам нужно извлекать зависимости из контейнера:

```go
err = builder.Add(di.Def{
	Name: "logger",
	Build: func(ctn di.Container) (interface{}, error) {
		// Получаем config из контейнера для инициализации logger
		var cfg *Config
		err = ctn.Fill("config", &cfg)
		if err != nil {
			return nil, err
		}

		// Инициализируем logger
		return log.New(os.Stdout, cfg.Prefix, 0), nil
	}
})
```

GO код, который получает зависимость из контейнера, не большой, но он будет копироваться между сущностями с похожими зависимостями.

Но также sarulabs/di имеет бонус — можно указать не только функцию создания, но и функцию-хук уничтожения контейнера. Уничтожение контейнера di начинается с вызова DeleteWithSubContainers и может быть выполнено при завершении программы.

```go
Close: func(obj interface{}) error {
	if _, ok := obj.(*log.Logger); ok {
		fmt.Println("logger close") // этот код вызывается при уничтожении logger
	}
	return nil
}
```

Как я упоминал ранее, di не использует `reflect` и также не хранит никакой информации о типах сущностей, поэтому мы должны использовать приведение типов в функции Close, чтобы получить logger к исходному типу.

Также есть бонусная функциональность sarulabs/dingo, от того же разработчика, которая также предоставляет строго типизированный контейнер и генерацию кода.

В результате: di — отличный инжектор, но есть некоторая логика копирования кода — для получения зависимости из контейнера.

dig здесь лучше.

### google/wire

С wire нам нужно поместить код шаблона функции построения для каждой сущности. Мы должны поместить комментарий `//+build wireinject` в начало таких файлов-шаблонов.

Затем мы должны запустить `go get github.com/google/wire/cmd/wire; wire`, который генерирует файлы `*_gen.go` для каждого файла-шаблона. Сгенерированный код будет содержать реальные функции-конструкторы, которые генерируются из шаблонов.

Для нашего примера logger-config шаблон конструктора logger будет выглядеть так:

```go
//+build wireinject

package main

import (
	"log"

	"github.com/google/wire"
)

// Шаблон для генерации
func GetLogger() (*log.Logger, error) {
	panic(wire.Build(NewLogger, NewConfig))
}
```

Сгенерированный код помещается в `*_gen.go` и выглядит так:

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

Как в elliotchance/dingo, в wire есть генерация кода. Но мне не удалось сгенерировать невалидный GO код. В каждой ситуации с невалидным шаблоном wire выводит ошибки и код не генерируется.

Есть один минус в wire — нам нужно реализовать шаблон конструктора, используя вызовы пакета wire. И эти вызовы не так выразительны, как GO код. Поэтому я также переместил всю логику конструктора в функции-конструкторы, чтобы просто вызывать эти функции-конструкторы из шаблонов.

## Таблица сравнения

| Библиотека | Формат зависимостей | Генерация GO кода | Типизация | Сокращение кода |
|------------|---------------------|-------------------|-----------|-----------------|
| uber-go/dig | GO код, анонимные функции с параметрами | Нет | Строгая, но также используется reflect | Максимальное |
| elliotchance/dingo | YAML | Да | Строгая | Максимальное, но есть смешивание GO кода в YAML |
| sarulabs/di | GO код, объявление функций Build с ручным получением параметров | Нет, но sarulabs/dingo позволяет это | Все зависимости хранятся как interface{}. Но sarulabs/dingo предлагает строго типизированный контейнер | Хорошее, но нам нужно получать зависимости из контейнера |
| google/wire | GO код — шаблоны функций-конструкторов | Да | Строгая | Максимальное |

