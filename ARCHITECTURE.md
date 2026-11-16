# Architecture — Case Study 15: Amazon SQS and Amazon SNS

## 1. Architecture Overview

This architecture demonstrates how an **Online Shopping Application** decouples **order processing** from **customer notifications** using **Amazon SQS** (point-to-point message queuing) and **Amazon SNS** (publish-subscribe notification delivery).

> **Scope Note (Implemented vs. Conceptual):**
> - **Actually Implemented in AWS (`ap-south-1`):** `OrderProcessingQueue` (Amazon SQS Standard Queue), `OrderNotifications` (Amazon SNS Standard Topic), and an Email Subscription (`naikvedh1@gmail.com`).
> - **Conceptual Application Components (Not Deployed):** The Online Shopping Application frontend/API, the Order Processing Worker (EC2/ECS/Lambda), and additional conceptual subscribers (e.g., SMS/Analytics). All Console operations were executed manually through the AWS Management Console to demonstrate the workflow without deploying backend compute.

---

## 2. Mermaid Architecture Diagram

```mermaid
flowchart TD
    Customer(["Customer\n(Conceptual User)"])
    App["Online Shopping Application\n(Conceptual Producer)"]
    SQS["Amazon SQS — Standard Queue\nOrderProcessingQueue\n[ACTUAL AWS RESOURCE]"]
    Worker["Order Processing Worker\n(Conceptual Consumer & Publisher)"]
    Event(["Order Confirmation Event\n(ORD-1001 CONFIRMED)"])
    SNS["Amazon SNS — Standard Topic\nOrderNotifications\n[ACTUAL AWS RESOURCE]"]
    EmailSub["Email Subscriber\nnaikvedh1@gmail.com\n[ACTUAL AWS SUBSCRIPTION]"]
    OtherSubs["Other Possible Subscribers\ne.g., SMS / Mobile Push / Analytics\n(Conceptual Examples Only)"]

    Customer -->|1. Places Order| App
    App -->|2. Sends Order JSON\nProducer -> Queue| SQS
    SQS -.->|3. Polls & Receives Message\nQueue -> Consumer| Worker
    Worker -.->|4. Validates & Processes Order\n5. Deletes Message from SQS| SQS
    Worker -.->|6. Publishes Confirmation Event\nPublisher -> Topic| Event
    Event -.-> SNS
    SNS -->|7. Fan-Out Push Delivery\nTopic -> Subscriber| EmailSub
    SNS -.->|Conceptual Fan-Out| OtherSubs

    classDef actual fill:#e6f4ea,stroke:#1e8e3e,stroke-width:2px,color:#0d652d;
    classDef conceptual fill:#f8f9fa,stroke:#5f6368,stroke-width:2px,stroke-dasharray: 5 5,color:#3c4043;

    class SQS,SNS,EmailSub actual;
    class Customer,App,Worker,Event,OtherSubs conceptual;
```

---

## 3. Component Roles & Responsibilities

| Role | Component | Type | Responsibility |
| :--- | :--- | :--- | :--- |
| **User** | Customer | Conceptual | Places an order (e.g., Wireless Headphones, Qty: 1). |
| **Producer** | Online Shopping Application | Conceptual | Accepts the customer checkout request and sends the order JSON payload to `OrderProcessingQueue`. |
| **Message Queue** | Amazon SQS (`OrderProcessingQueue`) | **Actual AWS Resource** | Durably buffers incoming order messages so the application does not lose orders if the worker is busy or temporarily down. |
| **Consumer & Publisher** | Order Processing Worker | Conceptual | Polls `OrderProcessingQueue`, processes/validates the order, publishes an `OrderConfirmed` event to `OrderNotifications`, and explicitly deletes the message from SQS. |
| **Notification Topic** | Amazon SNS (`OrderNotifications`) | **Actual AWS Resource** | Receives the confirmation event from the publisher and fans it out to all confirmed subscribers. |
| **Subscriber** | Email Subscriber (`naikvedh1@gmail.com`) | **Actual AWS Subscription** | Receives the `Order Confirmed - ORD-1001` notification email pushed by SNS. |

---

## 4. Producer-to-Consumer Flow (Amazon SQS)

1. **Send (Producer → Queue):** The Shopping Application (demonstrated via AWS Console *Send message*) places the order JSON (`ORD-1001`) into `OrderProcessingQueue`.
2. **Buffer & Persist:** SQS stores the message redundantly across Availability Zones for up to the configured retention period (`4 Days`).
3. **Receive (Queue → Consumer):** The Order Processing Worker (demonstrated via AWS Console *Poll for messages*) retrieves the message.
4. **Visibility Timeout (`30 Seconds`):** Once received, the message becomes temporarily invisible to other consumers for 30 seconds so two workers do not process the same order simultaneously.
5. **Explicit Delete:** After processing succeeds, the consumer **must explicitly delete** the message from `OrderProcessingQueue`. Receiving a message does **not** automatically delete it—if the worker crashes before deleting, the message becomes visible again after 30 seconds for retry.

---

## 5. Publisher-to-Subscriber Flow (Amazon SNS)

1. **Subscribe & Confirm:** An endpoint (`naikvedh1@gmail.com` via `Email` protocol) subscribes to `OrderNotifications`. It remains in `PendingConfirmation` until the recipient manually clicks the confirmation link, preventing spam.
2. **Publish (Publisher → Topic):** Once the order is confirmed, the publisher (demonstrated via AWS Console *Publish message*) sends the confirmation subject and body to `OrderNotifications`.
3. **Fan-Out Push (Topic → Subscribers):** SNS immediately pushes the message to all confirmed subscribers. While this practical demonstrates one Email subscriber, SNS can fan out the same event simultaneously to multiple subscribers (Email, Lambda, SQS, HTTP/S, etc.).

---

## 6. Relationship Between SQS and SNS

- **No Automatic SQS → SNS Link:** Amazon SQS does **not** automatically forward or publish messages to Amazon SNS.
- **How They Work Together:** In a real application, the **Order Processing Worker** acts as the bridge:
  1. It **consumes** the raw order from SQS (`OrderProcessingQueue`).
  2. It validates payment/inventory and updates the database.
  3. It **publishes** the `Order Confirmed` event to SNS (`OrderNotifications`).
  4. It **deletes** the processed message from SQS.
- **Why Both Are Needed:**
  - **SQS** ensures reliable, one-to-one, asynchronous **work queuing** (pull-based) so no order is lost during traffic spikes.
  - **SNS** provides one-to-many, real-time **event broadcasting** (push-based) so multiple notification channels or downstream services can react to the confirmation at once.
