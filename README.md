# IoT温湿度モニタリングシステム

このプロジェクトは、SwitchBot温湿度センサー、MacBook、およびAWSサービス（DynamoDB、AWS Lambdaなど）を使用して、簡単なIoTシステムを構築する方法を示します。このシステムはセンサーデータを収集し、DynamoDBに保存し、事前に定義された条件に基づいて通知をトリガーします。

## 目次

- [概要](#概要)
- [システム構成](#システム構成)
- [要件](#要件)
- [セットアップ](#セットアップ)
  - [AWSセットアップ](#awsセットアップ)
  - [ローカルセットアップ](#ローカルセットアップ)
- [使用方法](#使用方法)
  - [データ挿入スクリプト](#データ挿入スクリプト)
  - [Webアプリケーション](#webアプリケーション)
- [今後の改善](#今後の改善)
- [貢献](#貢献)
- [ライセンス](#ライセンス)

## 概要

このプロジェクトは、基本的なIoTシステムを示します。以下の機能を持っています。
1. SwitchBotセンサーから温度と湿度データを収集します。
2. データをAWS DynamoDBに送信します。
3. 特定の条件を満たすと通知をトリガーします（例：温度が閾値を超える）。

## システム構成

![システム構成](./architecture_diagram.png)

システム構成には以下が含まれます：
- **SwitchBotセンサー**: 温度と湿度データを収集します。
- **MacBook**: Bluetoothを介してセンサーからデータを受信し、AWSに送信します。
- **AWS DynamoDB**: センサーデータを保存します。
- **AWS Lambda**: データを処理し、通知をトリガーします。

## 要件

- **ハードウェア**:
  - SwitchBot温湿度センサー
  - Bluetooth対応のMacBook
- **ソフトウェア**:
  - Python 3.8以上
  - AWS CLI
  - Flask（Webアプリケーション用）
- **AWSサービス**:
  - DynamoDB
  - Lambda
  - SNS（通知用）

## セットアップ

### AWSセットアップ

1. **DynamoDBテーブルの作成**:
   ```sh
   aws dynamodb create-table --table-name TrialSensorData \
       --attribute-definitions AttributeName=ID,AttributeType=S \
       --key-schema AttributeName=ID,KeyType=HASH \
       --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1
   ```
2. **AWS Lambda関数の作成**:

   1. Lambda関数の作成:
      - AWSマネジメントコンソールでLambdaサービスに移動します。
      - 「関数の作成」をクリックします。
      - 「一から作成」を選択し、関数名を入力し、ランタイムとしてPython 3.xを選択します。
      - 必要なロールを作成または選択します。

   2. コードのアップロード:
      - Lambda関数のコードとして以下を使用します。
        ```python
        import json
        import boto3
        from decimal import Decimal

        def lambda_handler(event, context):
            dynamodb = boto3.resource('dynamodb')
            table = dynamodb.Table('TrialSensorData')

            # イベントからデータを抽出
            data = json.loads(event['body'])

            table.put_item(
                Item={
                    'ID': data['ID'],
                    'Temperature': Decimal(data['Temperature']),
                    'Humidity': Decimal(data['Humidity'])
                }
            )

            return {
                'statusCode': 200,
                'body': json.dumps('Data inserted successfully')
            }
        ```

   3. トリガーの設定:
      - Lambda関数にHTTPトリガー（API Gateway）を設定します。
      - 新しいAPIを作成し、メソッドとしてPOSTを選択します。

### ローカルセットアップ

1. **Python環境のセットアップ**:
   ```sh
   pip install boto3 Flask 
   ```

2. **AWS CLIのインストール**
  ```sh
    curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
    sudo installer -pkg AWSCLIV2.pkg -target /
    aws configure
  ```



