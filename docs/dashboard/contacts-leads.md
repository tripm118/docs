# Contacts & Leads Module

## 1. Giới thiệu

![Contacts & Leads Module](./images/image-20.png)

### 1.1 Contacts Screen
Quản lý tất cả contacts/leads trong hệ thống.

**Route:** `/contacts`

**Scope:** Xem, tạo, sửa, xóa contacts, import/export, merge duplicates.

### 1.2 Leads Screen
Chi tiết lead cụ thể với full information và activity timeline.

**Route:** `/leads/:id`

**Scope:** Xem chi tiết lead, edit info, view timeline, create tickets/events/tasks.

---

## 2. Contacts Screen

### 2.1 Yêu cầu chức năng
- **Danh sách contacts:**
  - Grid/List view
  - Filter theo: location, tags, status, source
  - Search theo: name, email, phone
  - Sort theo: name, created_at, updated_at
  - Pagination
- **CRUD Operations:**
  - Create contact (manual hoặc import CSV)
  - Update contact info
  - Delete contact (single/bulk)
  - Merge duplicate contacts
- **Advanced Features:**
  - Import contacts (CSV)
  - Export contacts (CSV/Excel)
  - Bulk operations (tag, delete, assign)
  - Custom fields

### 2.2 API Endpoints
| Method | Endpoint | Mục đích | Params |
|--------|----------|----------|--------|
| GET | `/api/leads` | Lấy danh sách leads | `page`, `limit`, `keyword?`, `location_id?`, `tags?`, `status?`, `source?` |
| GET | `/api/leads/:id` | Chi tiết lead | `id` |
| POST | `/api/leads` | Tạo lead | `LeadForm` |
| PUT | `/api/leads/:id` | Cập nhật lead | `id`, `LeadForm` |
| DELETE | `/api/leads/:id` | Xóa lead | `id` |
| POST | `/api/leads/import` | Import leads | `FormData` (CSV file) |
| GET | `/api/leads/export` | Export leads | `format?` (csv/excel) |
| POST | `/api/leads/merge` | Merge duplicate leads | `primary_id`, `secondary_ids[]` |
| PUT | `/api/leads/bulk` | Bulk update | `ids[]`, `action`, `data` |

### 2.3 LeadForm
```typescript
{
  first_name: string;
  last_name?: string;
  email?: string;
  phone?: string;
  address?: string;
  city?: string;
  state?: string;
  zip?: string;
  country?: string;
  location_id?: number;
  tags?: string[];
  status?: string;           // 'active' | 'inactive' | 'archived'
  source?: string;           // 'website' | 'referral' | 'social' | 'manual'
  custom_fields?: Record<string, any>;
}
```

---

## 3. Leads Detail Screen

### 3.1 Yêu cầu chức năng
- **Lead Information:**
  - Personal info (name, email, phone, address)
  - Custom fields
  - Tags
  - Status
  - Source
  - Location
- **Activity Timeline:**
  - Tickets/Conversations
  - Events/Appointments
  - Tasks
  - Calls
  - Messages (SMS, Email)
  - Faxes
  - Notes
  - Files
- **Quick Actions:**
  - Call lead
  - Send SMS
  - Send Email
  - Create ticket
  - Create event
  - Create task
  - Upload file

### 3.2 API Endpoints
| Method | Endpoint | Mục đích |
|--------|----------|----------|
| GET | `/api/leads/:id` | Lấy lead info | 
| GET | `/api/leads/:id/timeline` | Lấy activity timeline |
| POST | `/api/leads/:id/notes` | Tạo note |
| GET | `/api/leads/:id/files` | Lấy files |
| POST | `/api/leads/:id/files` | Upload file |
| GET | `/api/leads/:id/tags` | Lấy tags |
| POST | `/api/leads/:id/tags` | Add tags |
| DELETE | `/api/leads/:id/tags/:tagId` | Remove tag |

### 3.3 Timeline Items
```typescript
TimelineItem {
  type: 'ticket' | 'event' | 'task' | 'call' | 'message' | 'fax' | 'note' | 'file';
  id: number;
  title: string;
  description?: string;
  timestamp: Date;
  staff?: Staff;
  metadata?: Record<string, any>;
}
```

---

## 4. WebSocket Events

| Event Type | Trigger | Action |
|------------|---------|--------|
| `LEAD_UPDATED` | Lead info thay đổi | Invalidate lead query, refresh UI |
| `LEAD_ACTIVITY` | Activity mới (ticket, event, call...) | Invalidate timeline, update UI |

---

## 5. Redux State

### 5.1 Contact State
- `contact.lead.lead` - Current lead
- `contact.lead.loading` - Loading state
- `contact.list` - Contacts list
- `contact.filters` - Current filters

### 5.2 Actions
- `doSetLoadingLead()` - Set loading
- `doFetchLead()` - Fetch lead detail
- `doUpdateLead()` - Update lead
- `doDeleteLead()` - Delete lead

---

## 6. Lưu ý kỹ thuật

### 6.1 Import Contacts
- Support CSV format
- Required columns: `first_name`, `email` hoặc `phone`
- Optional columns: `last_name`, `address`, `city`, `state`, `zip`, `tags`
- Validate data trước khi import
- Show preview trước khi confirm
- Handle duplicates (skip/update/merge)

### 6.2 Export Contacts
- Export to CSV hoặc Excel
- Include all fields + custom fields
- Filter/search applied trước khi export
- Download file trực tiếp

### 6.3 Merge Duplicates
- Detect duplicates by: email, phone, name
- Select primary lead (keep)
- Select secondary leads (merge into primary)
- Merge logic:
  - Keep primary lead info
  - Merge timeline/activities từ secondary leads
  - Update references (tickets, events, tasks...)
  - Delete secondary leads

### 6.4 Custom Fields
- Dynamic custom fields per project
- Support types: text, number, date, dropdown, checkbox
- Store trong `custom_fields` JSON column
- Render dynamic form fields

### 6.5 Location Filter
- Chỉ show nếu `project.dashboard_location_filter === true`
- Filter leads theo location
- Default: all locations (`location_id = -1`)

---

## 7. Component Structure

```
pages/apps/contacts.tsx
└── containers/apps/contacts/
    ├── wrapper/
    ├── sidebar/              # Filters, search
    ├── main/                 # Contacts grid/list
    └── modals/
        ├── create-contact/
        ├── edit-contact/
        ├── import-contacts/
        └── merge-contacts/

pages/apps/leads.tsx
└── containers/apps/leads/
    ├── header/               # Lead info, quick actions
    ├── timeline/             # Activity timeline
    └── sidebar/              # Lead details, tags, files
```

---

## 8. Permissions
- **All roles:** Có thể view contacts
- **Staff:** Chỉ view contacts được assign hoặc trong tickets
- **Admin/Owner:** View tất cả contacts, import/export, merge

---

## 9. Subscription Limits
- **Contact Limit:** Theo subscription plan
- Show modal upgrade khi reach limit
- Check `subscription.quantity` vs current contact count
- Prevent create/import khi over limit

---

