# Implementation Evidence — Case Study 15: Amazon SQS and Amazon SNS

All 12 screenshots below were extracted directly from the live Google Chrome AWS Management Console session in `ap-south-1` (with the 12-digit AWS Account ID automatically redacted).

---

## Saved Screenshot Files in `EVIDENCE/`

| # | File Name | Verified Content |
| :--- | :--- | :--- |
| **01** | [01-aws-console-identity-and-region.png](./01-aws-console-identity-and-region.png) | IAM user `user1` with `AmazonSQSFullAccess` and `AmazonSNSFullAccess` attached (`2 policies added to user1`). |
| **02** | [02-sqs-queue-list.png](./02-sqs-queue-list.png) | Amazon SQS Queues page in `Asia Pacific (Mumbai)` (`ap-south-1`). |
| **03** | [03-sqs-queue-details.png](./03-sqs-queue-details.png) | `OrderProcessingQueue` created (`Standard`, `30 Seconds` visibility timeout, `4 Days` retention, `SSE-SQS`). |
| **04** | [04-sqs-message-sent.png](./04-sqs-message-sent.png) | Sample order `ORD-1001` sent (`Message ID: fd668429-dde6-4731-8333-e7409afc9ae7`, `MD5: 48940e2d9e01afd5a8ddbd581e03054a`). |
| **05** | [05-sqs-message-received.png](./05-sqs-message-received.png) | Polled message `fd668429-dde6-4731-8333-e7409afc9ae7` (`Size: 120 bytes`, `Receive count: 1`) showing exact JSON payload. |
| **06** | [06-sqs-queue-after-cleanup.png](./06-sqs-queue-after-cleanup.png) | `"1 message deleted successfully."` and `Messages (0)` after explicit deletion. |
| **07** | [07-sns-topic-list.png](./07-sns-topic-list.png) | Amazon SNS Topics view in `ap-south-1`. |
| **08** | [08-sns-topic-details.png](./08-sns-topic-details.png) | `"Topic OrderNotifications created successfully."` (`Type: Standard`, `ap-south-1`). |
| **09** | [09-sns-email-subscription-confirmed.png](./09-sns-email-subscription-confirmed.png) | Subscription `b320063a-3cd3-40a5-afbb-d5b18f691d10` (`naikvedh1@gmail.com`) showing status **`Confirmed`**. |
| **10** | [10-sns-message-publication.png](./10-sns-message-publication.png) | Publish message form filled with Subject `Order Confirmed - ORD-1001` and confirmation body. |
| **11** | [11-sns-publication-result.png](./11-sns-publication-result.png) | `"Message published to topic OrderNotifications successfully."` (`Message ID: a3f9d3a6-a8c2-5400-afe1-b0d390b39721`). |
| **12** | [12-architecture-diagram.png](./12-architecture-diagram.png) | Visual architecture diagram separating actual AWS resources from conceptual application components. |
