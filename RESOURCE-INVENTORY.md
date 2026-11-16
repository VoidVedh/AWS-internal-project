# Resource Inventory — Case Study 15: Amazon SQS and Amazon SNS

> **Security Note:** The 12-digit AWS Account ID has been redacted as `<ACCOUNT_ID>` across all URLs and ARNs in public documentation.

---

## 1. Actual AWS Resources Created

| Resource Name / ID | AWS Service | Resource Type | Region | Actual Configuration | Creation Status | Verification Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **`OrderProcessingQueue`** | Amazon SQS | Standard Queue | `ap-south-1` (Asia Pacific — Mumbai) | • **URL:** `https://sqs.ap-south-1.amazonaws.com/<ACCOUNT_ID>/OrderProcessingQueue`<br>• **ARN:** `arn:aws:sqs:ap-south-1:<ACCOUNT_ID>:OrderProcessingQueue`<br>• **Visibility Timeout:** `30 Seconds`<br>• **Message Retention:** `4 Days`<br>• **Delivery Delay:** `0 Seconds`<br>• **Receive Wait Time:** `0 Seconds`<br>• **Max Message Size:** `1024 KiB`<br>• **Encryption:** `Amazon SQS key (SSE-SQS)`<br>• **Dead-Letter Queue:** Disabled (`-`) | **COMPLETED** (`2026-11-16T22:22+05:30`) | **COMPLETED** — Sent, polled, inspected, and deleted `ORD-1001` (`Message ID: fd668429-dde6-4731-8333-e7409afc9ae7`, `0` messages remaining) |
| **`OrderNotifications`** | Amazon SNS | Standard Topic | `ap-south-1` (Asia Pacific — Mumbai) | • **ARN:** `arn:aws:sns:ap-south-1:<ACCOUNT_ID>:OrderNotifications`<br>• **Type:** `Standard`<br>• **Access Policy:** Default topic policy | **COMPLETED** (`2026-11-16`) | **COMPLETED** — Published `Order Confirmed - ORD-1001` (`Message ID: a3f9d3a6-a8c2-5400-afe1-b0d390b39721`) |
| **`b320063a-3cd3-40a5-afbb-d5b18f691d10`** | Amazon SNS | Email Subscription | `ap-south-1` (Asia Pacific — Mumbai) | • **ARN:** `arn:aws:sns:ap-south-1:<ACCOUNT_ID>:OrderNotifications:b320063a-3cd3-40a5-afbb-d5b18f691d10`<br>• **Topic:** `OrderNotifications`<br>• **Protocol:** `EMAIL`<br>• **Endpoint:** `naikvedh1@gmail.com` | **COMPLETED** (`2026-11-16`) | **COMPLETED** — Manually confirmed via email link; SNS Console status verified as `Confirmed` |
| **`user1`** | AWS IAM | IAM User | Global | • **Managed Policies Attached:**<br>  1. `AmazonSQSFullAccess`<br>  2. `AmazonSNSFullAccess`<br>  3. `IAMUserChangePassword` | **COMPLETED** | **COMPLETED** — Verified `Permissions policies (3)` attached |

---

## 2. Test Messages & Artifacts Logged

| Test Operation | Service | Identifier / Metadata | Final State |
| :--- | :--- | :--- | :--- |
| **Sample Order Enqueued (`ORD-1001`)** | Amazon SQS (`OrderProcessingQueue`) | • **Message ID:** `fd668429-dde6-4731-8333-e7409afc9ae7`<br>• **MD5:** `48940e2d9e01afd5a8ddbd581e03054a`<br>• **Size:** `120 bytes`<br>• **Receive Count:** `1` | **Explicitly Deleted** after retrieval verification (`Messages available: 0`) |
| **Order Confirmation Published (`ORD-1001`)** | Amazon SNS (`OrderNotifications`) | • **Subject:** `Order Confirmed - ORD-1001`<br>• **Message ID:** `a3f9d3a6-a8c2-5400-afe1-b0d390b39721`<br>• **Request ID:** `8378613b-76f2-5975-a3c4-024e4e835a54` | **Published** to `OrderNotifications` and dispatched to `naikvedh1@gmail.com` |

---

## 3. Conceptual Components (Not Deployed as Live AWS Compute)

| Component | Role in Architecture | Implementation Status | Reason |
| :--- | :--- | :--- | :--- |
| **Online Shopping Application** | Producer (sends order JSON to SQS) | **NOT IMPLEMENTED** (Simulated via AWS Console *Send message*) | Out of scope; avoids deploying unnecessary web servers or API Gateway |
| **Order Processing Worker** | Consumer & Publisher (polls SQS, processes order, publishes to SNS, deletes SQS message) | **NOT IMPLEMENTED** (Simulated via AWS Console *Poll*, *Delete*, and *Publish*) | Out of scope; avoids creating EC2 instances or Lambda functions per assignment restrictions |
