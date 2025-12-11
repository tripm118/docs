# PilotPractice API - Development Guide

## 1. Giới thiệu
PilotPractice API là Laravel backend cho hệ thống quản lý lead, appointment, messaging, và workflow automation.

**Tech Stack:**
- Laravel Framework
- MySQL Database
- Redis Queue
- Twilio Integration (SMS/Voice)
- Daily.co (Video calls)
- Snooze Package (Scheduled notifications)

---

## 2. Cấu trúc Project

### 2.1 Directories chính
```
app/
├── Actions/          # Business logic actions
├── Ai/              # AI-related features
├── Channels/        # Broadcasting channels
├── Clients/         # External API clients
├── Console/         # Artisan commands & schedulers
├── Events/          # Domain events
├── Jobs/            # Queue jobs
├── Listeners/       # Event listeners
├── Models/          # Eloquent models
├── Notifications/   # Email/SMS notifications
├── Http/
│   ├── Controllers/ # API controllers
│   └── Middleware/  # Custom middleware
└── Providers/       # Service providers
```

---

## 3. Queue System

### 3.1 Queue Names
Project sử dụng nhiều queue khác nhau:

| Queue Name | Mục đích | Command |
|------------|----------|---------|
| `default` | General jobs | `php artisan queue:work` |
| `sync` | Sync jobs (DrChrono, Zenoti, etc.) | `php artisan queue:work --queue=sync` |
| `messages` | SMS/messaging jobs | `php artisan queue:work --queue=messages` |
| `notifications` | Email/notification jobs | `php artisan queue:work --queue=notifications` |
| `workflows` | Workflow automation | `php artisan queue:work --queue=workflows` |
| `image_compression` | Image/video processing | `php artisan queue:work --queue=image_compression` |
| `qa_automation` | QA & testing jobs | `php artisan queue:work --queue=qa_automation` |

### 3.2 Chạy Queue Workers
```bash
# Run all queues
php artisan queue:work

# Run specific queue
php artisan queue:work --queue=sync

# Run multiple queues (priority order)
php artisan queue:work --queue=default,sync,notifications
```

---

## 4. Jobs

### 4.1 Jobs ví dụ

**Sync Jobs:**
- `SyncEventWithDrChrono` - Sync appointments với DrChrono
- `ZenotiSyncJob` - Sync với Zenoti integration
- `SyncGoogleCalendarEventJob` - Sync Google Calendar

**Messaging Jobs:**
- `SendSmsMessageJob` - Gửi SMS qua Twilio
- `CreateLeadFromInboundMessageJob` - Tạo lead từ tin nhắn

**Notification Jobs:**
- `SendEmailReportJob` - Gửi email reports
- `SendMissedCallAlertJob` - Alert missed calls
- `SendReviewFollowUpJob` - Review follow-up emails

**Processing Jobs:**
- `ImportLeadsJob` - Import leads từ CSV
- `ExportLeadToCSVJob` - Export leads
- `ProcessImageOptimizationJob` - Optimize images

### 4.2 Queue Configuration
Một số jobs có custom queue:
```php
// Example: SyncEventWithDrChrono.php
$this->onQueue(Queue::SYNC);

// Example: SendSmsMessageJob.php
$this->onQueue(Queue::MESSAGES);
```

---

## 5. Events & Listeners

### 5.1 Events
```
app/Events/
├── FileGeneratedSuccessfullyEvent.php
├── Forms/FormResponseSubmitted.php
├── Integration/ConnectionComplete.php
├── LeadsMergedEvent.php
├── Visitors/VisitorMatched.php
└── Workflow/WorkflowStarted.php
```

### 5.2 Listeners
```
app/Listeners/
└── LeadsMergedListener.php
```

**⚠️ LƯU Ý QUAN TRỌNG:**
- **Nhiều logic xử lý trong listeners** - Cần kiểm tra kỹ trigger events của modal/form
- Listeners có thể dispatch thêm jobs hoặc events khác
- Kiểm tra `EventServiceProvider` để xem event-listener mapping

---

## 6. Notifications

### 6.1 Notification Classes
```
app/Notifications/
├── CreateRoomNotification.php
├── FileRequestLinkNotification.php
├── FileRequestReminderNotification.php
├── MissedCallReply.php
├── SendLinkReviewNotification.php
└── StaffMentionedNotification.php
```

### 6.2 Chạy Notification Queue
```bash
# Run notifications queue
php artisan queue:work --queue=notifications
```

**⚠️ LƯU Ý:**
- Email notifications chạy qua queue `notifications`
- Một số notifications có scheduling qua Snooze package

---

## 7. Scheduled Commands

### 7.1 Snooze Package
Package `thomasjohnkane/snooze` để schedule notifications.

**Config:** `config/snooze.php`
- Frequency: `everySecond` (configurable)
- Auto-schedule: enabled

**Command:**
```bash
# Send scheduled notifications (auto-run by scheduler)
php artisan snooze:send

# Prune old notifications
php artisan snooze:prune
```

### 7.2 Scheduled Tasks
Xem `app/Console/Kernel.php` để biết tất cả scheduled commands:

| Command | Schedule | Mục đích |
|---------|----------|----------|
| `snooze:send` | Every 5 seconds | Send scheduled notifications |
| `queue:work` | - | Process queue jobs |
| `events:mark-complete` | Every minute | Mark completed events |
| `integrations:modmed-sync` | Every 5 minutes | Sync ModMed appointments |
| `zenoti:sync` | Daily 3:00 AM | Sync Zenoti appointments |
| `drchrono:sync` | Daily 12:00 AM | Sync DrChrono appointments |

---

## 8. API Routes

### 8.1 Main Route Files
```
routes/
├── api.php          # Main API routes (authenticated)
├── auth-api.php     # Auth endpoints
├── webhooks.php     # Webhook endpoints (Twilio, etc.)
├── public.php       # Public endpoints (widget, forms)
└── portal.php       # Client portal routes
```

### 8.2 API Structure
- Base URL: `/api/*`
- Authentication: Laravel Sanctum
- Middleware: `auth:sanctum`, `tokenCan:*`

**Example Routes:**
```php
GET    /api/leads              # List leads
POST   /api/leads              # Create lead
GET    /api/leads/{id}         # Show lead
PUT    /api/leads/{id}         # Update lead
DELETE /api/leads/{id}         # Delete lead

GET    /api/calls              # List calls
POST   /api/messages           # Send message
GET    /api/events             # List appointments
```

---

## 9. Integrations

### 9.1 Third-party Services
- **Twilio** - SMS, Voice calls
- **Daily.co** - Video meetings
- **Google Calendar** - Calendar sync
- **DrChrono** - EHR integration
- **Zenoti** - Spa/salon software
- **MindBody** - Wellness software
- **Stripe** - Payments
- **Meta/Facebook** - Lead ads, messaging
- **Jira** - Ticket management

### 9.2 Sync Jobs
Một vài integration có sync jobs chạy theo schedule:
```bash
# DrChrono: Daily 12:00 AM
# Zenoti: Daily 3:00 AM
# MindBody: Daily 3:10 AM
# SimplePractice: Daily 3:40 AM
```

### 9.3 Integration Flow
Mục đích của integration là sync dữ liệu từ các hệ thống bên ngoài vào hệ thống của chúng ta.

**Ví dụ flow:**
- Khi create event ở app → đồng thời call API đến app bên ngoài để tạo event
- Để tạo được event cần biết API requirements: `title`, `start_time`, `end_time`, `staff`, `lead`, `type`, `status`, `location`
- Mapping fields giữa app chúng ta với app bên ngoài thông qua `vendor_id`

---

## 10. Development Workflow

### 10.1 Running Services
```bash
# Start queue workers (multiple terminals)
php artisan queue:work --queue=default,sync,notifications,messages

# Start scheduler (in production use cron)
php artisan schedule:work
```

---

## 11. Lưu Ý Quan Trọng

### 11.1 Queue Jobs
**⚠️ QUAN TRỌNG:**
- **Nhiều logic xử lý trong jobs** - Cần run queue workers để xử lý
- Một số jobs có queue `sync` - Cần chạy: `php artisan queue:work --queue=default,sync`
- Jobs có thể fail và retry - Check `failed_jobs` table

### 11.2 Event Listeners
**⚠️ QUAN TRỌNG:**
- **Nhiều logic xử lý trong listeners** - Cần kiểm tra trigger events của modal/form
- Listeners có thể dispatch jobs hoặc fire events khác
- Check `EventServiceProvider` để xem mappings

### 11.3 Notifications
**⚠️ QUAN TRỌNG:**
- **Email notifications** - Chạy queue: `php artisan queue:work --queue=notifications`
- **Scheduled notifications** - Chạy: `php artisan snooze:send` (auto-scheduled)
- Một số send mail với schedule - Cần run command `snooze:send`

### 11.4 Database
- Sử dụng soft deletes cho nhiều models
- Custom fields system (package `givebutter/laravel-custom-fields`)
- Audit logging enabled

---

## 12. Debugging

### 12.1 Queue Debugging
```bash
# Check failed jobs
php artisan queue:failed

# Retry failed job
php artisan queue:retry {id}

# Retry all failed
php artisan queue:retry all

# Clear failed jobs
php artisan queue:flush
```

### 12.2 Logs
```bash
# Laravel logs
storage/logs/laravel.log

# Queue logs
Check queue worker output
```

### 12.3 Common Issues
1. **Jobs not processing** → Check queue workers running
2. **Emails not sending** → Check `notifications` queue worker
3. **Sync not working** → Check `sync` queue worker
4. **Scheduled tasks not running** → Check cron/scheduler

---
