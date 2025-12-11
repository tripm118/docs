# Workflows Module - Automation Rules

## 1. Giới thiệu
Module Workflows cho phép tạo automation rules để tự động hóa các tác vụ marketing và communication (send SMS/Email, track engagement).

**Route:** `/workflows`

**Scope:** Tạo, sửa, xóa workflows, view execution metrics, enable/disable workflows.

**Lưu ý:** Workflows trong hệ thống này là **simple automation rules**, KHÔNG phải visual workflow builder phức tạp.

---

## 2. Yêu cầu chức năng

### 2.1 Workflow Management
- **CRUD Workflows:**
  - Create workflow
  - Edit workflow
  - Delete workflow
  - Enable/Disable workflow (`active: true/false`)
- **Workflow Configuration:**
  - Set workflow name
  - Select trigger
  - Configure actions (payload)
  - Set active status

![Workflows List](./images/image-18.png)

### 2.2 Workflow Metrics
- **Execution Tracking:**
  - Email sent count
  - SMS sent count
  - Email opened count
- **Performance Monitoring:**
  - Track workflow effectiveness
  - View engagement metrics

---

## 3. API Endpoints

**Lưu ý:** API endpoints cần verify thêm từ actual API files. Dưới đây là endpoints dự kiến dựa trên pattern chung:

| Method | Endpoint | Mục đích | Params | Response |
|--------|----------|----------|--------|----------|
| GET | `/api/workflows` | Lấy danh sách workflows | `page?`, `limit?`, `active?` | `CommonPagination<WorkflowResponse>` |
| GET | `/api/workflows/:id` | Chi tiết workflow | `id` | `WorkflowResponse` |
| POST | `/api/workflows` | Tạo workflow | `WorkflowData` | `WorkflowResponse` |
| PUT | `/api/workflows/:id` | Cập nhật workflow | `id`, `WorkflowData` | `WorkflowResponse` |
| DELETE | `/api/workflows/:id` | Xóa workflow | `id` | `void` |
| PUT | `/api/workflows/:id/toggle` | Enable/Disable | `id`, `active: boolean` | `WorkflowResponse` |

### 3.1 WorkflowResponse (từ source)
```typescript
{
  id: number;
  name: string;
  project_id: number;
  workflow_trigger_id: number;
  active: boolean;
  payload?: any;                // Workflow configuration (structure not defined)
  uuid?: any;
  created_at: Date;
  updated_at: Date;
  
  // Metrics
  email_sent_count?: number;
  sms_sent_count?: number;
  email_opened_count?: number;
}
```

### 3.2 WorkflowData
```typescript
{
  name: string;
  workflow_trigger_id: number;
  active: boolean;
  payload?: {
    // Workflow configuration
    // Structure depends on trigger type
    // Cần verify từ actual workflow builder component
  };
}
```

---

## 4. Workflow Structure

### 4.1 Workflow Components
Dựa trên type definition, workflow có các thành phần:
- **Name:** Tên workflow
- **Trigger:** Trigger ID (reference to workflow_trigger table)
- **Payload:** Configuration data (structure không rõ từ types)
- **Active:** Enable/disable status

### 4.2 Metrics Tracking
Workflows track 3 metrics chính:
- **email_sent_count:** Số email đã gửi
- **sms_sent_count:** Số SMS đã gửi
- **email_opened_count:** Số email đã mở

---

## 5. Lưu ý kỹ thuật

### 5.1 Workflow Payload
- `payload` field có type `any` - structure không được define trong types
- Có thể chứa:
  - Trigger conditions
  - Action configurations
  - Timing rules
  - Target audience filters
- **Cần verify:** Check workflow builder component để biết chính xác structure

### 5.2 Trigger System
- Workflows sử dụng `workflow_trigger_id`
- Trigger types không được define trong types
- **Cần verify:** Check workflow_trigger table/API để biết trigger types

### 5.3 Active Status
- `active: true` → Workflow đang chạy
- `active: false` → Workflow tạm dừng
- Toggle via API: `PUT /api/workflows/:id/toggle`

### 5.4 Metrics Collection
- Metrics được track tự động khi workflow execute
- Email opened tracking: Có thể dùng tracking pixel
- SMS sent tracking: Track qua Twilio webhook

---

## 6. Reputation/Review Workflows

**Bonus:** Hệ thống có thêm `ReputationResponse` type cho review workflows:

```typescript
interface ReputationResponse {
  id: number;
  link: string;                 // Review link
  project_id: number;
  location_id: number;
  trigger: string;              // Trigger type
  type: string;                 // Reputation type
  wait_in_minutes: number;      // Delay before sending
  stage_id?: number;            // Lead stage trigger
  created_at: Date;
  updated_at: Date;
  procedures: Procedure[];      // Filter by procedures
  locations: Location[];        // Filter by locations
  project_review_link: string;
}
```

**Use case:** Tự động gửi review request sau khi:
- Event completed
- Ticket closed
- Specific lead stage reached
- Specific procedure completed

---

## 7. Component Structure

```
pages/apps/workflows.tsx
└── containers/apps/workflows/
    ├── list/                 # Workflows list
    ├── create/               # Create workflow
    ├── edit/                 # Edit workflow
    └── metrics/              # Workflow metrics dashboard
```

**Lưu ý:** Cần check actual component structure để verify.

---

## 8. Permissions
- **All roles:** Có thể view workflows
- **Admin/Owner:** Create/edit/delete workflows

---