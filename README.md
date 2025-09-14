# Easy_pay
I built a Payment State Monitor system that addresses a common UPI problem—transactions stuck in pending state. I used AWS SQS to queue pending payments, Lambda workers to check gateway status, and SNS/EventBridge to notify users in real-time about the reason behind delays. This not only improves transparency but also reduces duplicate payments and disputes. I believe this approach could significantly improve user trust in digital payments.


**The architechture that I have followed**

[ User App (UPI Frontend: React) ]
                |
                v
    [ Backend API (Spring Boot) ]
                |
                v
   ┌─────────────────────────────────────────────┐
   │  1. Payment Initiation                      │
   │  - Send request to Payment Gateway API      │
   │  - Store status in DB (Initiated)           │
   └─────────────────────────────────────────────┘
                |
                v
   ┌───────────────────────────┐
   │ If Success/Fail → Update  │
   └───────────────────────────┘
                |
                v
   ┌─────────────────────────────────────────────┐
   │  2. Pending Transactions                    │
   │  - Push txnId + metadata into SQS Queue     │
   └─────────────────────────────────────────────┘
                |
                v
   ┌─────────────────────────────────────────────┐
   │  3. Lambda Worker (Retry Engine)            │
   │  - Polls SQS for "pending" transactions     │
   │  - Calls Payment Gateway periodically       │
   │  - Updates status in DB                     │
   └─────────────────────────────────────────────┘
                |
                v
   ┌─────────────────────────────────────────────┐
   │  4. EventBridge Rules                       │
   │  - On status update → Trigger Event         │
   │  - Routes to notification service           │
   └─────────────────────────────────────────────┘
                |
                v
   ┌─────────────────────────────────────────────┐
   │  5. Notification Layer (SNS)   │
   │  - Payer & Payee get messages:              │
   │     ⏳ Pending reason (Bank downtime)        │
   │     ✅ Success (Txn complete)                │
   │     ❌ Failure (Refund initiated)            │
   └─────────────────────────────────────────────┘

**AWS Services Used**

API Gateway + Backend (Spring Boot) → Handles transaction request.

Amazon RDS → Stores transaction states.

Amazon SQS → Holds pending transactions for retries.

AWS Lambda → Retry engine, polls queue, checks payment status.

Amazon EventBridge → Listens for transaction state updates and routes events.

Amazon SNS → Sends alerts to users.

CloudWatch → Monitoring + alerts if queue grows unusually (indicating systemic issue).

