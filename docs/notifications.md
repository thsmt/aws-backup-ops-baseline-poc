# 通知

## 概要

AWS Backup の通知方法は複数あります。

このリポジトリでは、CloudFormation で構成しやすく、運用でも説明しやすい EventBridge + SNS を基本構成にしています。

<br>

## 通知方式

| 方式 | 通知先 | 特徴 |
| --- | --- | --- |
| AWS Backup native notification | SNS | Backup Vault 単位で AWS Backup イベントを通知 |
| EventBridge | SNS / Lambda / SQS など | Backup、Copy、Restore、Vault など幅広いイベントを扱える |
| AWS User Notifications | Console / Email / Mobile / Chat | AWS通知を集約して管理できる |
| Amazon Q Developer in chat applications | Slack / Microsoft Teams / Amazon Chime | SNS通知をチャットへ転送できる |
| Lambda Webhook | Slack / LINE / Teams など | メッセージ整形や外部API連携を柔軟に実装できる |

<br>

## このテンプレートで作成する通知

このテンプレートでは、EventBridge Rule で以下の AWS Backup イベントを検知し、SNS Topic に通知します。

| イベント | 状態 |
| --- | --- |
| Backup Job State Change | `FAILED` / `EXPIRED` / `ABORTED` |
| Copy Job State Change | `FAILED` / `EXPIRED` / `ABORTED` |
| Restore Job State Change | `FAILED` / `EXPIRED` / `ABORTED` |

通常運用では、成功通知をすべて送ると通知量が多くなりやすいため、失敗系イベントを優先して通知します。

<br>

## メール通知

SNS Topic にメール購読を追加することで、バックアップ失敗をメールで受け取れます。

CloudFormation でメール購読を作成する場合は、以下を設定します。

| パラメータ | 値 |
| --- | --- |
| `CreateEmailSubscription` | `true` |
| `NotificationEmail` | 通知先メールアドレス |

スタック作成後、メールに届く SNS 購読確認を承認すると通知が有効になります。

<br>

## Slack / Microsoft Teams 通知

Slack や Microsoft Teams へ通知したい場合は、Amazon Q Developer in chat applications を利用する方法が自然です。

構成例:

```text
AWS Backup
  -> EventBridge
  -> SNS Topic
  -> Amazon Q Developer in chat applications
  -> Slack / Microsoft Teams
```

Amazon Q Developer in chat applications は SNS Topic の通知をチャットチャンネルへ転送できます。

<br>

## LINE 通知

LINE へ通知したい場合は、EventBridge または SNS から Lambda を呼び出し、LINE Messaging API へ送信する構成が考えられます。

構成例:

```text
AWS Backup
  -> EventBridge
  -> Lambda
  -> LINE Messaging API
```

LINE のアクセストークンは Secrets Manager などで管理し、Lambda 環境変数に直接埋め込まないようにします。

<br>

## 通知方式の使い分け

| 要件 | 推奨方式 |
| --- | --- |
| まずメールで受け取りたい | SNS Email |
| Slack / Teams に運用通知を集約したい | Amazon Q Developer in chat applications |
| 通知文面を整形したい | EventBridge + Lambda |
| LINE などAWS外部サービスへ送りたい | EventBridge + Lambda + 外部API |
| 複数サービスの通知を一元管理したい | AWS User Notifications |

<br>

## 参考

- [Notification options with AWS Backup](https://docs.aws.amazon.com/aws-backup/latest/devguide/backup-notifications.html)
- [Monitoring AWS Backup events using Amazon EventBridge](https://docs.aws.amazon.com/aws-backup/latest/devguide/eventbridge.html)
- [What is Amazon Q Developer in chat applications?](https://docs.aws.amazon.com/chatbot/latest/adminguide/what-is.html)
- [What is AWS User Notifications?](https://docs.aws.amazon.com/notifications/latest/userguide/what-is-service.html)
