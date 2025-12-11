# Tickets Screen - Quản lý yêu cầu hỗ trợ

## 1. Giới thiệu
Màn hình Tickets là trung tâm quản lý tất cả các yêu cầu hỗ trợ/conversation từ khách hàng qua nhiều kênh (SMS, Email, Voice, Fax, Chat Widget...).

**Routes:**
- `/tickets` - Danh sách tất cả tickets
- `/tickets/:id` - Chi tiết ticket cụ thể

**Scope:** Xem, phân công, cập nhật trạng thái, trả lời tin nhắn, tạo meeting room, quản lý files.

---

![Tickets View](./images/image-13.png)

## 2. Yêu cầu chức năng

### 2.1 Hiển thị Tickets
- Danh sách tickets theo filter:
  - **Status:** `open`, `assigned`, `closed`, `mentioned`
  - **Type:** `sms`, `email`, `voice`, `fax`, `chat`, `form`
  - **Staff:** Filter theo staff được assign
  - **Channel:** Filter theo kênh giao tiếp
  - **Keyword:** Tìm kiếm theo subject, description, lead name
- Pagination: 25 tickets/page (default)
- Sort theo: `created_at`, `updated_at`, `priority`
- Location filter (nếu project bật multi-location)

### 2.2 Chi tiết Ticket
- Hiển thị thông tin ticket:
  - Subject, description, status, type
  - Lead information (name, phone, email, address)
  - Assigned staff
  - Messages/conversation history
  - Attached files
  - Related events/appointments
- Real-time updates qua WebSocket

![Ticket Detail View](./images/image-9.png)

### 2.3 Quản lý Ticket
- **Assign staff:** Phân công ticket cho staff
- **Update status:** Chuyển trạng thái `open` ↔ `assigned` ↔ `closed`
- **Mark as read/unread:** Đánh dấu đã đọc
- **Bulk actions:**
  - Bulk update status
  - Bulk assign staff
  - Bulk delete
![Ticket Filters](./images/image-10.png)

### 2.4 Trả lời Ticket
- Gửi tin nhắn qua các kênh:
  - SMS
  - Email
  - Voice call
  - Fax
- Attach files (images, documents, videos)
- Mention staff (@mention)
- Create tasks từ ticket

### 2.5 Meeting Room
- Tạo video meeting room cho ticket
- Join meeting từ ticket detail
- End meeting và lưu lại history

### 2.6 Statistics
- Tổng số tickets theo status
- Tickets theo type (SMS, Email, Voice...)
- Tickets theo staff
- Tickets theo location

---

## 3. API Endpoints

| Method | Endpoint | Mục đích | Params | Response |
|--------|----------|----------|--------|----------|
| GET | `/api/tickets` | Lấy danh sách tickets | `page`, `limit`, `status?`, `type?`, `staff_id?`, `staff_ids?`, `keyword?`, `channel_id?`, `sortBy?`, `sortOrder?` | `CommonPagination<Ticket>` |
| GET | `/api/tickets/:id` | Lấy chi tiết ticket | `id` | `Ticket` |
| PUT | `/api/tickets/:id` | Cập nhật ticket | `id`, `staff_id?`, `status?`, `read?` | `Ticket` |
| PUT | `/api/tickets/bulk` | Cập nhật nhiều tickets | `ids[]`, `status`, `staff_id?` | `unknown` |
| DELETE | `/api/tickets/bulk` | Xóa nhiều tickets | `ids[]` | `unknown` |
| GET | `/api/tickets/statistics` | Lấy thống kê tickets | `location_id?` | `TicketStatisticResponse` |
| GET | `/api/tickets/type/count` | Đếm tickets theo type | `staff_ids[]`, `status`, `location_id?` | `CountOfTicket` |
| POST | `/api/videos/room` | Tạo meeting room cho ticket | `phone?`, `lead_id` | `DailyRoomResponse` |
| POST | `/api/videos/meeting` | Tạo meeting room mới | - | `MeetingRoomResponse` |
| POST | `/api/leads/:id` | Cập nhật lead info | `id`, `phone` | `Lead` |
| GET | `/api/files/:id/download` | Download file attachment | `id` | `Blob` |

### 3.1 GetTicketsApi Params
```typescript
{
  page: number;               // Page number (default: 1)
  limit?: number;             // Items per page (default: 25)
  status?: TicketStatus;      // 'open' | 'closed' | 'pending' | 'resolved'
  staff_id?: number;          // Filter by single staff
  staff_ids?: number[];       // Filter by multiple staffs
  keyword?: string;           // Search keyword
  type?: string;              // 'sms' | 'email' | 'voice' | 'fax' | 'chat' | 'form'
  channel_id?: number;        // Filter by channel
  sortBy?: string;            // 'created_at' | 'updated_at' | 'priority'
  sortOrder?: string;         // 'asc' | 'desc'
}
```

### 3.2 UpdateTicketApi Body
```typescript
{
  staff_id?: number | null;   // Assign to staff (null to unassign)
  status?: string;            // Update status
  read?: boolean;             // Mark as read/unread
}
```

---

## 4. WebSocket Events

| Event Type | Trigger | Action |
|------------|---------|--------|
| `NEW_TICKET` | Ticket mới được tạo | Invalidate `['fetchTickets']`, show notification, update statistics |
| `TICKET_UPDATED` | Ticket được cập nhật | Invalidate `['fetchTickets']`, `['tickets', 'ticket-details']`, refresh UI |
| `NEW_MESSAGE` | Tin nhắn mới trong ticket | Invalidate `['messages']`, update conversation |
| `MESSAGE_UPDATED` | Tin nhắn được cập nhật | Invalidate `['messages']`, refresh conversation |

---

## 5. Redux State

### 5.1 Ticket State
- `ticket.list.getTicketsParams` - Params hiện tại cho fetch tickets
- `ticket.statistic` - Thống kê tickets
- `ticket.daily.room` - Meeting room hiện tại
- `ticket.newMeeting.room` - New meeting room
- `ticket.channel.data` - Danh sách channels

### 5.2 Actions
- `start()` - Bắt đầu fetch statistics
- `getCountOfTicket()` - Lấy count tickets
- `clearRoom()` - Clear meeting room
- `clearMeetingRoom()` - Clear new meeting room
- `doCheckMeetingRoom()` - Check meeting room status

---

## 6. Lưu ý kỹ thuật

### 6.1 Real-time Updates
- WebSocket connection cho real-time ticket updates
- Auto refresh khi có ticket mới (nếu không ở trang tickets)
- Browser notification cho ticket mới (nếu user cho phép)
- Title blinking khi có ticket mới (tab không active)

### 6.2 Location Filter
- Chỉ hiện tickets thuộc location đang chọn
- Logic filter:
  ```javascript
  const isLocationFilterEnabled = user?.project.dashboard_location_filter;
  const isMatchingLocation = 
    ticket.location_id === Number(localStorage.getItem('location_id')) ||
    Number(localStorage.getItem('location_id')) === -1;
  ```

### 6.3 Notification Logic
```javascript
// Chỉ show notification nếu:
// 1. Không ở trang tickets
// 2. Location filter disabled HOẶC ticket thuộc location đang chọn
if (!isOnTicketsPage && (!isLocationFilterEnabled || isMatchingLocation)) {
  handleNotification(ticket);
}
```

### 6.4 Pagination & Sorting
- Default: 25 items/page
- Sort params format: `sort[by]=created_at&sort[order]=desc`
- Filter params format: `in[status]=open,pending`

### 6.5 Bulk Operations
- Select multiple tickets
- Bulk update status: `PUT /api/tickets/bulk`
- Bulk delete: `DELETE /api/tickets/bulk`
- Confirm dialog trước khi bulk delete

---

## 7. Component Structure

```
pages/apps/tickets.tsx
└── containers/apps/ticket/
    ├── wrapper/
    ├── sidebar/          # Ticket list với filters
    ├── group/            # Ticket groups/categories
    └── main/             # Ticket detail & conversation
        ├── ticket-header/
        ├── ticket-body/
        │   ├── chat-group/      # Messages
        │   ├── lead-info/       # Lead details
        │   └── files/           # Attachments
        └── chat-form/           # Reply form
```

---

## 8. Dependencies
- `react-query` / `@tanstack/react-query` - Data fetching
- `react-toastify` - Toast notifications
- WebSocket - Real-time updates
- `@twilio/voice-sdk` - Voice calls (nếu có)

---

## 9. Permissions
- **Staff role:**
  - Xem tickets được assign
  - Reply tickets
  - Không thể delete tickets
- **Admin/Owner role:**
  - Xem tất cả tickets
  - Assign/reassign tickets
  - Bulk operations
  - Delete tickets

---

