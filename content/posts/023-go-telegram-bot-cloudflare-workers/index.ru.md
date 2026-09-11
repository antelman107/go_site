+++
title = 'С AWS на Cloudflare Workers: сравнение бесплатных тиров и холодного старта'
date = 2026-09-11T23:00:00+03:00
draft = false
tags = ['telegram', 'aws', 'lambda', 's3', 'cloudflare', 'workers', 'r2', 'go']
url = '/ru/post/telegram-bot-cloudflare-workers.html'
featured_image = 'featured.svg'

[quiz]
  [[quiz.questions]]
    question = "У какого serverless-рантайма обычно самый короткий cold start?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Cloudflare Workers (V8 isolate)"
      correct = true
    [[quiz.questions.answers]]
      text = "AWS Lambda на custom runtime provided.al2023"
      correct = false
    [[quiz.questions.answers]]
      text = "Azure Functions Consumption"
      correct = false

  [[quiz.questions]]
    question = "Какой always-free лимит у Cloudflare R2 на хранение?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "10 ГБ в месяц, без срока «первый год»"
      correct = true
    [[quiz.questions.answers]]
      text = "5 ГБ только первые 12 месяцев"
      correct = false
    [[quiz.questions.answers]]
      text = "Безлимитное хранение"
      correct = false

  [[quiz.questions]]
    question = "Что из этого верно про бесплатные тиры объектного хранилища?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "У AWS S3 щедрый год, но 2 000 PUT в месяц — узкое место"
      correct = true
    [[quiz.questions.answers]]
      text = "У R2 1 млн Class A и 10 млн Class B операций каждый месяц"
      correct = true
    [[quiz.questions.answers]]
      text = "У Azure Blob storage входит в free grant Azure Functions"
      correct = false
+++

В [прошлой статье](/ru/post/telegram-bots-zero-cost-aws.html) я рассказывал, как уложил Telegram-бота-организатора волейбольных игр в AWS Free Tier: Lambda + S3 + EventBridge и счёт около нуля долларов. Схема жила месяцами. Потом я перенёс того же бота на **Cloudflare Workers** и **R2** — тот же Go, те же JSON-файлы стейта, тот же Gemini.

<!--more-->

Зачем трогать то, что уже бесплатно? Из-за **холодного старта**. Lambda на custom runtime `provided.al2023` после простоя просыпается сотни миллисекунд, иногда дольше секунды. Для Telegram webhook это ощущается: первый апдейт после тишины приходит заметно медленнее, чем второй. У Workers isolate поднимается за **десятки миллисекунд**. На реальном деплое этого бота Wrangler показывал Worker Startup Time **16–37 ms**.

Ниже — практическое сравнение бесплатных тиров AWS, GCP, Azure и Cloudflare по двум вещам, которые нужны такому боту: **compute в стиле Lambda** и **объектное хранилище в стиле S3**. Цифры — ориентир на сентябрь 2026, перед продом всегда сверяю официальные страницы.

![Сравнение холодного старта](featured.svg)

## Что осталось тем же ботом

Логика не менялась: webhook из Telegram, Gemini как роутер, JSON-стейт игр и памяти чата, джобы до/после игры и Gmail-полл оплат.

На AWS это были отдельные Lambda и EventBridge one-shot (`at(...)`) на каждую игру. На Cloudflare:

1. Один Worker на Go, собранный в Wasm через [syumai/workers-go](https://github.com/syumai/workers-go).
2. Тот же набор ключей в **R2**, что раньше в S3 (`state/game_state.json` и соседи).
3. Вместо EventBridge — JSON-календарь jobs в R2 и **cron раз в 10 минут**, который запускает due-задачи.

One-shot «разбуди меня 12 сентября в 15:30» у Workers нет. Зато нет и отдельного зоопарка функций: webhook и cron — один бинарник, разные входы.

## Compute: Lambda-подобные сервисы

| | AWS Lambda | GCP Cloud Run functions | Azure Functions (Consumption) | Cloudflare Workers |
|---|---|---|---|---|
| Бесплатные вызовы | **1 млн / мес**, always free | **2 млн / мес**, always free | **1 млн / мес** (grant на Consumption) | Free: **100 000 / день** (~3 млн / мес). Paid: 10 млн / мес в $5 |
| Compute | 400 000 GB-s / мес | 400 000 GB-s + 200 000 GHz-s | 400 000 GB-s | Free: **10 ms CPU** на вызов. Paid: 30 млн CPU-ms / мес |
| Что считается временем | wall-clock × память | wall-clock × память/CPU | wall-clock × память | **CPU**, не ожидание HTTP |
| Cold start | сотни ms (Go custom runtime) | сотни ms … секунды | часто **секунды** | **десятки ms** (isolate) |
| Go | native `provided.al2023` | native | custom handler | **Wasm** (`GOOS=js`) |

Источники: [AWS Lambda pricing](https://aws.amazon.com/lambda/pricing/), [Google Cloud Free Tier](https://docs.cloud.google.com/free/docs/free-cloud-features), [Azure Functions pricing](https://azure.microsoft.com/pricing/details/functions/), [Workers pricing](https://developers.cloudflare.com/workers/platform/pricing/), [Workers limits](https://developers.cloudflare.com/workers/platform/limits/).

Три больших облака устроены похоже: миллионы запросов и сотни тысяч GB-секунд. Этого хватает маленькому боту. Разница не в «влезу / не влезу», а в **как просыпается рантайм** и не кончится ли бесплатность через год.

### Холодный старт

Классическая Lambda поднимает микро-ВМ, тянет runtime и ваш zip/image. Для Go на `provided.al2023` это обычно **сотни миллисекунд**. GCP Gen2 ближе к Cloud Run: лучше Lambda первого поколения, но cold start всё ещё «контейнерный». Azure Consumption — самый неприятный из четвёрки: после простоя первый запрос легко уходит в секунды.

Workers работают иначе. Это не контейнер, а **V8 isolate**. Cloudflare держит лимит старта в 1 секунду, но реальный старт моего Wasm-воркера — **16–37 ms**. Повторные запросы в тёплый isolate ещё короче. Для чат-бота это важнее таблички «1 млн запросов»: люди замечают паузу, а не GB-секунды.

Отдельный бонус биллинга: пока Worker ждёт ответ Gemini, wall-clock идёт, а **CPU почти нет**. На Lambda вы платите за всё время, пока функция жива. На Workers Paid в лимит входят CPU-ms, а не секунды ожидания HTTP.

### Честно про Free plan Workers

Бесплатный Workers щедрый по **количеству запросов** (100k в день — это больше, чем always-free Lambda). Но для этого бота Free plan не подходит:

- лимит размера воркера на Free — **3 MB**, у меня Wasm gzip ~8 MB;
- **10 ms CPU** на вызов не хватит на Go Wasm + Gemini.

Поэтому прод живёт на **Workers Paid за $5 / мес**: 10 млн запросов и нормальный CPU. Это всё ещё дешевле VPS и предсказуемее, чем «бесплатно, пока не кончится 12-месячный тир». Сам **R2 free tier при этом остаётся**.

## Object storage: S3 и аналоги

| | AWS S3 | GCP Cloud Storage | Azure Blob | Cloudflare R2 |
|---|---|---|---|---|
| Бесплатное хранение | 5 ГБ **первые 12 месяцев** | 5 GB-month always free, только `us-central1` / `us-east1` / `us-west1` | 5 ГБ LRS **первые 12 месяцев** (Free Account) | **10 ГБ / мес always free** |
| Чтения | 20 000 GET / мес (год 1) | 50 000 Class B | ~20 000 read (год 1) | **10 млн Class B / мес** |
| Записи | **2 000 PUT** / мес (год 1) | 5 000 Class A | ~10 000 write (год 1) | **1 млн Class A / мес** |
| Egress | лимитируется, потом $/GB | 100 ГБ (ограниченные регионы) | отдельно | **бесплатно** |
| После льготы | ~$0.023/ГБ, PUT $0.005/1k | обычные тарифы GCS | storage account **не входит** в Functions grant | сверх лимита $0.015/ГБ, без egress |

Источники: [AWS S3 pricing](https://aws.amazon.com/s3/pricing/), [GCP Free Tier](https://docs.cloud.google.com/free/docs/free-cloud-features), [Azure Functions pricing](https://azure.microsoft.com/pricing/details/functions/) (storage billed separately), [R2 pricing](https://developers.cloudflare.com/r2/pricing/).

В прошлой статье узким местом S3 были **2 000 PUT в первый год**. Ленивая загрузка и запись только в конце запроса как раз про это. На R2 лимит записи — **миллион Class A в месяц**, always free. Для бота, который пишет несколько JSON-файлов на апдейт, это другой порядок.

Второй сюрприз Azure: grant Functions **не покрывает** storage account, который создаётся вместе с Function App. «Бесплатные лямбды» легко начинают капать из-за Blob.

GCP Cloud Storage always free есть, но только в трёх US-регионах и со скромными 5 000 Class A. Для JSON-стейта хватит, для привычки «PUT на каждое сообщение» — уже впритык.

R2 ещё и без платы за отдачу в интернет. Для бота это почти неважно (стейт мелкий), но как бесплатный S3-аналог линейка выглядит честнее: **10 ГБ + много операций + без «первый год»**.

## Что я вынес из переноса

Код бота почти не переписывал. Поменялся рантайм и планировщик:

- `GOOS=js GOARCH=wasm` вместо `linux/amd64`;
- `http.DefaultClient` через Workers `fetch` (обычный `net/http` на JS падает с `Illegal invocation`);
- зона `Europe/Istanbul` через embed `time/tzdata` — в Wasm нет IANA zoneinfo;
- EventBridge one-shot заменил календарём в R2 и кроном `*/10 * * * *`.

Жертва точности: after-game может опоздать до 10 минут. Для «спасибо за игру» это нормально. Выигрыш: один деплой, старт isolate **в десятки раз короче** типичного cold start Lambda, объектное хранилище с always-free лимитами, которые не надо вылизывать под 2 000 PUT.

## Вывод

Если рисовать «бесплатный serverless» только по миллионам запросов, AWS, GCP и Azure выглядят почти одинаково. Если добавить **холодный старт** и **срок жизни free tier хранилища**, картинка другая.

- **AWS** — отличный always-free Lambda и привычный S3, но S3-щедрость на год, а cold start Go-Lambda заметный.
- **GCP** — больше бесплатных вызовов functions, storage always free только в трёх регионах США.
- **Azure** — похожий grant на Functions, но Blob не в комплекте, cold start Consumption часто самый длинный.
- **Cloudflare Workers** — самый короткий старт из этой четвёрки (isolate, у меня 16–37 ms). Free plan сильный по запросам; для тяжёлого Go Wasm нужен Paid за $5. **R2** при этом даёт лучший always-free объектный тир: 10 ГБ, миллион записей, 10 миллионов чтений, без egress.

Для этого бота я остановился на Workers + R2. Не потому что AWS «перестал быть бесплатным», а потому что первый апдейт после паузы больше не ждёт, пока проснётся Lambda.
