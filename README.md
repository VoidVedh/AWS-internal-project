# Case Study 15: Amazon SQS and Amazon SNS

**Student Name:** Vedh Naik  
**Course:** B.Tech Computer Science Engineering  
**Assignment:** AWS Practical — Case Study 15  
**AWS Region:** Asia Pacific (Mumbai) (`ap-south-1`)  

---

## 1. Problem Statement
> *"An online shopping application needs to send order notifications and process customer orders. It wants to understand how message queues and notification services help different application components communicate."*

### Required Tasks
1. Explore **Amazon SQS** and **Amazon SNS** and explain the difference between a **message queue** and a **notification topic**.
2. Draw a simple architecture diagram showing how an online shopping application can use **SQS** for order processing and **SNS** for sending notifications.
3. Demonstrate the workflow in the AWS Management Console within the AWS Free account plan without deploying unnecessary paid infrastructure.

---

## 2. Project Objectives
- Understand **asynchronous, decoupled communication** in cloud applications.
- Implement a real **Amazon SQS Standard Queue** (`OrderProcessingQueue`) in `ap-south-1` and test sending, polling, inspecting, and explicitly deleting a JSON order payload (`ORD-1001`).
- Implement a real **Amazon SNS Standard Topic** (`OrderNotifications`) in `ap-south-1`, subscribe an email endpoint (`naikvedh1@gmail.com`), verify manual confirmation, and publish an order confirmation notification.
- Clearly distinguish between **actual AWS resources created** and **conceptual application components** (the shopping application frontend and the order-processing worker).

---

## 3. AWS Services, Technology & Platform
- **Cloud Provider:** Amazon Web Services (AWS) — Free Account Plan
- **Region:** `ap-south-1` (Asia Pacific — Mumbai)
- **Services Implemented:**
  - **Amazon Simple Queue Service (SQS):** Standard Queue (`OrderProcessingQueue`)
  - **Amazon Simple Notification Service (SNS):** Standard Topic (`OrderNotifications`) + Email Subscription (`naikvedh1@gmail.com`)
  - **AWS IAM:** IAM User `user1` configured with `AmazonSQSFullAccess` and `AmazonSNSFullAccess`
- **Management Interface:** AWS Management Console via Google Chrome

---

## 4. What is Amazon SQS?
**Amazon Simple Queue Service (SQS)** is a fully managed **message queuing** service that enables decoupled, point-to-point asynchronous communication between software components:
- A **Producer** (e.g., the Online Shopping Application) sends an order message into the queue.
- The queue **buffers and persists** the message redundantly across multiple Availability Zones for up to its retention period (`4 Days` by default).
- A **Consumer** (e.g., the Order Processing Worker) **polls (pulls)** messages from the queue, processes them while they are hidden by a **Visibility Timeout** (`30 Seconds` by default), and **explicitly deletes** each message once processing succeeds.

---

## 5. What is Amazon SNS?
**Amazon Simple Notification Service (SNS)** is a fully managed **publish-subscribe (pub/sub)** notification service that enables one-to-many event broadcasting:
- A **Publisher** sends a single message/event to a central **Topic** (`OrderNotifications`).
- SNS immediately **pushes** a copy of that message to all confirmed **Subscribers** (Email, SMS, AWS Lambda, SQS queues, HTTP/S webhooks) in parallel—a pattern known as **Fan-Out**.

---

## 6. Difference Between Amazon SQS and Amazon SNS

| Feature | Amazon SQS (Message Queue) | Amazon SNS (Notification Topic) |
| :--- | :--- | :--- |
| **Communication Model** | **Queue (Point-to-Point / 1-to-1)** | **Publish-Subscribe (Pub/Sub / 1-to-Many)** |
| **Delivery Mechanism** | **Pull (Polling):** Consumers actively poll the queue for messages | **Push:** SNS actively pushes messages out to subscribers |
| **Message Persistence** | Retains messages from 1 minute to 14 days (`4 Days` default) until deleted | Does not retain messages like a queue; pushes immediately to confirmed subscribers |
| **Consumer Handling** | One message is processed by **one** consumer (protected by Visibility Timeout) | One published message is broadcast to **all** subscribed endpoints (Fan-Out) |
| **Role in Case Study 15** | Buffers customer orders (`OrderProcessingQueue`) so no order is lost if workers are busy | Sends order confirmation alerts (`OrderNotifications`) to the customer's email |

---

## 7. Architecture Overview
See [ARCHITECTURE.md](./ARCHITECTURE.md) for the full Mermaid diagram and component breakdown.

```text
Customer (Conceptual)
   │
   ▼
Online Shopping Application (Conceptual Producer)
   │  [Sends Order JSON: ORD-1001]
   ▼
Amazon SQS — OrderProcessingQueue [ACTUAL AWS RESOURCE]
   │  [Worker Polls, Processes & Explicitly Deletes Message]
   ▼
Order Processing Worker (Conceptual Consumer & Publisher)
   │  [Publishes Order Confirmation Event]
   ▼
Amazon SNS — OrderNotifications [ACTUAL AWS RESOURCE]
   │  [Pushes Email Notification]
   ▼
Email Subscriber — naikvedh1@gmail.com [ACTUAL AWS SUBSCRIPTION]
```

> **Important Architectural Note:** Amazon SQS and Amazon SNS are **not** automatically connected. In a full production deployment, an **Order Processing Worker** polls SQS, validates/processes the order, publishes an `OrderConfirmed` event to SNS, and then deletes the SQS message. In this practical, we executed those producer, worker, and publisher operations through the AWS Management Console.

---

## 8. Actual AWS Resource Inventory
See [RESOURCE-INVENTORY.md](./RESOURCE-INVENTORY.md) for full details (with AWS Account ID redacted as `<ACCOUNT_ID>`):

| Resource Name | Service | Type | Region | Status |
| :--- | :--- | :--- | :--- | :--- |
| `OrderProcessingQueue` | Amazon SQS | Standard Queue | `ap-south-1` | **Created & Verified** |
| `OrderNotifications` | Amazon SNS | Standard Topic | `ap-south-1` | **Created & Verified** |
| `naikvedh1@gmail.com` | Amazon SNS | Email Subscription (`b320063a-3cd3-40a5-afbb-d5b18f691d10`) | `ap-south-1` | **Confirmed & Verified** |

---

## 9. Implementation Workflow & Testing Methodology
1. **Environment & IAM Setup:** Configured `user1` with `AmazonSQSFullAccess` and `AmazonSNSFullAccess` and selected `Asia Pacific (Mumbai)` (`ap-south-1`).
2. **SQS Queue Creation & Test:**
   - Created Standard queue `OrderProcessingQueue` (`30 Seconds` visibility timeout, `4 Days` retention, `SSE-SQS`).
   - Sent JSON order payload (`orderId: "ORD-1001"`, `Wireless Headphones`, `quantity: 1`, `status: "PLACED"`).
   - Verified send confirmation (**Message ID:** `fd668429-dde6-4731-8333-e7409afc9ae7`, **MD5:** `48940e2d9e01afd5a8ddbd581e03054a`).
   - Polled for messages, retrieved `fd668429-dde6-4731-8333-e7409afc9ae7` (`Size: 120 bytes`, `Receive count: 1`), verified the JSON body, and **explicitly deleted** the message (`Messages available: 0`).
3. **SNS Topic, Email Subscription & Test:**
   - Created Standard topic `OrderNotifications`.
   - Subscribed `naikvedh1@gmail.com` (`Email` protocol, **Subscription ID:** `b320063a-3cd3-40a5-afbb-d5b18f691d10`).
   - Paused while status showed `Pending confirmation`, confirmed the subscription link from the Gmail inbox, and verified status changed to **`Confirmed`**.
   - Published one test notification (`Subject: Order Confirmed - ORD-1001`) and verified AWS SNS accepted publication (**Message ID:** `a3f9d3a6-a8c2-5400-afe1-b0d390b39721`).

---

## 10. Results & Limitations
### Results
- Successfully demonstrated point-to-point order queuing in **Amazon SQS** and pub/sub email notification delivery in **Amazon SNS** in `ap-south-1`.
- Confirmed that receiving a message in SQS only hides it during the **Visibility Timeout** (`30 Seconds`) and requires an **explicit delete** operation to remove it from the queue.
- Confirmed that SNS email subscriptions require explicit opt-in confirmation before notifications are delivered.

### Limitations (By Design for Free Tier Scope)
- **Conceptual Worker:** No EC2 instance or Lambda function was deployed to automatically bridge SQS and SNS; Console actions simulated the producer, consumer, and publisher roles.
- **No Dead-Letter Queue (DLQ):** Kept disabled for this basic demonstration.
- **Standard Ordering:** Standard queues/topics provide best-effort ordering and at-least-once delivery rather than strict FIFO ordering.

---

## 11. Conclusion
This practical successfully demonstrated how **Amazon SQS** and **Amazon SNS** solve two distinct communication challenges in an online shopping application: **SQS** buffers customer orders reliably for asynchronous background processing, while **SNS** broadcasts real-time order confirmation notifications to subscribed endpoints.

---

## Project Files
- [README.md](./README.md) — Project overview and SQS vs. SNS comparison
- [ARCHITECTURE.md](./ARCHITECTURE.md) — Mermaid architecture diagram and component flows
- [PRACTICAL-REPORT.md](./PRACTICAL-REPORT.md) — Step-by-step lab report and observations
- [RESOURCE-INVENTORY.md](./RESOURCE-INVENTORY.md) — Verified AWS resource configurations (Account ID redacted)
- [VIVA-PREPARATION.md](./VIVA-PREPARATION.md) — 45-second intro, 3-minute demo guide, and 25 viva Q&As
- [EVIDENCE/README.md](./EVIDENCE/README.md) — Screenshot checklist and browser recording evidence
