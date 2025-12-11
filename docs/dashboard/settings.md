# Settings Module - Cấu hình hệ thống

## 1. Giới thiệu
Module Settings quản lý tất cả cấu hình của project, bao gồm account, practice info, services, channels, users, billing, subscriptions...

**Main Route:** `/settings`

**Nested Routes:** (19 sub-routes)

![Settings Dashboard](./images/image-16.png)

---

## 2. Nested Routes Structure

### 2.1 Account & Practice
| Route | Component | Mục đích |
|-------|-----------|----------|
| `/settings/account` | Account | Thông tin tài khoản cá nhân (profile, password, 2FA) |
| `/settings/practice` | Practice | Thông tin practice (name, address, logo, timezone, business hours) |

### 2.2 Subscription & Billing
| Route | Component | Mục đích |
|-------|-----------|----------|
| `/settings/subscription` | Subscription | Quản lý subscription plan, upgrade/downgrade |
| `/settings/billing` | Billing | Payment methods, billing history, invoices |

### 2.3 Services & Procedures
| Route | Component | Mục đích |
|-------|-----------|----------|
| `/settings/services` | Services | Danh sách services/procedures |
| `/settings/services/:id` | ServiceDetails | Chi tiết service (name, price, duration, category) |

### 2.4 Blogs
| Route | Component | Mục đích |
|-------|-----------|----------|
| `/settings/blogs` | Blogs | Quản lý blog posts (cho website/marketing) |
| `/settings/blogs/:id` | BlogDetails | Chi tiết blog post (title, content, SEO) |

### 2.5 Channels
| Route | Component | Mục đích |
|-------|-----------|----------|
| `/settings/channels` | Channels | Danh sách channels (SMS, Email, Voice, Fax, Social...) |
| `/settings/channels/webhook-url/` | ChannelAppWebhookUrlDetail | Webhook URL config |
| `/settings/channels/:channelApp/` | ChannelAppDetail | Chi tiết channel app (Jira, Slack, Zapier...) |
| `/settings/channels/:channelApp/:name` | ChannelDetail | Chi tiết channel cụ thể |

![Channels Dashboard](./images/image-17.png)

### 2.6 Notification
| Route | Component | Mục đích |
|-------|-----------|----------|
| `/settings/notification` | NotificationSetting | Cấu hình notifications (email, SMS, push) |

### 2.7 Admin & Users
| Route | Component | Mục đích |
|-------|-----------|----------|
| `/settings/admin` | Admin | Admin settings (locations, teams, roles) |
| `/settings/admin/users` | Admin (isUsers) | User management trong admin |
| `/settings/manage-users` | ManageUsers | Quản lý staffs (chỉ admin/owner) |
| `/settings/manage-users/staff/:id` | StaffDetails | Chi tiết staff (permissions, schedule, locations) |

### 2.8 Default Route
- Nếu không match route nào → Redirect to `/settings/practice`

---

## 3. API Endpoints (Tổng hợp)

### 3.1 Account
| Method | Endpoint | Mục đích |
|--------|----------|----------|
| GET | `/api/user` | Lấy thông tin user |
| PUT | `/api/user` | Cập nhật user profile |
| PUT | `/api/user/password` | Đổi password |
| POST | `/api/user/2fa/enable` | Bật 2FA |
| POST | `/api/user/2fa/disable` | Tắt 2FA |

### 3.2 Practice
| Method | Endpoint | Mục đích |
|--------|----------|----------|
| GET | `/api/project` | Lấy thông tin project |
| PUT | `/api/project` | Cập nhật project info |
| POST | `/api/project/logo` | Upload logo |

### 3.3 Services
| Method | Endpoint | Mục đích |
|--------|----------|----------|
| GET | `/api/service-categories` | Lấy categories |
| GET | `/api/procedures` | Lấy danh sách services |
| GET | `/api/procedures/:id` | Chi tiết service |
| POST | `/api/procedures` | Tạo service |
| PUT | `/api/procedures/:id` | Cập nhật service |
| DELETE | `/api/procedures/:id` | Xóa service |

### 3.4 Channels
| Method | Endpoint | Mục đích |
|--------|----------|----------|
| GET | `/api/channels` | Lấy danh sách channels |
| GET | `/api/channels/:id` | Chi tiết channel |
| POST | `/api/channels` | Tạo channel |
| PUT | `/api/channels/:id` | Cập nhật channel |
| DELETE | `/api/channels/:id` | Xóa channel |

### 3.5 Manage Users (Staffs)
| Method | Endpoint | Mục đích |
|--------|----------|----------|
| GET | `/api/staffs` | Lấy danh sách staffs |
| GET | `/api/staffs/:id` | Chi tiết staff |
| POST | `/api/staffs` | Tạo staff (invite) |
| PUT | `/api/staffs/:id` | Cập nhật staff |
| DELETE | `/api/staffs/:id` | Xóa staff |

### 3.6 Subscription
| Method | Endpoint | Mục đích |
|--------|----------|----------|
| GET | `/api/subscriptions` | Lấy subscription info |
| POST | `/api/subscriptions/upgrade` | Upgrade plan |
| POST | `/api/subscriptions/downgrade` | Downgrade plan |
| POST | `/api/subscriptions/cancel` | Cancel subscription |

### 3.7 Billing
| Method | Endpoint | Mục đích |
|--------|----------|----------|
| GET | `/api/payment-methods` | Lấy payment methods |
| POST | `/api/payment-methods` | Thêm payment method |
| DELETE | `/api/payment-methods/:id` | Xóa payment method |
| GET | `/api/invoices` | Lấy billing history |

---

## 4. Lưu ý kỹ thuật

### 4.1 Permissions
- **Staff role:** Không thể access `/settings/manage-users`, `/settings/billing`, `/settings/subscription`
- **Admin/Owner:** Full access tất cả settings
- Check `staffRole !== "staff"` trước khi render manage-users routes

### 4.2 Service Categories
- Fetch service categories khi mount Settings Main
- Store trong Redux: `setting.services.procedure.pageCategory`
- Pagination: `pageCategory` state
- Use cho cả Services settings và Widget service selection

### 4.3 Channel Apps
- **Channel Apps:** Jira, Slack, Zapier, Webhook, Google Calendar, Outlook...
- Route pattern: `/settings/channels/:channelApp/:name`
  - `:channelApp` - App type (jira, slack, zapier...)
  - `:name` - Channel name/ID
- Webhook URL: Special route `/settings/channels/webhook-url/`

### 4.4 Nested Routing
- Settings sử dụng nested `<Switch>` trong Main component
- Mỗi sub-route render component riêng
- Shared Sidebar component cho navigation
- ScrollBar component wrap content

### 4.5 Redux State
- `setting.services.procedure.serviceCategories` - Service categories
- `setting.services.procedure.pageCategory` - Current page
- `setting.services.procedure.totalPageCategory` - Total pages
- `setting.channels.widgetServiceCategories` - Widget service categories
- `authentication.user.staff.type` - Staff role (staff/admin/owner)

---

## 5. Component Structure

```
pages/apps/settings.tsx
└── containers/apps/settings/
    ├── sidebar/              # Settings navigation menu
    └── main/
        └── <Switch>          # Nested routes
            ├── Account
            ├── Practice
            ├── Services
            ├── ServiceDetails
            ├── Blogs
            ├── BlogDetails
            ├── Channels
            ├── ChannelDetail
            ├── ChannelAppDetail
            ├── ChannelAppWebhookUrlDetail
            ├── NotificationSetting
            ├── ManageUsers
            ├── StaffDetails
            ├── Admin
            ├── Subscription
            └── Billing
```

---

## 6. Settings Sidebar Menu

### 6.1 Menu Structure
```
Settings
├── Account
├── Practice Info
├── Services
├── Blogs
├── Channels
├── Notifications
├── Manage Users (Admin/Owner only)
├── Admin (Admin/Owner only)
├── Subscription
└── Billing (Admin/Owner only)
```

### 6.2 Conditional Menu Items
- **Manage Users:** Chỉ show nếu `staffRole !== "staff"`
- **Billing:** Chỉ show nếu `staffRole !== "staff"`
- **Admin:** Chỉ show nếu `staffRole !== "staff"`

---

## 7. Key Features

### 7.1 Account Settings
- Update profile (name, email, phone, avatar)
- Change password
- Enable/Disable 2FA
- Notification preferences

### 7.2 Practice Settings
- Practice name, address, phone
- Logo upload
- Timezone selection
- Business hours
- Multi-location setup (nếu có)

### 7.3 Services Management
- CRUD services/procedures
- Categorize services
- Set price, duration
- Attach to booking widget
- Service addons

### 7.4 Channels Management
- Configure communication channels:
  - SMS (Twilio)
  - Email (SMTP, Gmail, Outlook)
  - Voice (Twilio)
  - Fax (Twilio)
  - Facebook Messenger
  - Instagram DM
  - Chat Widget
- Integration apps:
  - Jira
  - Slack
  - Zapier
  - Google Calendar
  - Outlook Calendar
  - Webhook

### 7.5 Manage Users
- Invite staffs
- Set permissions
- Assign locations
- Set schedule/availability
- Deactivate/Delete users

---

