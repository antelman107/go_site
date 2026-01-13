+++
title = 'Деплой Telegram бота на AWS Lambda с Function URL'
date = 2024-12-20T09:00:00+03:00
draft = false
tags = ['go', 'telegram', 'aws', 'lambda', 'deploy', 'function-url', 'cli']
url = '/ru/post/go-telegram-bot-aws-lambda-deploy.html'

[quiz]
  [[quiz.questions]]
    question = "Какой runtime используется для Go функций в AWS Lambda?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "provided.al2023"
      correct = true
    [[quiz.questions.answers]]
      text = "go1.x"
      correct = false
    [[quiz.questions.answers]]
      text = "go1.21"
      correct = false
  
  [[quiz.questions]]
    question = "Какая архитектура предпочтительнее для Lambda функций на Go?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "arm64 (дешевле и быстрее)"
      correct = true
    [[quiz.questions.answers]]
      text = "x86_64"
      correct = false
    [[quiz.questions.answers]]
      text = "Не имеет значения"
      correct = false
  
  [[quiz.questions]]
    question = "Какой тип аутентификации Function URL используется для Telegram webhook?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "NONE (публичный доступ)"
      correct = true
    [[quiz.questions.answers]]
      text = "AWS_IAM"
      correct = false
    [[quiz.questions.answers]]
      text = "OAuth"
      correct = false
  
  [[quiz.questions]]
    question = "Какое имя исполняемого файла требуется для Lambda функции на Go?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "bootstrap"
      correct = true
    [[quiz.questions.answers]]
      text = "main"
      correct = false
    [[quiz.questions.answers]]
      text = "handler"
      correct = false
  
  [[quiz.questions]]
    question = "Какие преимущества имеет Lambda Function URL по сравнению с API Gateway?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "Проще настроить"
      correct = true
    [[quiz.questions.answers]]
      text = "Ниже задержка"
      correct = true
    [[quiz.questions.answers]]
      text = "Дешевле"
      correct = true
    [[quiz.questions.answers]]
      text = "Больше возможностей для rate limiting"
      correct = false
  
  [[quiz.questions]]
    question = "Как проверить подключение к AWS после настройки CLI?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "aws sts get-caller-identity"
      correct = true
    [[quiz.questions.answers]]
      text = "aws configure list"
      correct = false
    [[quiz.questions.answers]]
      text = "aws login"
      correct = false
  
  [[quiz.questions]]
    question = "Где хранятся логи Lambda функций?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "AWS CloudWatch Logs"
      correct = true
    [[quiz.questions.answers]]
      text = "S3"
      correct = false
    [[quiz.questions.answers]]
      text = "DynamoDB"
      correct = false
  
  [[quiz.questions]]
    question = "Какой метод аутентификации AWS CLI рекомендуется для организаций?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "AWS IAM Identity Center (SSO)"
      correct = true
    [[quiz.questions.answers]]
      text = "Прямые IAM учетные данные"
      correct = false
    [[quiz.questions.answers]]
      text = "SSH ключи"
      correct = false
+++

Эта статья является продолжением статьи ["Создание AI-телеграм бота на Go с Gemini API и AWS Lambda"](/ru/post/go-telegram-gemini-ai-bot.html) и содержит детальные инструкции по настройке и деплою Telegram бота на AWS Lambda с использованием Function URL.

<!--more-->

## Введение

В предыдущей статье мы рассмотрели архитектуру Telegram бота с AI роутером и интеграцией с Gemini API. Теперь мы детально разберём процесс деплоя этого бота на AWS Lambda, включая настройку AWS CLI, создание Lambda функции, настройку Function URL и автоматизацию процесса деплоя.

## Подготовка: AWS CLI и аутентификация

Перед началом работы с AWS Lambda необходимо установить и настроить AWS CLI. AWS CLI - это унифицированный инструмент командной строки для управления сервисами AWS.

### Установка AWS CLI

```bash
# macOS
brew install awscli

# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Проверка установки
aws --version
```

### Настройка аутентификации

AWS CLI использует несколько методов аутентификации. Самый простой и безопасный способ для начала работы - использование AWS IAM Identity Center (ранее AWS SSO) или прямые учетные данные IAM.

**Метод 1: AWS IAM Identity Center (рекомендуется для организаций)**

```bash
# Настройка SSO профиля
aws configure sso

# Вам будет предложено:
# - SSO start URL: https://your-company.awsapps.com/start
# - SSO region: us-east-1 (или ваш регион)
# - SSO account ID: 123456789012
# - SSO role name: AdministratorAccess (или другая роль)
# - CLI default region: us-east-1
# - CLI default output format: json

# Вход в SSO
aws sso login --profile your-profile-name
```

**Метод 2: Прямые учетные данные IAM (для личных проектов)**

```bash
# Настройка учетных данных
aws configure

# Вам будет предложено ввести:
# AWS Access Key ID: AKIAIOSFODNN7EXAMPLE
# AWS Secret Access Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
# Default region name: us-east-1
# Default output format: json
```

### Проверка подключения

```bash
# Проверка текущего пользователя/роли
aws sts get-caller-identity

# Должен вернуть что-то вроде:
# {
#     "UserId": "AIDAIOSFODNN7EXAMPLE",
#     "Account": "123456789012",
#     "Arn": "arn:aws:iam::123456789012:user/your-username"
# }
```

### Работа с несколькими профилями

Если у вас несколько AWS аккаунтов, вы можете использовать профили:

```bash
# Создание профиля
aws configure --profile production

# Использование профиля
aws lambda list-functions --profile production

# Или установка переменной окружения
export AWS_PROFILE=production
```

## Архитектура обработки запросов

Перед тем как перейти к созданию Lambda функции, давайте разберём, как AWS обрабатывает запрос от Telegram:

```mermaid
sequenceDiagram
    participant User as Пользователь Telegram
    participant TG as Telegram API
    participant FunctionURL as Lambda Function URL
    participant Lambda as Lambda Function
    participant S3 as AWS S3
    participant Gemini as Google Gemini API
    participant CloudWatch as CloudWatch Logs

    User->>TG: Отправляет сообщение боту
    TG->>FunctionURL: POST webhook (JSON update)
    FunctionURL->>Lambda: Вызывает функцию
    Lambda->>CloudWatch: Логирует начало выполнения
    
    Lambda->>S3: Загружает историю разговора (chatID)
    S3-->>Lambda: Возвращает историю (JSON)
    
    Lambda->>Lambda: Роутер определяет тип запроса
    alt Текстовый запрос
        Lambda->>Gemini: Chat API запрос с контекстом
        Gemini-->>Lambda: Текстовый ответ
    else Генерация изображения
        Lambda->>Gemini: Image Generation API
        Gemini-->>Lambda: Изображение
    end
    
    Lambda->>TG: Отправляет ответ пользователю
    Lambda->>S3: Сохраняет обновлённую историю
    Lambda->>CloudWatch: Логирует завершение
    
    Lambda-->>FunctionURL: HTTP 200 OK
    FunctionURL-->>TG: Подтверждение получения
    TG-->>User: Показывает ответ бота
```

## Подготовка кода для Lambda

Для работы с Lambda Function URL нам нужно адаптировать код. Function URL использует стандартный HTTP формат, а не формат API Gateway. Вот адаптированный обработчик:

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "net/http"
    "os"
    
    "github.com/aws/aws-lambda-go/events"
    "github.com/aws/aws-lambda-go/lambda"
    "github.com/aws/aws-sdk-go-v2/aws"
    "github.com/aws/aws-sdk-go-v2/config"
    "github.com/aws/aws-sdk-go-v2/service/s3"
    tgbotapi "github.com/go-telegram-bot-api/telegram-bot-api/v5"
    "google.golang.org/api/option"
    "google.golang.org/api/generativeai"
)

var (
    bot    *tgbotapi.BotAPI
    gemini *generativeai.Client
    s3Client *s3.Client
)

func init() {
    // Инициализация при холодном старте Lambda
    cfg := loadConfig()
    
    var err error
    bot, err = tgbotapi.NewBotAPI(cfg.TelegramToken)
    if err != nil {
        panic(fmt.Sprintf("Failed to create bot: %v", err))
    }
    
    gemini, err = generativeai.NewClient(context.Background(),
        option.WithAPIKey(cfg.GeminiAPIKey))
    if err != nil {
        panic(fmt.Sprintf("Failed to create Gemini client: %v", err))
    }
    
    // Инициализация S3 клиента
    awsCfg, err := config.LoadDefaultConfig(context.Background())
    if err != nil {
        panic(fmt.Sprintf("Failed to load AWS config: %v", err))
    }
    s3Client = s3.NewFromConfig(awsCfg)
}

func Handler(ctx context.Context, request events.LambdaFunctionURLRequest) 
    (events.LambdaFunctionURLResponse, error) {
    
    // Парсинг Telegram webhook обновления
    var update tgbotapi.Update
    if err := json.Unmarshal([]byte(request.Body), &update); err != nil {
        return events.LambdaFunctionURLResponse{
            StatusCode: http.StatusBadRequest,
            Body:       fmt.Sprintf("Invalid request: %v", err),
        }, nil
    }
    
    // Обработка обновления асинхронно
    go handleMessage(ctx, update)
    
    // Немедленно возвращаем ответ Telegram
    return events.LambdaFunctionURLResponse{
        StatusCode: http.StatusOK,
        Body:       "OK",
    }, nil
}

func main() {
    lambda.Start(Handler)
}
```

## Создание Lambda функции через AWS CLI

Теперь создадим Lambda функцию используя современный AWS CLI v2:

### Шаг 1: Создание IAM роли для Lambda

Сначала создадим роль, которую будет использовать Lambda функция:

```bash
# Создание trust policy для Lambda
cat > trust-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Создание роли
aws iam create-role \
  --role-name telegram-bot-lambda-role \
  --assume-role-policy-document file://trust-policy.json

# Прикрепление базовой политики для выполнения Lambda
aws iam attach-role-policy \
  --role-name telegram-bot-lambda-role \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# Создание политики для доступа к S3
cat > s3-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::bot-conversations/*"
    }
  ]
}
EOF

aws iam put-role-policy \
  --role-name telegram-bot-lambda-role \
  --policy-name S3AccessPolicy \
  --policy-document file://s3-policy.json

# Получение ARN роли (нужен для создания функции)
ROLE_ARN=$(aws iam get-role \
  --role-name telegram-bot-lambda-role \
  --query 'Role.Arn' \
  --output text)

echo "Role ARN: $ROLE_ARN"
```

### Шаг 2: Создание S3 бакета для разговоров (опционально)

```bash
# Создание бакета (имена бакетов должны быть глобально уникальными)
BUCKET_NAME="telegram-bot-conversations-$(date +%s)"
aws s3 mb s3://$BUCKET_NAME --region us-east-1

# Сохранение имени бакета для дальнейшего использования
echo "BUCKET_NAME=$BUCKET_NAME" > .env.bucket
```

### Шаг 3: Сборка Go приложения

```bash
# Сборка для Linux (требуется для Lambda)
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -o bootstrap

# Создание ZIP архива (Lambda требует именно bootstrap как имя исполняемого файла)
zip function.zip bootstrap

# Проверка размера (Lambda имеет ограничение на размер deployment package)
ls -lh function.zip
```

### Шаг 4: Создание Lambda функции

```bash
# Создание функции (замените $ROLE_ARN на значение из предыдущего шага)
aws lambda create-function \
  --function-name telegram-bot \
  --runtime provided.al2023 \
  --role $ROLE_ARN \
  --handler bootstrap \
  --zip-file fileb://function.zip \
  --timeout 30 \
  --memory-size 512 \
  --architectures arm64 \
  --environment Variables="{
    TELEGRAM_TOKEN=your-telegram-token,
    GEMINI_API_KEY=your-gemini-api-key,
    S3_BUCKET=$BUCKET_NAME
  }" \
  --description "Telegram bot with Gemini AI integration"

# Проверка создания функции
aws lambda get-function --function-name telegram-bot
```

**Важные параметры:**
- `--runtime provided.al2023`: Использует Amazon Linux 2023 runtime для Go
- `--architectures arm64`: Использует ARM64 архитектуру (дешевле и быстрее)
- `--timeout 30`: Максимальное время выполнения (секунды)
- `--memory-size 512`: Выделенная память (MB). Больше памяти = больше CPU

## Создание Lambda Function URL

Lambda Function URL - это простой способ предоставить HTTP(S) endpoint для Lambda функции без необходимости настраивать API Gateway. Это идеально подходит для Telegram webhook:

```bash
# Создание Function URL с CORS (если нужно)
aws lambda create-function-url-config \
  --function-name telegram-bot \
  --auth-type NONE \
  --cors '{
    "AllowOrigins": ["*"],
    "AllowMethods": ["POST"],
    "AllowHeaders": ["content-type"],
    "MaxAge": 300
  }'

# Получение URL функции
FUNCTION_URL=$(aws lambda get-function-url-config \
  --function-name telegram-bot \
  --query 'FunctionUrl' \
  --output text)

echo "Function URL: $FUNCTION_URL"

# Добавление разрешения на публичный доступ
aws lambda add-permission \
  --function-name telegram-bot \
  --statement-id FunctionURLAllowPublicAccess \
  --action lambda:InvokeFunctionUrl \
  --principal "*" \
  --function-url-auth-type NONE
```

**Типы аутентификации Function URL:**
- `NONE`: Публичный доступ (подходит для Telegram webhook)
- `AWS_IAM`: Требует подпись AWS Signature Version 4

## Настройка Telegram Webhook

Теперь настроим Telegram для отправки обновлений на наш Function URL:

```bash
# Установка webhook
TELEGRAM_TOKEN="your-telegram-bot-token"
curl -X POST "https://api.telegram.org/bot${TELEGRAM_TOKEN}/setWebhook" \
  -H "Content-Type: application/json" \
  -d "{\"url\": \"${FUNCTION_URL}\"}"

# Проверка статуса webhook
curl "https://api.telegram.org/bot${TELEGRAM_TOKEN}/getWebhookInfo"
```

## Обновление кода функции (деплой)

Когда вам нужно обновить код функции:

```bash
# Пересборка
GOOS=linux GOARCH=arm64 CGO_ENABLED=0 go build -o bootstrap
zip function.zip bootstrap

# Обновление кода функции
aws lambda update-function-code \
  --function-name telegram-bot \
  --zip-file fileb://function.zip \
  --architectures arm64

# Обновление переменных окружения (если нужно)
aws lambda update-function-configuration \
  --function-name telegram-bot \
  --environment Variables="{
    TELEGRAM_TOKEN=your-new-token,
    GEMINI_API_KEY=your-new-key,
    S3_BUCKET=$BUCKET_NAME
  }"

# Ожидание завершения обновления
aws lambda wait function-updated --function-name telegram-bot
```

## Мониторинг и логирование

AWS Lambda автоматически отправляет логи в CloudWatch:

```bash
# Просмотр последних логов
aws logs tail /aws/lambda/telegram-bot --follow

# Просмотр метрик функции
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Invocations \
  --dimensions Name=FunctionName,Value=telegram-bot \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Sum

# Просмотр информации о функции
aws lambda get-function \
  --function-name telegram-bot \
  --query 'Configuration.[FunctionName,Runtime,LastModified,MemorySize,Timeout]' \
  --output table
```

## Автоматизация деплоя (Makefile)

Для упрощения процесса деплоя можно создать Makefile:

```makefile
.PHONY: build deploy update logs clean

FUNCTION_NAME=telegram-bot
REGION=us-east-1

build:
	@echo "Building for Lambda..."
	GOOS=linux GOARCH=arm64 CGO_ENABLED=0 go build -o bootstrap
	zip function.zip bootstrap
	@echo "Build complete. Size:"
	@ls -lh function.zip

deploy: build
	@echo "Deploying to Lambda..."
	aws lambda update-function-code \
		--function-name $(FUNCTION_NAME) \
		--zip-file fileb://function.zip \
		--region $(REGION)
	@echo "Waiting for update to complete..."
	aws lambda wait function-updated \
		--function-name $(FUNCTION_NAME) \
		--region $(REGION)
	@echo "Deployment complete!"

update-env:
	@echo "Updating environment variables..."
	aws lambda update-function-configuration \
		--function-name $(FUNCTION_NAME) \
		--environment Variables="{TELEGRAM_TOKEN=$(TELEGRAM_TOKEN),GEMINI_API_KEY=$(GEMINI_API_KEY),S3_BUCKET=$(S3_BUCKET)}" \
		--region $(REGION)

logs:
	@echo "Tailing logs..."
	aws logs tail /aws/lambda/$(FUNCTION_NAME) --follow --region $(REGION)

clean:
	rm -f bootstrap function.zip
	@echo "Clean complete"
```

Использование:

```bash
# Установка переменных окружения
export TELEGRAM_TOKEN="your-token"
export GEMINI_API_KEY="your-key"
export S3_BUCKET="your-bucket-name"

# Сборка и деплой
make deploy

# Просмотр логов
make logs
```

## Преимущества Function URL vs API Gateway

**Lambda Function URL:**
- ✅ Проще настроить (один AWS CLI вызов)
- ✅ Ниже задержка (нет промежуточного слоя)
- ✅ Дешевле (нет платы за API Gateway)
- ✅ Идеально для простых webhook

**API Gateway:**
- ✅ Больше возможностей (rate limiting, API keys, caching)
- ✅ Поддержка REST и WebSocket
- ✅ Интеграция с другими AWS сервисами
- ✅ Лучше для публичных API

Для Telegram бота Function URL - оптимальный выбор благодаря простоте и экономичности.

## Заключение

В этой статье мы рассмотрели детальный процесс деплоя Telegram бота на AWS Lambda с использованием Function URL. Мы настроили AWS CLI, создали IAM роли, развернули Lambda функцию и настроили webhook интеграцию с Telegram. Использование Function URL упрощает процесс по сравнению с API Gateway и делает деплой более экономичным для простых webhook сценариев.

Для получения дополнительной информации о создании самого бота и его архитектуре, обратитесь к предыдущей статье: ["Создание AI-телеграм бота на Go с Gemini API и AWS Lambda"](/ru/post/go-telegram-gemini-ai-bot.html).

