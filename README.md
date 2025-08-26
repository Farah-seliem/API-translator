# API-translator
Serverless translation API using AWS Lambda, API Gateway, and Amazon Translate.
Auto-detects source language and returns English text.

🚀 Architecture

Amazon API Gateway → Exposes /translate endpoint

AWS Lambda (Python) → Handles requests, calls Amazon Translate

Amazon Translate → Detects source language & translates to English

IAM Role → Grants Lambda access to Translate

⚡ Quick Deploy
1️⃣ Create IAM Role

Go to IAM → Roles → Create Role

Choose Lambda as trusted entity

Attach policy:

AmazonTranslateFullAccess

CloudWatchLogsFullAccess (for logging)

Save role as TranslateLambdaRole.

2️⃣ Create Lambda Function

Runtime: Python 3.12

Role: TranslateLambdaRole

Upload code (lambda_function.py):

import json
import boto3

translate = boto3.client("translate")

def lambda_handler(event, context):
    try:
        body = json.loads(event.get("body", "{}"))
        text = body.get("text", "").strip()

        if not text:
            return {
                "statusCode": 400,
                "body": json.dumps({"error": "Missing field 'text'"})
            }

        # Call Amazon Translate
        response = translate.translate_text(
            Text=text,
            SourceLanguageCode="auto",
            TargetLanguageCode="en"
        )

        return {
            "statusCode": 200,
            "body": json.dumps({
                "detectedSourceLang": response["SourceLanguageCode"],
                "targetLang": "en",
                "translatedText": response["TranslatedText"]
            })
        }

    except Exception as e:
        return {
            "statusCode": 500,
            "body": json.dumps({"error": str(e)})
        }

3️⃣ Create API Gateway

Go to API Gateway → Create API → HTTP API

Add route: POST /translate

Integration: Lambda → select your function

Deploy API

4️⃣ Test Endpoint
curl -X POST https://<api-id>.execute-api.<region>.amazonaws.com/translate \
  -H "Content-Type: application/json" \
  -d '{"text":"Hola, ¿cómo estás?"}'


Response:

{
  "detectedSourceLang": "es",
  "targetLang": "en",
  "translatedText": "Hello, how are you?"
}

🔐 Security (Optional)

Enable API Key in API Gateway

Add WAF (Web Application Firewall)

Use CloudWatch Logs to monitor requests

📦 Deployment with AWS SAM (Optional)

template.yaml:

AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Resources:
  TranslateApi:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: .
      Handler: lambda_function.lambda_handler
      Runtime: python3.12
      Policies:
        - TranslateFullAccess
      Events:
        TranslateApi:
          Type: Api
          Properties:
            Path: /translate
            Method: post
