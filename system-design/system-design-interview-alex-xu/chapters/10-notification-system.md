# Ch10: Design a Notification System

## What questions does this chapter answer?
- How does a notification system support iOS push, Android push, SMS, and email through a single architecture?
- Why must message queues sit between notification servers and third-party providers like APNs and FCM?
- How do you guarantee at-least-once delivery while preventing duplicate notifications?
- How do user preferences and rate limiting prevent notification spam?
- What happens when APNs or FCM is temporarily unavailable?
- How do notification templates, event tracking, and monitoring make the system operationally complete?

## Key Concepts

### Notification Channels and Third-Party Providers
A notification system must support four distinct delivery channels, each backed by a different third-party provider:

**iOS push notification** requires three components:
- **Provider**: your server builds the notification payload (device token + JSON payload) and sends it to APNs
- **APNs** (Apple Push Notification Service): Apple's remote service that propagates push notifications to iOS devices
- **iOS Device**: receives the push notification

Example iOS notification payload:
```json
{
  "aps": {
    "alert": {
      "title": "Game Request",
      "body": "Bob wants to play chess"
    },
    "badge": 9
  }
}
```

**Android push notification**: Same pattern, but uses **Firebase Cloud Messaging (FCM)** instead of APNs.

**SMS message**: Third-party services like Twilio, Nexmo, or Amazon SNS handle SMS delivery. High deliverability, but costs per message.

**Email**: SendGrid, Mailchimp, or Amazon SES handle spam filtering, delivery tracking, and templates. Better deliverability than self-hosted email servers.

Your service never communicates directly with end-user devices; it hands off to these providers, which handle last-mile delivery.

### Contact Info Gathering Flow
To send notifications, the system must first collect contact information:
- When a user installs the app or signs up, the API server collects device tokens (for push), phone numbers (for SMS), and email addresses
- This data is stored in two tables:
  - `user` table: email address, phone number
  - `device` table: device_id, user_id, device_token (push_id)
- A user can have multiple devices — a notification may be sent to all of them simultaneously

### Why Message Queues Are Essential (Three Problems Without Them)
A single notification server without queues has three critical problems:
1. **Single point of failure (SPOF)**: one server means the whole system fails if it crashes
2. **Hard to scale**: notification processing (rendering HTML, waiting for third-party responses) is resource-intensive; scaling databases, caches, and processing components independently is impossible with a monolith
3. **Performance bottleneck**: during peak hours, processing all notifications on one system causes overload

### Message Queue Architecture
The solution: insert a message queue between notification servers and delivery workers, with a **separate queue per channel**:

```
Service A/B/N → Notification Servers → [iOS Queue | Android Queue | SMS Queue | Email Queue]
                                                    ↓              ↓             ↓            ↓
                                              iOS Workers   Android Workers  SMS Workers  Email Workers
                                                    ↓              ↓             ↓            ↓
                                                 APNs            FCM         Twilio       SendGrid
```

Benefits:
- iOS workers can fail without affecting email delivery
- Each channel scales its worker pool independently
- Messages survive worker crashes — they remain in the queue until consumed
- Providers can be swapped without changing notification servers

The notification server flow:
1. Receives API call from triggering service (billing, booking, marketing)
2. Fetches metadata (user info, device tokens, notification settings) from cache/DB
3. Sends notification event to the appropriate channel queue
4. Returns immediately — no blocking on delivery

### At-Least-Once Delivery and Deduplication
Every notification is saved to a database with status `pending` before the worker attempts to send it. After a successful provider response, the status is updated to `sent`. If a worker crashes mid-delivery, the notification remains `pending` and another worker retries it — **at-least-once delivery**.

This introduces the risk of duplicate sends. To prevent duplicates reaching the user, each notification event is assigned a unique **event ID**. Before sending, the worker checks whether this event ID has already been processed (in a Redis SET or processed-events table). If yes, the send is skipped. This converts at-least-once to effectively exactly-once from the user's perspective.

### Notification Templates
A large notification system sends millions of notifications daily. Many follow similar formats. **Notification templates** avoid building every notification from scratch:

```
BODY: You dreamed of it. We dared it. [ITEM NAME] is back — only until [DATE].
CTA:  Order Now. Or, Save My [ITEM NAME]
```

Templates provide:
- **Consistent format** across all notifications of a type
- **Reduced margin for error** — parameterized templates are safer than ad-hoc strings
- **Time savings** — engineers configure parameters rather than writing full notification content

### Notification Settings (User Preferences)
Users receive too many notifications and can easily feel overwhelmed. Fine-grained opt-out control is stored in a notification settings table:

```
user_id    bigInt
channel    varchar  -- "push", "sms", "email"
opt_in     boolean
```

Before any notification is sent, the notification server checks these settings and **silently discards** the notification for any channel where the user has opted out. This check happens before the message enters the queue — no wasted queue space or worker CPU for notifications that will be discarded.

Respecting opt-outs is not optional:
- CAN-SPAM (email) and GDPR impose legal obligations
- APNs and FCM actively throttle or ban providers with too many spam reports

### Rate Limiting
Even when a user has opted in, enforce per-user send-rate limits to prevent overwhelming them. Example limits: 20 push notifications per user per day, 5 SMS per user per day, 3 marketing emails per user per week. Rate limits also protect against provider bans — APNs and FCM throttle providers that send at excessive rates. Rate limit state is tracked in Redis counters with TTL.

### Retry with Exponential Backoff
When a provider call fails, the worker retries with exponential backoff: wait 1s, then 2s, then 4s, up to a configured maximum. If all retries are exhausted, the notification is moved to a **Dead Letter Queue (DLQ)** and an alert fires to on-call engineers.

Why exponential backoff specifically: retrying immediately often hits the same transient error. Doubling the wait gives providers time to recover from temporary outages without hammering them.

### Security: AppKey and AppSecret
For iOS and Android apps, **appKey** and **appSecret** are used to secure push notification APIs. Only authenticated and verified clients are allowed to send push notifications using the API. This prevents unauthorized services from abusing the notification infrastructure.

### Monitoring Queued Notifications
A key operational metric: the **total number of queued notifications**. If this number grows, notification events are not being processed fast enough by workers. Alert threshold: if queue depth exceeds X, add more workers automatically (autoscaling) or page on-call. Without this monitoring, a silent provider outage or worker crash can cause notification delays of hours that engineers don't discover until users complain.

### Event Tracking
Notification metrics drive product decisions:
- **Open rate**: percentage of push notifications tapped by users
- **Click rate**: percentage of email notifications where the user clicked a link
- **Engagement metrics**: downstream actions taken after notification (purchases, page visits)

An analytics service captures these events. Integration between the notification system and the analytics service typically requires:
- Tracking pixels in emails (HTTP request on open)
- Click-through URLs that redirect through the analytics service
- SDK callbacks in mobile apps when a push notification is tapped

### Device Token Lifecycle
Device tokens are not permanent. When a user uninstalls the app:
- APNs returns `410 Gone` when a notification is sent to the invalid token
- FCM returns `registration-token-not-registered`

Workers must detect these responses and **immediately delete** the invalid token from the device table to prevent wasting resources on future sends. Failing to clean up stale tokens degrades deliverability metrics and wastes compute.

### Notification Lifecycle State Machine
Every notification travels through a defined set of states from creation to final user interaction. Tracking these states enables debugging, retry logic, and product analytics:

```mermaid
stateDiagram-v2
    [*] --> Pending: Notification event created\n(event_id assigned, saved to DB)
    Pending --> Sent: Worker calls provider\n(APNs / FCM / Twilio / SendGrid)\nProvider returns 200 OK
    Pending --> Failed: Provider error after\nall retries exhausted\n→ move to DLQ
    Sent --> Delivered: OS delivers notification\nto device (confirmed by SDK callback\nor delivery receipt)
    Sent --> Bounced: Invalid device token\n(APNs 410 Gone / FCM not-registered)\n→ delete token from device table
    Delivered --> Clicked: User taps notification\n(SDK callback / click-through URL)
    Delivered --> Dismissed: User swipes away
    Delivered --> Unsubscribed: User opts out from\nnotification settings
    Failed --> [*]: Alert fires to on-call\nDLQ for manual replay
    Clicked --> [*]: Analytics event recorded\n(engagement metric)
    Dismissed --> [*]: No action
    Unsubscribed --> [*]: opt_in = false written to DB
```

State transitions are the basis for product metrics: sent-to-delivered rate tells you provider health; delivered-to-clicked is the engagement metric; delivered-to-unsubscribed signals spam fatigue.

### Per-Channel Communication Protocols
Each delivery channel has a different protocol and payload format. Workers must implement each separately.

#### iOS (APNs) Communication
```mermaid
sequenceDiagram
    participant Worker as iOS Worker
    participant APNs as Apple Push Notification Service
    participant iPhone

    Note over Worker: Has device_token + JSON payload
    Worker->>APNs: HTTP/2 POST /3/device/{device_token}\nHeaders: apns-topic, apns-priority, authorization (JWT)\nBody: {"aps":{"alert":{"title":"...","body":"..."},"badge":9}}
    
    alt Success
        APNs-->>Worker: 200 OK
        APNs->>iPhone: Push notification delivered
        Worker->>DB: UPDATE status=sent
    else Invalid token
        APNs-->>Worker: 410 Gone\n{"reason": "Unregistered"}
        Worker->>DB: DELETE FROM device WHERE device_token=...
    else Provider error
        APNs-->>Worker: 500 / 503
        Worker->>Worker: Retry with exponential backoff\n(1s → 2s → 4s → DLQ)
    end
```

APNs uses HTTP/2 with a persistent connection — workers maintain a connection pool to avoid TLS handshake overhead on every notification.

#### Android (FCM) Communication
```mermaid
sequenceDiagram
    participant Worker as Android Worker
    participant FCM as Firebase Cloud Messaging
    participant Android

    Worker->>FCM: POST https://fcm.googleapis.com/fcm/send\nAuthorization: key=<server_key>\nBody: {"to": "<registration_token>",\n "notification": {"title":"...", "body":"..."},\n "data": {"order_id": "12345"}}
    
    alt Success
        FCM-->>Worker: {"success": 1, "message_id": "..."}
        FCM->>Android: Push delivered
        Worker->>DB: UPDATE status=sent
    else Stale token
        FCM-->>Worker: {"failure": 1,\n "results": [{"error": "NotRegistered"}]}
        Worker->>DB: DELETE stale token
    end
```

FCM supports two message types: **notification messages** (displayed automatically by the OS) and **data messages** (delivered silently to the app for custom handling). The payload structure differs for each.

#### SMS (Twilio) Communication
```mermaid
sequenceDiagram
    participant Worker as SMS Worker
    participant Twilio
    participant Phone

    Worker->>Twilio: POST https://api.twilio.com/2010-04-01/Accounts/{AccountSid}/Messages.json\nBasic Auth: AccountSid:AuthToken\nBody: To=+14155552671&From=+14155552672&Body=Your+order+shipped

    alt Success
        Twilio-->>Worker: 201 Created\n{"status": "queued", "sid": "SMxxxxxxx"}
        Note over Twilio,Phone: Twilio handles carrier routing
        Twilio->>Phone: SMS delivered
        Worker->>DB: UPDATE status=sent
    else Invalid number
        Twilio-->>Worker: 400 Bad Request\n{"code": 21211, "message": "Invalid To number"}
        Worker->>DB: UPDATE status=invalid
    end
```

SMS has per-message cost and regulatory constraints (TCPA in the US, GDPR in EU). Workers must not retry SMS blindly — delivery failure codes must distinguish transient errors from permanent ones (invalid number = never retry).

#### Email (SendGrid) Communication
```mermaid
sequenceDiagram
    participant Worker as Email Worker
    participant SendGrid
    participant Inbox

    Worker->>SendGrid: POST https://api.sendgrid.com/v3/mail/send\nAuthorization: Bearer <API_KEY>\nBody: {"personalizations":[{"to":[{"email":"user@example.com"}]}],\n "from":{"email":"noreply@example.com"},\n "subject":"Your order shipped",\n "content":[{"type":"text/html","value":"<html>..."}]}

    alt Delivered
        SendGrid-->>Worker: 202 Accepted
        Note over SendGrid: Handles spam filtering,\nbounce management, delivery tracking
        SendGrid->>Inbox: Email delivered
    else Hard bounce (invalid email)
        SendGrid-->>Worker: Webhook: {event:"bounce", type:"permanent"}
        Worker->>DB: Mark email as invalid,\nsuppress future sends
    else Spam complaint
        SendGrid-->>Worker: Webhook: {event:"spamreport"}
        Worker->>DB: Set opt_in=false for email channel
    end
```

Email uses **webhooks** (not synchronous responses) for delivery confirmation. Workers must expose an endpoint to receive these async callbacks from SendGrid and update the notification log accordingly.

## Architecture Diagrams

### High-Level Notification System Architecture (Improved)
This diagram shows the full flow from triggering services through queues and workers to third-party providers, including the authentication, rate-limiting, and monitoring components added in the improved design.

```mermaid
graph TD
    BS["Billing Service"] --> NS["Notification Servers\n(auth + rate limiting + validation)"]
    BookS["Booking Service"] --> NS
    MktS["Marketing Service"] --> NS

    NS --> Cache["Cache\n(user info, device tokens, templates)"]
    NS --> DB["User DB\n(device tokens, notification settings)"]

    NS --> iOSQ["iOS Queue"]
    NS --> DroidQ["Android Queue"]
    NS --> SMSQ["SMS Queue"]
    NS --> EmailQ["Email Queue"]

    iOSQ --> iOSW["iOS Workers\n(dedup check + send)"]
    DroidQ --> DroidW["Android Workers"]
    SMSQ --> SMSW["SMS Workers"]
    EmailQ --> EmailW["Email Workers"]

    iOSW --> APNs["Apple APNs"]
    DroidW --> FCM["Firebase FCM"]
    SMSW --> Twilio["Twilio / Nexmo"]
    EmailW --> SendGrid["SendGrid / Amazon SES"]

    iOSW --> NotifLog["Notification Log DB\n(status: pending → sent)"]
    DroidW --> NotifLog
    SMSW --> NotifLog
    EmailW --> NotifLog

    iOSW --> Monitor["Queue Monitor\n(alert if depth grows)"]
    iOSW --> Analytics["Analytics Service\n(open rate, click rate)"]

    APNs --> iPhone["iPhone"]
    FCM --> Android["Android Device"]
    Twilio --> Phone["Phone (SMS)"]
    SendGrid --> Inbox["Email Inbox"]
```

### Contact Info Gathering Flow

```mermaid
sequenceDiagram
    participant User
    participant App as Mobile App
    participant APIServer
    participant DB

    User->>App: Install app / sign up
    App->>App: Request push notification permission
    App->>App: Get device token from OS (APNs/FCM)
    App->>APIServer: POST /register {user_id, device_token, phone, email}
    APIServer->>DB: INSERT into device table (device_token)
    APIServer->>DB: INSERT/UPDATE user table (phone, email)
    APIServer-->>App: 200 OK
```

A user can register multiple devices. The notification server queries the device table and creates one notification job per device.

### Full Notification Delivery Sequence (iOS)

```mermaid
sequenceDiagram
    participant Service
    participant NS as Notification Server
    participant DB
    participant Queue
    participant Worker
    participant APNs
    participant iPhone

    Service->>NS: send_notification(user_id=42, message="Your order shipped")
    NS->>DB: Lookup device_tokens for user_id=42
    NS->>DB: Check opt_in for iOS channel (user_id=42, channel=push)
    NS->>NS: Check rate limit (< 20 pushes today for user 42?)
    NS->>Queue: Enqueue {event_id, device_token, payload} to iOS Queue
    NS->>DB: Save notification log (status=pending)
    NS-->>Service: 200 OK (async, not waiting for delivery)

    Queue->>Worker: Dequeue job
    Worker->>Worker: Check event_id in Redis (dedup)
    Worker->>APNs: POST /3/device/{device_token} with payload
    APNs->>iPhone: Push notification delivered
    APNs-->>Worker: 200 OK
    Worker->>DB: Update notification log (status=sent)
    Worker->>Analytics: Record delivery event (event_id, delivered_at)
```

### Retry and Dead Letter Queue Flow

```mermaid
flowchart TD
    Worker["Worker dequeues notification"] --> Send["Call APNs/FCM/Twilio/SendGrid"]
    Send --> Result{Response?}
    Result -->|"200 OK"| Done["Update DB: status=sent\nRecord in analytics"]
    Result -->|"Error (transient)"| Retry{Retry count\n< max?}
    Retry -->|"Yes"| Backoff["Wait (1s → 2s → 4s)\nexponential backoff"] --> Send
    Retry -->|"No"| DLQ["Move to Dead Letter Queue"]
    DLQ --> Alert["Alert on-call engineer"]
    Result -->|"410 Gone (APNs)\nor not-registered (FCM)"| Delete["Delete stale device token\nfrom device table"]
```

### User Preferences and Rate Limit Gate

```mermaid
flowchart LR
    Event["Notification event\nuser_id: 42, channel: push"] --> PrefCheck["Check opt_in\nWHERE user_id=42 AND channel=push"]
    PrefCheck --> OptIn{opt_in = true?}
    OptIn -->|"No"| Discard1["Discard silently"]
    OptIn -->|"Yes"| RateCheck["Check rate limit\nRedis counter: pushes_today:42"]
    RateCheck --> WithinLimit{count < 20?}
    WithinLimit -->|"No"| Discard2["Discard (rate limited)"]
    WithinLimit -->|"Yes"| Enqueue["Enqueue to iOS Queue\nIncrement rate limit counter"]
```

### Queue Depth Monitoring

```mermaid
graph LR
    iOSQ["iOS Queue"] --> Monitor["Queue Monitor\n(check depth every 30s)"]
    DroidQ["Android Queue"] --> Monitor
    Monitor --> Check{Queue depth\n> threshold?}
    Check -->|"Yes"| Scale["Autoscale workers\n+ Page on-call"]
    Check -->|"No"| OK["Normal operation"]
    Monitor --> Dashboard["Metrics Dashboard\n(queue depth over time)"]
```

A queue that's growing means workers can't keep up. This could be due to: a provider outage, insufficient workers, or an unusual spike in notification volume. The queue depth metric is the earliest warning signal — long before users notice delays.

### Notification System Improved Design — Component Map

The improved design (from the initial single-server sketch) adds five critical elements: authentication, rate limiting, notification templates, analytics/tracking, and the retry/DLQ path. This diagram maps every component to its responsibility:

```mermaid
graph TD
    subgraph Triggers["Triggering Services"]
        BS["Billing Service"]
        BookS["Booking Service"]
        MktS["Marketing Service"]
    end

    subgraph NS["Notification Servers (per request)"]
        Auth["1. Authenticate request\n(appKey + appSecret)"]
        Pref["2. Check user opt-in\n(notification settings table)"]
        Rate["3. Check rate limit\n(Redis counter with TTL)"]
        Tmpl["4. Apply template\n(render parameterized content)"]
        Enq["5. Enqueue to channel queue"]
    end

    subgraph Storage["Storage Layer"]
        Cache["Redis Cache\n(user info, device tokens, templates)"]
        DB["User DB\n(device tokens, settings)"]
        Log["Notification Log DB\n(status: pending → sent)"]
    end

    subgraph Queues["Per-Channel Queues"]
        iOSQ["iOS Queue"]
        DroidQ["Android Queue"]
        SMSQ["SMS Queue"]
        EmailQ["Email Queue"]
    end

    subgraph Workers["Workers (per channel)"]
        iOSW["iOS Workers\n(dedup check → APNs)"]
        DroidW["Android Workers\n(dedup check → FCM)"]
        SMSW["SMS Workers\n(dedup check → Twilio)"]
        EmailW["Email Workers\n(dedup check → SendGrid)"]
    end

    subgraph Reliability["Reliability"]
        Redis["Redis SET\n(processed event_ids)"]
        DLQ["Dead Letter Queue\n(exhausted retries)"]
        Alert["On-Call Alert"]
    end

    Triggers --> Auth
    Auth --> Pref
    Pref --> Rate
    Rate --> Tmpl
    Tmpl --> Enq
    Enq --> Queues
    NS <--> Cache
    NS <--> DB
    Queues --> Workers
    Workers --> Redis
    Workers --> Log
    Workers --> DLQ
    DLQ --> Alert
```

The key insight: notification servers are **stateless validators and routers**. They enforce business rules (auth, opt-in, rate limits) and then immediately hand off to a queue. Workers handle the stateful retry logic and provider calls.

## Interview Questions

- "Design a notification system that sends 10M push, 1M SMS, and 5M emails per day." → Clarify notification types and triggers. Walk through: triggering services → notification servers (validate, check opt-in, check rate limit, enqueue) → per-channel queues → channel-specific workers → providers (APNs, FCM, Twilio, SendGrid). Emphasize: separate queues per channel for isolation, at-least-once delivery with DB logging, dedup via event IDs, templates for consistency, exponential backoff retries, queue depth monitoring.

- "What happens if APNs is unavailable for 2 hours?" → Messages stay in the iOS queue (not lost). Workers retry with exponential backoff. If retry budget is exhausted, messages go to the DLQ. Alert fires to on-call. When APNs recovers, DLQ messages can be replayed. Monitor queue depth — if iOS queue grows while Android/SMS/Email queues are normal, the problem is APNs-specific.

- "How do you prevent sending the same notification twice?" → Assign a unique event ID to every notification event. Save to DB with status=pending. Before calling the provider, check if this event ID is in a Redis processed-events SET. If yes, skip. If no, send, then add to SET and update DB status=sent.

- "How do you handle a user with 10 devices?" → The device table stores all device tokens for a user. The notification server queries all tokens and enqueues one job per device. Workers process each independently. If APNs returns 410 Gone for any token, that token is deleted from the device table.

- "How would you implement scheduled notifications (send at 9am user's local time)?" → Store the target send time in the notification DB. Use a delayed queue (Amazon SQS delay queues support up to 15-minute delays) or a cron scheduler that queries for notifications due in the next minute and enqueues them. For longer delays, the cron approach is more flexible.

- "What operational metrics matter for a notification system?" → Queue depth per channel (growing = workers can't keep up), delivery success rate per provider, retry rate, DLQ message count, notification open rate (product metric), device token invalid rate (stale token cleanup effectiveness), and per-channel latency (time from enqueue to delivery).

## Related Chapters
- [Ch09 - Web Crawler](09-web-crawler.md) — both use message queues to decouple producers from consumers and rely on worker retry logic for reliability
- [Ch11 - News Feed](11-news-feed.md) — news feed uses push notifications for out-of-app updates; understanding this chapter informs how feeds reach offline users
- [Ch12 - Chat System](12-chat-system.md) — chat falls back to APNs/FCM push notifications for offline users, using the same provider infrastructure described here
- [Ch04 - Rate Limiter](04-rate-limiter.md) — the per-user rate limiting in this chapter uses the same Redis counter pattern described in Ch04
- [Ch01 - Scale from Zero to Millions](01-scale.md) — message queues for async decoupling and stateless web servers introduced here are the foundation for the notification server architecture
