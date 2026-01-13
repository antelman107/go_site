+++
title = 'Принципы работы типа map в GO'
date = 2020-04-02T09:00:00+03:00
draft = false
tags = ['map', 'sources']
featured_image = 'map.svg'
url = '/ru/post/map-principles-golang.html'

[quiz]
  [[quiz.questions]]
    question = "Какие типы могут использоваться в качестве ключей map в Go?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "Сравнимые типы, такие как простые скалярные типы"
      correct = true
    [[quiz.questions.answers]]
      text = "Каналы"
      correct = true
    [[quiz.questions.answers]]
      text = "Массивы"
      correct = true
    [[quiz.questions.answers]]
      text = "Срезы"
      correct = false
    [[quiz.questions.answers]]
      text = "Карты"
      correct = false
  
  [[quiz.questions]]
    question = "Каковы требования к хеш-функции, используемой в картах Go?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "Детерминированная"
      correct = true
    [[quiz.questions.answers]]
      text = "Равномерное распределение"
      correct = true
    [[quiz.questions.answers]]
      text = "Быстрое выполнение"
      correct = true
    [[quiz.questions.answers]]
      text = "Криптографически безопасная"
      correct = false
  
  [[quiz.questions]]
    question = "Сколько бакетов добавляется при вставке первого ключа в пустую карту?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "8 бакетов"
      correct = true
    [[quiz.questions.answers]]
      text = "1 бакет"
      correct = false
    [[quiz.questions.answers]]
      text = "16 бакетов"
      correct = false
  
  [[quiz.questions]]
    question = "Почему Go не позволяет взять адрес значения карты?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Во время роста карты адреса данных могут изменяться"
      correct = true
    [[quiz.questions.answers]]
      text = "Значения карты неизменяемы"
      correct = false
    [[quiz.questions.answers]]
      text = "Это ограничение дизайна языка"
      correct = false
+++

Программный интерфейс map в Go описан в блоге Go. Нам просто нужно вспомнить, что map — это хранилище ключ-значение, и оно должно извлекать значения по ключу как можно быстрее.

<!--more-->

Любой сравниваемый тип может быть ключом — все простые скалярные типы, каналы и массивы.
Несравниваемые типы включают срезы, карты и функции.

Ключи и значения map должны храниться в памяти, выделенной для map, последовательно. Для работы с адресами памяти мы должны использовать хеш-функцию.

У нас есть следующие требования к хеш-функции:

- **Детерминированная** — должна возвращать одно и то же значение для одного и того же ключа
- **Равномерная** — значения должны быть каким-то образом равномерно распределены по всем бакетам
- **Быстрая** — должна выполняться быстро, чтобы использоваться многократно

## Реализация

Теперь к реализации. Упрощённая структура map представлена ниже (из исходников):

```go
// Заголовок для Go map.
type hmap struct {
	count      int // # живых ячеек == размер map. Должно быть первым (используется встроенной функцией len())
	B          uint8  // log_2 от количества бакетов (может содержать до loadFactor * 2^B элементов)
	buckets    unsafe.Pointer // массив из 2^B бакетов. может быть nil, если count==0.
	oldbuckets unsafe.Pointer // предыдущий массив бакетов вдвое меньшего размера, не-nil только при росте
}
```

Адреса данных хранятся разделёнными на части — в массиве buckets.

`unsafe.Pointer` — может быть указателем любого типа переменной — решение проблемы дженериков (разработчики Go используют это для создания ключей и значений любого типа).

Buckets равен nil, если в map нет данных. При первом ключе добавляется 8 бакетов.

## Получение данных из Map

Нам нужно найти адрес памяти ключа и значения.

Сначала, чтобы найти бакет. Он выбирается сравнением первых 8 бит хеша ключа с соответствующими данными, хранящимися в структуре бакета.

Затем, чтобы найти значение ключа по его адресу:

```go
k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
if t.indirectkey() {
	k = *((*unsafe.Pointer)(k))
}
```

Затем, чтобы найти значение тем же способом:

```go
if t.key.equal(key, k) {
	e := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
	if t.indirectelem() {
		e = *((*unsafe.Pointer)(e))
	}
	return e
}
```

## Рост Map

Мы работаем со свойством `oldbuckets` структуры map во время миграции данных.

Миграция начинается, когда в бакетах слишком много данных. Мы сохраняем текущее значение buckets в oldbuckets, затем создаём удвоенное количество бакетов в свойстве buckets. Данные map копируются из oldbuckets в buckets.

Количество бакетов всегда является степенью двойки.
Вот почему есть свойство B — это степень двойки, которая показывает текущее количество бакетов.

Во время миграции map доступна. Вот почему в исходном коде много работы с обоими buckets и oldbuckets в одних и тех же функциях. После миграции oldbuckets устанавливается в nil.

Во время миграции адрес данных может измениться, поэтому Go не позволяет получить указатель на значение map:

```go
mymap := map[string]string{"1": "1"}
fmt.Println(&mymap["1"])

// нельзя взять адрес mymap["1"]
```

Отличная лекция GopherCon 2016 по теме:
