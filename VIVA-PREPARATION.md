# Viva Preparation — Case Study 15: Amazon SQS and Amazon SNS

**Student:** Vedh Naik (B.Tech Computer Science Engineering)  
**Case Study:** Case Study 15 — Amazon SQS and Amazon SNS  
**AWS Region:** `ap-south-1` (Asia Pacific (Mumbai))  

---

## A. 45-Second Project Introduction (Ready to Speak)

> "Good morning/afternoon Sir/Ma'am. My project is **Case Study 15: Amazon SQS and Amazon SNS**, where I implemented a decoupled communication design for an online shopping application in AWS (`ap-south-1`).  
> When a customer places an order, we don't want the order to be lost if the backend is busy, so we use **Amazon SQS** (`OrderProcessingQueue`, a Standard Queue) to reliably buffer order messages for asynchronous processing. Once an order is processed, we want to notify the customer—and potentially other systems—so we use **Amazon SNS** (`OrderNotifications`, a Standard Topic) with an email subscription to broadcast the confirmation.  
> In the AWS Console under my IAM user, I created both resources, sent and polled a sample JSON order (`ORD-1001`), verified visibility timeout and explicit message deletion, confirmed the email subscription, and published an order confirmation notification."

---

## B. 3-Minute Practical Demonstration Sequence (For Your Viva)

1. **Show IAM & Region (`0:00 – 0:30`):** Point to the top-right AWS Console bar showing `IAM user: user1` and region `Asia Pacific (Mumbai) (ap-south-1)`. Explain why we use an IAM user instead of Root for security.
2. **Show Amazon SQS (`0:30 – 1:30`):** Open `OrderProcessingQueue` → **Details** tab. Show `Standard` type, `30 Seconds` visibility timeout, `4 Days` retention, and `SSE-SQS` encryption. Open **Send and receive messages**, send a sample JSON order, click **Poll for messages**, inspect the body, and explain why you must **explicitly delete** it after processing.
3. **Show Amazon SNS (`1:30 – 2:30`):** Open `OrderNotifications` → **Subscriptions** tab. Show the `Email` subscription in `Confirmed` state (explain why `PendingConfirmation` prevents spam). Click **Publish message**, send an order confirmation subject/body, and show the published Message ID.
4. **Explain the Architecture (`2:30 – 3:00`):** Show [ARCHITECTURE.md](./ARCHITECTURE.md) and clarify that SQS and SNS are **not** automatically wired together—a conceptual **Order Processing Worker** would poll SQS, process the order, publish to SNS, and delete the SQS message.

---

## C. 25+ Viva Interview Questions & Answers

### 1. What is Amazon SQS?
- **Simple Answer:** Amazon SQS (Simple Queue Service) is a managed message queue service that lets applications store and pass messages asynchronously without losing them.
- **Technical Explanation:** It is a fully managed, distributed message queuing service supporting point-to-point (one-to-one) communication. Producers send messages to a queue, and consumers poll the queue to retrieve and process them independently.
- **Project Example:** We created `OrderProcessingQueue` in `ap-south-1` to hold incoming customer orders like `ORD-1001` until a worker is ready to process them.

### 2. What is a message queue?
- **Simple Answer:** A message queue is like a waiting line for tasks—messages stay in the queue until a worker picks them up one by one.
- **Technical Explanation:** It is an asynchronous buffer between a producer and a consumer that stores messages durably, decoupling the sender's speed from the receiver's processing speed.
- **Project Example:** If 1,000 customers place orders at once during a sale, `OrderProcessingQueue` buffers them safely instead of crashing the order-processing worker.

### 3. What is a producer in SQS?
- **Simple Answer:** A producer is the component that creates and sends a message into the queue.
- **Technical Explanation:** Any application or service calling the `SendMessage` API (or using the Console's *Send message* interface) to enqueue a payload into an SQS queue.
- **Project Example:** The **Online Shopping Application** (simulated by us in the AWS Console) acts as the producer when it sends the JSON payload for `ORD-1001` (`Wireless Headphones`) into `OrderProcessingQueue`.

### 4. What is a consumer in SQS?
- **Simple Answer:** A consumer is the worker that reads messages from the queue, processes them, and deletes them.
- **Technical Explanation:** A compute process (e.g., EC2, ECS, or Lambda) that polls the queue using `ReceiveMessage`, executes business logic, and calls `DeleteMessage` upon success.
- **Project Example:** Our conceptual **Order Processing Worker** (simulated via the Console's *Poll for messages* and *Delete* actions) is the consumer.

### 5. What is asynchronous communication?
- **Simple Answer:** It means the sender drops off a request and moves on immediately without waiting for the receiver to finish processing it.
- **Technical Explanation:** Unlike synchronous request-response (where the caller blocks until processing completes), asynchronous communication decouples sender and receiver via a buffer (like SQS), improving fault tolerance and responsiveness.
- **Project Example:** The customer's checkout page doesn't freeze while inventory, payment, and email are processed; the order is placed in `OrderProcessingQueue` and handled in the background.

### 6. What is Visibility Timeout in SQS?
- **Simple Answer:** It is a temporary "lock" timer after a worker picks up a message so other workers don't process the same message at the same time.
- **Technical Explanation:** When a consumer receives a message, SQS hides it from other consumers for the configured visibility timeout duration. If the consumer deletes the message before the timer expires, it's gone; if the consumer crashes, the message becomes visible again after the timeout for another worker to retry.
- **Project Example:** `OrderProcessingQueue` has a visibility timeout of **30 seconds**. When we polled `ORD-1001`, it was hidden from other pollers for 30 seconds until we deleted it.

### 7. What is Message Retention Period in SQS?
- **Simple Answer:** It is how long SQS keeps an unprocessed message in the queue before automatically deleting it.
- **Technical Explanation:** Configurable from 1 minute up to 14 days (default is 4 days). If no consumer processes and deletes a message within this window, SQS purges it.
- **Project Example:** `OrderProcessingQueue` uses the default retention of **4 days**, meaning an order message can wait up to 4 days if all workers are offline.

### 8. What is Polling (Short Polling vs. Long Polling) in SQS?
- **Simple Answer:** Polling is how a consumer checks the queue for new messages. Short polling returns immediately even if empty; long polling waits a bit for a message to arrive.
- **Technical Explanation:** Short polling (`WaitTimeSeconds = 0`) queries a subset of SQS servers and returns immediately, which can produce empty responses. Long polling (`WaitTimeSeconds` up to 20s) holds the connection open until a message arrives or the timer expires, reducing empty responses and cost.
- **Project Example:** `OrderProcessingQueue` has a default Receive message wait time of **0 seconds** (short polling), and we manually clicked *Poll for messages* in the Console to retrieve `ORD-1001`.

### 9. What is a Dead-Letter Queue (DLQ)?
- **Simple Answer:** A DLQ is a separate "quarantine" queue where failed messages are sent after too many unsuccessful processing attempts.
- **Technical Explanation:** Configured via a Redrive Policy with a `maxReceiveCount`. If a message is received and not deleted more than `maxReceiveCount` times (e.g., due to malformed data or bugs), SQS moves it to the DLQ so it doesn't block or infinitely loop in the main queue.
- **Project Example:** We kept DLQ **disabled** in `OrderProcessingQueue` to keep the practical simple and within Free Tier scope, though in production we would enable one.

### 10. What happens when a worker fails to process an SQS message?
- **Simple Answer:** The worker doesn't delete the message, so after the visibility timeout expires, the message reappears in the queue for another worker to try again.
- **Technical Explanation:** Because SQS uses at-least-once delivery and requires explicit deletion, a crashed consumer simply lets the visibility timeout expire (`30s` in our queue). The message returns to the queue with an incremented `ReceiveCount`.
- **Project Example:** If our conceptual Order Processing Worker crashed while validating `ORD-1001`, the message would become visible again in `OrderProcessingQueue` after 30 seconds.

### 11. Why must we explicitly delete messages in SQS?
- **Simple Answer:** Receiving a message does **not** delete it. We delete it only after processing succeeds so SQS knows the job is truly done.
- **Technical Explanation:** Separating `ReceiveMessage` from `DeleteMessage` guarantees reliability: if a network drop or worker crash occurs mid-processing, the message isn't lost—it will be retried.
- **Project Example:** After polling `ORD-1001` and verifying its JSON body in the Console, we explicitly selected the message and clicked **Delete** to confirm `0` messages remained.

### 12. What is the difference between Standard and FIFO queues?
- **Simple Answer:** Standard queues are super fast and allow occasional duplicates or out-of-order delivery; FIFO queues guarantee exact order and no duplicates.
- **Technical Explanation:** **Standard** queues offer nearly unlimited throughput, at-least-once delivery, and best-effort ordering. **FIFO** (First-In-First-Out) queues preserve strict order and exactly-once processing using deduplication IDs, with lower throughput limits.
- **Project Example:** We chose a **Standard** queue (`OrderProcessingQueue`) because e-commerce order intake prioritizes high throughput and simplicity, and individual orders (`ORD-1001`) can be processed independently.

### 13. Can duplicate messages occur in SQS?
- **Simple Answer:** Yes, in Standard queues a message can occasionally be delivered more than once, or if a worker takes longer than the visibility timeout.
- **Technical Explanation:** Standard queues guarantee *at-least-once* delivery due to distributed architecture. Also, if a worker exceeds the 30-second visibility timeout before deleting, another worker can receive the same message. Consumers should therefore be **idempotent** (safe to run twice).
- **Project Example:** If `ORD-1001` were delivered twice, checking `orderId: "ORD-1001"` in the database before confirming prevents charging or shipping twice.

### 14. What is Amazon SNS?
- **Simple Answer:** Amazon SNS (Simple Notification Service) is a managed publish-subscribe messaging service that pushes notifications to multiple subscribers at once.
- **Technical Explanation:** SNS is a pub/sub service where publishers send messages to a logical **Topic**, and SNS automatically pushes copies of that message to all confirmed **Subscribers** (Email, SMS, Lambda, SQS, HTTP/S, mobile push).
- **Project Example:** We created the `OrderNotifications` SNS topic in `ap-south-1` to send order confirmation emails.

### 15. What is a Topic in SNS?
- **Simple Answer:** A topic is a central communication channel or "broadcast group" where publishers send events and subscribers listen.
- **Technical Explanation:** An SNS Topic is a logical access point (identified by an ARN) that decouples publishers from subscribers. Publishers only need the Topic ARN, not the list of individual recipients.
- **Project Example:** Our topic is named `OrderNotifications` (Standard topic in `ap-south-1`).

### 16. What is a Publisher vs. a Subscriber in SNS?
- **Simple Answer:** A publisher sends a message to the topic; a subscriber is a recipient that gets a copy of that message.
- **Technical Explanation:** Publishers call `Publish` on a Topic ARN. Subscribers register an endpoint (protocol + address) on that topic and receive pushed payloads whenever a message is published.
- **Project Example:** In our practical, we acted as the publisher via the Console's **Publish message** button, and `naikvedh1@gmail.com` (`Email` protocol) was the subscriber.

### 17. What is the Publish-Subscribe (Pub/Sub) model?
- **Simple Answer:** It's a one-to-many broadcasting pattern where a sender publishes an event once, and everyone subscribed to that topic receives it.
- **Technical Explanation:** Unlike point-to-point queuing (one message → one consumer), pub/sub sends every published message to *all* active subscribers of the topic simultaneously, decoupling event producers from multiple independent consumers.
- **Project Example:** Publishing `Order Confirmed - ORD-1001` to `OrderNotifications` notifies the email subscriber—and could simultaneously trigger an SMS or analytics service if subscribed.

### 18. What is Fan-Out in AWS?
- **Simple Answer:** Fan-out means sending one message to an SNS topic and having it automatically delivered to many different subscribers or queues in parallel.
- **Technical Explanation:** A common cloud pattern where a single event published to SNS is replicated to multiple endpoints (e.g., multiple SQS queues, Lambda functions, and email endpoints) without the publisher knowing about each downstream system.
- **Project Example:** `OrderNotifications` can fan out an `OrderConfirmed` event to our email subscriber and, conceptually, to an SMS service or warehouse system at the same time.

### 19. What is an Email Subscription and why must it be confirmed?
- **Simple Answer:** An email subscription sends SNS messages to an email address, and AWS requires clicking a confirmation link first to prevent spam.
- **Technical Explanation:** When an Email endpoint is subscribed, SNS sets its status to `PendingConfirmation` and sends a verification email with a `Confirm subscription` URL. Until the owner clicks it, SNS will not deliver published messages to that address.
- **Project Example:** We subscribed `naikvedh1@gmail.com` to `OrderNotifications`, paused at `PendingConfirmation`, manually confirmed it from Gmail, and only then published `Order Confirmed - ORD-1001`.

### 20. How does SNS distribute messages (Push vs. Pull)?
- **Simple Answer:** SNS uses a **push** model—it immediately sends messages out to subscribers as soon as they are published.
- **Technical Explanation:** Unlike SQS (where consumers **pull**/poll for messages), SNS actively **pushes** messages to subscribed endpoints over HTTP/S, Email, SMS, Lambda invocation, or SQS delivery.
- **Project Example:** As soon as we clicked **Publish message** on `OrderNotifications`, SNS pushed the email to `naikvedh1@gmail.com` without the inbox having to poll AWS.

### 21. Why do we use BOTH SQS and SNS in this application?
- **Simple Answer:** SQS is for reliably processing each order one by one, while SNS is for broadcasting notifications once the order is confirmed.
- **Technical Explanation:** Order processing requires durable buffering, load leveling, and single-consumer execution (**SQS**). Order notification is an event broadcast that may need to reach email, SMS, or other services simultaneously (**SNS**).
- **Project Example:** `OrderProcessingQueue` buffers `ORD-1001` (`PLACED`), and `OrderNotifications` broadcasts `ORD-1001` (`CONFIRMED`).

### 22. Why not use ONLY SQS or ONLY SNS?
- **Simple Answer:** SQS alone can't easily push emails or broadcast to many services at once; SNS alone doesn't hold messages in a durable queue if a worker is offline.
- **Technical Explanation:** Using **only SQS** means only one consumer gets each message and there is no built-in email/SMS push. Using **only SNS** means if a subscriber/worker is down or overwhelmed, there is no pull-based queue buffering or visibility-timeout retry for heavy order processing.
- **Project Example:** Combining `OrderProcessingQueue` + `OrderNotifications` gives both reliable order buffering and instant customer email notifications.

### 23. Does SQS automatically send messages to SNS?
- **Simple Answer:** No! SQS and SNS are separate services and are not automatically connected here.
- **Technical Explanation:** While SNS *can* push to an SQS queue (SNS → SQS fan-out), SQS does **not** automatically push to SNS. In our architecture, a **consumer worker** reads from SQS, processes the order, publishes to SNS, and then deletes the SQS message.
- **Project Example:** In our practical, we manually performed the worker's steps in the Console: polled/deleted from `OrderProcessingQueue`, then published to `OrderNotifications`.

### 24. What happens if the Order Processing Worker is temporarily unavailable?
- **Simple Answer:** Nothing is lost—incoming orders simply wait safely in the SQS queue until the worker comes back online.
- **Technical Explanation:** Because SQS persists messages across multiple Availability Zones for up to the retention period (`4 Days` in our queue), orders remain queued even if the worker crashes or scales down, and are processed once the worker resumes polling.
- **Project Example:** Even with no live worker deployed in our practical, `ORD-1001` stayed safely in `OrderProcessingQueue` until we polled it.

### 25. Which AWS region and resources did we actually deploy vs. keep conceptual?
- **Simple Answer:** In `ap-south-1` (Mumbai), we actually created `OrderProcessingQueue` (SQS Standard), `OrderNotifications` (SNS Standard), and one confirmed Email subscription (`naikvedh1@gmail.com`). The shopping app and worker were conceptual.
- **Technical Explanation:** Under the AWS Free account plan using IAM user `user1`, we avoided paid/unnecessary compute (no EC2, Lambda, RDS, or VPC) and demonstrated the full message lifecycle directly via the AWS Management Console.
- **Project Example:** Actual AWS resources: `OrderProcessingQueue` + `OrderNotifications` + Email subscription. Conceptual: Customer, Online Shopping Application, and Order Processing Worker.
