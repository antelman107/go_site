+++
title = 'Deploying Telegram Bot to AWS Lambda with Function URL'
date = 2024-12-20T09:00:00+03:00
draft = false
tags = ['go', 'telegram', 'aws', 'lambda', 'deploy', 'function-url', 'cli']
url = '/en/post/go-telegram-bot-aws-lambda-deploy.html'

[quiz]
  [[quiz.questions]]
    question = "What runtime is used for Go functions in AWS Lambda?"
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
    question = "Which architecture is preferred for Lambda functions on Go?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "arm64 (cheaper and faster)"
      correct = true
    [[quiz.questions.answers]]
      text = "x86_64"
      correct = false
    [[quiz.questions.answers]]
      text = "Doesn't matter"
      correct = false
  
  [[quiz.questions]]
    question = "What Function URL authentication type is used for Telegram webhook?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "NONE (public access)"
      correct = true
    [[quiz.questions.answers]]
      text = "AWS_IAM"
      correct = false
    [[quiz.questions.answers]]
      text = "OAuth"
      correct = false
  
  [[quiz.questions]]
    question = "What executable name is required for Lambda function on Go?"
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
    question = "What advantages does Lambda Function URL have over API Gateway?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "Simpler to set up"
      correct = true
    [[quiz.questions.answers]]
      text = "Lower latency"
      correct = true
    [[quiz.questions.answers]]
      text = "Cheaper"
      correct = true
    [[quiz.questions.answers]]
      text = "More rate limiting features"
      correct = false
  
  [[quiz.questions]]
    question = "How to verify AWS connection after CLI setup?"
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
    question = "Where are Lambda function logs stored?"
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
    question = "Which AWS CLI authentication method is recommended for organizations?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "AWS IAM Identity Center (SSO)"
      correct = true
    [[quiz.questions.answers]]
      text = "Direct IAM credentials"
      correct = false
    [[quiz.questions.answers]]
      text = "SSH keys"
      correct = false
+++

This article is a continuation of ["Building an AI Telegram Bot with Go, Gemini API, and AWS Lambda"](/en/post/go-telegram-gemini-ai-bot.html) and contains detailed instructions for setting up and deploying a Telegram bot to AWS Lambda using Function URL.

<!--more-->

## Introduction

In the previous article, we explored the architecture of a Telegram bot with an AI router and Gemini API integration. Now we'll dive deep into the deployment process of this bot to AWS Lambda, including AWS CLI setup, Lambda function creation, Function URL configuration, and deployment automation.

## Setup: AWS CLI and Authentication

Before working with AWS Lambda, you need to install and configure AWS CLI. AWS CLI is a unified command-line tool for managing AWS services.

### Installing AWS CLI

```bash
# macOS
brew install awscli

# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Verify installation
aws --version
```

### Configuring Authentication

AWS CLI uses several authentication methods. The simplest and most secure way to get started is using AWS IAM Identity Center (formerly AWS SSO) or direct IAM credentials.

**Method 1: AWS IAM Identity Center (recommended for organizations)**

```bash
# Configure SSO profile
aws configure sso

# You'll be prompted for:
# - SSO start URL: https://your-company.awsapps.com/start
# - SSO region: us-east-1 (or your region)
# - SSO account ID: 123456789012
# - SSO role name: AdministratorAccess (or another role)
# - CLI default region: us-east-1
# - CLI default output format: json

# Login to SSO
aws sso login --profile your-profile-name
```

**Method 2: Direct IAM Credentials (for personal projects)**

```bash
# Configure credentials
aws configure

# You'll be prompted to enter:
# AWS Access Key ID: AKIAIOSFODNN7EXAMPLE
# AWS Secret Access Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
# Default region name: us-east-1
# Default output format: json
```

### Verify Connection

```bash
# Check current user/role
aws sts get-caller-identity

# Should return something like:
# {
#     "UserId": "AIDAIOSFODNN7EXAMPLE",
#     "Account": "123456789012",
#     "Arn": "arn:aws:iam::123456789012:user/your-username"
# }
```

### Working with Multiple Profiles

If you have multiple AWS accounts, you can use profiles:

```bash
# Create a profile
aws configure --profile production

# Use a profile
aws lambda list-functions --profile production

# Or set environment variable
export AWS_PROFILE=production
```

## Request Processing Architecture

Before diving into creating the Lambda function, let's understand how AWS processes requests from Telegram:

```mermaid
sequenceDiagram
    participant User as Telegram User
    participant TG as Telegram API
    participant FunctionURL as Lambda Function URL
    participant Lambda as Lambda Function
    participant S3 as AWS S3
    participant Gemini as Google Gemini API
    participant CloudWatch as CloudWatch Logs

    User->>TG: Sends message to bot
    TG->>FunctionURL: POST webhook (JSON update)
    FunctionURL->>Lambda: Invokes function
    Lambda->>CloudWatch: Logs execution start
    
    Lambda->>S3: Loads conversation history (chatID)
    S3-->>Lambda: Returns history (JSON)
    
    Lambda->>Lambda: Router determines request type
    alt Text Request
        Lambda->>Gemini: Chat API request with context
        Gemini-->>Lambda: Text response
    else Image Generation
        Lambda->>Gemini: Image Generation API
        Gemini-->>Lambda: Image
    end
    
    Lambda->>TG: Sends response to user
    Lambda->>S3: Saves updated history
    Lambda->>CloudWatch: Logs completion
    
    Lambda-->>FunctionURL: HTTP 200 OK
    FunctionURL-->>TG: Acknowledgment
    TG-->>User: Shows bot response
```

## Preparing Code for Lambda

To work with Lambda Function URL, we need to adapt our code. Function URL uses standard HTTP format, not API Gateway format. Here's the adapted handler:

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
    // Initialize on Lambda cold start
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
    
    // Initialize S3 client
    awsCfg, err := config.LoadDefaultConfig(context.Background())
    if err != nil {
        panic(fmt.Sprintf("Failed to load AWS config: %v", err))
    }
    s3Client = s3.NewFromConfig(awsCfg)
}

func Handler(ctx context.Context, request events.LambdaFunctionURLRequest) 
    (events.LambdaFunctionURLResponse, error) {
    
    // Parse Telegram webhook update
    var update tgbotapi.Update
    if err := json.Unmarshal([]byte(request.Body), &update); err != nil {
        return events.LambdaFunctionURLResponse{
            StatusCode: http.StatusBadRequest,
            Body:       fmt.Sprintf("Invalid request: %v", err),
        }, nil
    }
    
    // Process update asynchronously
    go handleMessage(ctx, update)
    
    // Immediately return response to Telegram
    return events.LambdaFunctionURLResponse{
        StatusCode: http.StatusOK,
        Body:       "OK",
    }, nil
}

func main() {
    lambda.Start(Handler)
}
```

## Creating Lambda Function via AWS CLI

Now let's create the Lambda function using modern AWS CLI v2:

### Step 1: Create IAM Role for Lambda

First, let's create a role that the Lambda function will use:

```bash
# Create trust policy for Lambda
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

# Create role
aws iam create-role \
  --role-name telegram-bot-lambda-role \
  --assume-role-policy-document file://trust-policy.json

# Attach basic execution policy for Lambda
aws iam attach-role-policy \
  --role-name telegram-bot-lambda-role \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# Create policy for S3 access
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

# Get role ARN (needed for function creation)
ROLE_ARN=$(aws iam get-role \
  --role-name telegram-bot-lambda-role \
  --query 'Role.Arn' \
  --output text)

echo "Role ARN: $ROLE_ARN"
```

### Step 2: Create S3 Bucket for Conversations (optional)

```bash
# Create bucket (bucket names must be globally unique)
BUCKET_NAME="telegram-bot-conversations-$(date +%s)"
aws s3 mb s3://$BUCKET_NAME --region us-east-1

# Save bucket name for later use
echo "BUCKET_NAME=$BUCKET_NAME" > .env.bucket
```

### Step 3: Build Go Application

```bash
# Build for Linux (required for Lambda)
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -o bootstrap

# Create ZIP archive (Lambda requires bootstrap as executable name)
zip function.zip bootstrap

# Check size (Lambda has deployment package size limits)
ls -lh function.zip
```

### Step 4: Create Lambda Function

```bash
# Create function (replace $ROLE_ARN with value from previous step)
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

# Verify function creation
aws lambda get-function --function-name telegram-bot
```

**Important Parameters:**
- `--runtime provided.al2023`: Uses Amazon Linux 2023 runtime for Go
- `--architectures arm64`: Uses ARM64 architecture (cheaper and faster)
- `--timeout 30`: Maximum execution time (seconds)
- `--memory-size 512`: Allocated memory (MB). More memory = more CPU

## Creating Lambda Function URL

Lambda Function URL is a simple way to provide an HTTP(S) endpoint for a Lambda function without needing to set up API Gateway. This is perfect for Telegram webhooks:

```bash
# Create Function URL with CORS (if needed)
aws lambda create-function-url-config \
  --function-name telegram-bot \
  --auth-type NONE \
  --cors '{
    "AllowOrigins": ["*"],
    "AllowMethods": ["POST"],
    "AllowHeaders": ["content-type"],
    "MaxAge": 300
  }'

# Get function URL
FUNCTION_URL=$(aws lambda get-function-url-config \
  --function-name telegram-bot \
  --query 'FunctionUrl' \
  --output text)

echo "Function URL: $FUNCTION_URL"

# Add permission for public access
aws lambda add-permission \
  --function-name telegram-bot \
  --statement-id FunctionURLAllowPublicAccess \
  --action lambda:InvokeFunctionUrl \
  --principal "*" \
  --function-url-auth-type NONE
```

**Function URL Authentication Types:**
- `NONE`: Public access (suitable for Telegram webhook)
- `AWS_IAM`: Requires AWS Signature Version 4 signing

## Configure Telegram Webhook

Now let's configure Telegram to send updates to our Function URL:

```bash
# Set webhook
TELEGRAM_TOKEN="your-telegram-bot-token"
curl -X POST "https://api.telegram.org/bot${TELEGRAM_TOKEN}/setWebhook" \
  -H "Content-Type: application/json" \
  -d "{\"url\": \"${FUNCTION_URL}\"}"

# Check webhook status
curl "https://api.telegram.org/bot${TELEGRAM_TOKEN}/getWebhookInfo"
```

## Updating Function Code (Deployment)

When you need to update the function code:

```bash
# Rebuild
GOOS=linux GOARCH=arm64 CGO_ENABLED=0 go build -o bootstrap
zip function.zip bootstrap

# Update function code
aws lambda update-function-code \
  --function-name telegram-bot \
  --zip-file fileb://function.zip \
  --architectures arm64

# Update environment variables (if needed)
aws lambda update-function-configuration \
  --function-name telegram-bot \
  --environment Variables="{
    TELEGRAM_TOKEN=your-new-token,
    GEMINI_API_KEY=your-new-key,
    S3_BUCKET=$BUCKET_NAME
  }"

# Wait for update to complete
aws lambda wait function-updated --function-name telegram-bot
```

## Monitoring and Logging

AWS Lambda automatically sends logs to CloudWatch:

```bash
# View recent logs
aws logs tail /aws/lambda/telegram-bot --follow

# View function metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Invocations \
  --dimensions Name=FunctionName,Value=telegram-bot \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Sum

# View function information
aws lambda get-function \
  --function-name telegram-bot \
  --query 'Configuration.[FunctionName,Runtime,LastModified,MemorySize,Timeout]' \
  --output table
```

## Deployment Automation (Makefile)

To simplify the deployment process, you can create a Makefile:

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

Usage:

```bash
# Set environment variables
export TELEGRAM_TOKEN="your-token"
export GEMINI_API_KEY="your-key"
export S3_BUCKET="your-bucket-name"

# Build and deploy
make deploy

# View logs
make logs
```

## Function URL vs API Gateway Benefits

**Lambda Function URL:**
- ✅ Simpler to set up (single AWS CLI call)
- ✅ Lower latency (no intermediate layer)
- ✅ Cheaper (no API Gateway charges)
- ✅ Perfect for simple webhooks

**API Gateway:**
- ✅ More features (rate limiting, API keys, caching)
- ✅ REST and WebSocket support
- ✅ Integration with other AWS services
- ✅ Better for public APIs

For a Telegram bot, Function URL is the optimal choice due to simplicity and cost-effectiveness.

## Conclusion

In this article, we explored the detailed process of deploying a Telegram bot to AWS Lambda using Function URL. We set up AWS CLI, created IAM roles, deployed the Lambda function, and configured webhook integration with Telegram. Using Function URL simplifies the process compared to API Gateway and makes deployment more cost-effective for simple webhook scenarios.

For more information about creating the bot itself and its architecture, refer to the previous article: ["Building an AI Telegram Bot with Go, Gemini API, and AWS Lambda"](/en/post/go-telegram-gemini-ai-bot.html).

