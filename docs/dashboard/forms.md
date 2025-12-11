# Forms Module - PDF-based Templates với E-Signature

## 1. Giới thiệu
Module Forms quản lý PDF-based templates với e-signature support. Forms được tạo từ PDF templates, thêm custom fields overlay, và gửi cho leads/staffs để điền và ký.

![Forms Module](./images/image-21.png)

**Routes:**
- `/forms` - Danh sách templates và forms
- `/forms/:id` - Template editor hoặc form viewer
- `/client/portal/:uuidProject/forms/:id` - Form trong client portal
- `/:projectId/template/:id` - Public form template
- `/forms/sign-complete` - Trang hoàn thành ký

**Scope:** Tạo PDF templates, add custom fields, send forms, collect signatures, view submissions.

---

## 2. Yêu cầu chức năng

### 2.1 Template Management
- **CRUD Templates:**
  - Create template (upload PDF hoặc create blank)
  - Edit template (add/remove pages, add fields)
  - Delete template
  - Duplicate template
  - Mark as favorite
- **Template Types:**
  - `pdf` - PDF document
  - `form` - Form template
  - `document` - General document
  - `email` - Email template
  - `sms` - SMS template
  - `note` - Note template
  - `quick_reply` - Quick reply template

![Upload PDF Template](./images/image-6.png)

### 2.2 Template Editor
- **Upload PDF Pages:**
  - Upload PDF file
  - Convert to pages
  - Add/remove pages
  - Reorder pages
- **Add Custom Fields:**
  - Drag-drop fields onto PDF pages
  - Position fields (x, y coordinates)
  - Resize fields (width, height)
  - Field types (xem section 3.2)

![Field Configuration](./images/image-7.png)

- **Field Configuration:**
  - Set required/optional
  - Set default value
  - Set placeholder
  - Assign to signatory (lead/staff)
  - Set field color
  - Set font size

### 2.3 Form Distribution
- **Send to Leads:**
  - Create form from template
  - Assign signatories (lead + staff nếu cần)
  - Set due date
  - Send via email/SMS
  - Client portal access
- **Public Forms:**
  - Public link (không cần login)
  - Require authentication option
  - Signature verification

### 2.4 Form Filling & Signing
- **Fill Form:**
  - Lead fills custom fields
  - Save progress (draft)
  - Multi-page forms
- **E-Signature Flow:**
  1. Lead fills all required fields
  2. Lead signs (signature field)
  3. If require staff signature → staff signs
  4. Form status = completed
  5. Generate signed PDF

![E-Signature Pad](./images/image-8.png)

- **Signature Types:**
  - `signature` - Full signature
  - `initials` - Initials only

### 2.5 Form Submissions
- **View Submissions:**
  - List all forms
  - Filter by: status, signing_status, lead, template
  - View filled form (PDF with responses)
  - Download signed PDF
- **Form Status:**
  - `draft` - Chưa hoàn thành
  - `completed` - Đã điền xong
  - (Status values cần verify thêm)
- **Signing Status:**
  - `pending_lead_signature` - Chờ lead ký
  - `pending_staff_signature` - Chờ staff ký
  - `signed` - Đã ký đầy đủ

---

## 3. API Endpoints

| Method | Endpoint | Mục đích | Params | Response |
|--------|----------|----------|--------|----------|
| GET | `/api/templates` | Lấy danh sách templates | `name?`, `type?`, `types?`, `keyword?`, `status?`, `page?`, `limit?` | `CommonPagination<TemplateInfo>` |
| GET | `/api/templates/:id` | Chi tiết template | `id` | `Template` |
| POST | `/api/templates` | Tạo template | `CreateNewTemplateRequestParams` | `Template` |
| PUT | `/api/templates/:id` | Cập nhật template | `id`, `UpdateTemplateRequest` | `Template` |
| DELETE | `/api/templates/:id` | Xóa template | `id` | `void` |
| POST | `/api/templates/:id/duplicate` | Duplicate template | `id` | `Template` |
| POST | `/api/templates/:id/page` | Upload PDF page | `id`, `FormData` (PDF file) | `Template` |
| PUT | `/api/templates/:id/page/:pageId` | Update page | `id`, `pageId`, `contents`, `width`, `height` | `void` |
| DELETE | `/api/templates/:id/page/:pageId` | Delete page | `id`, `pageId` | `void` |
| POST | `/api/templates/:id/page/:pageId/field` | Add custom field | `id`, `pageId`, `CreateNewCustomFieldParams` | `CustomField` |
| PUT | `/api/templates/:id/page/:pageId/field/:fieldId` | Update field | `id`, `pageId`, `fieldId`, `UpdateCustomFieldParams` | `CustomField` |
| DELETE | `/api/templates/:id/page/:pageId/field/:fieldId` | Delete field | `id`, `pageId`, `fieldId` | `void` |
| POST | `/api/templates/:id/page/:pageId/field/copy` | Copy field | `id`, `pageId`, `CopyCustomFieldParams` | `CustomField` |
| PUT | `/api/templates/:id/page/:pageId/reorder` | Reorder fields | `id`, `pageId`, `fields[]` | `Template` |
| GET | `/api/forms/:id` | Lấy form | `id` | `Forms` |
| PUT | `/api/forms/:id/page/:pageId/response` | Update form response | `id`, `pageId`, `custom_fields` | `Forms` |
| PUT | `/api/forms/:id/staff/sign` | Staff sign form | `id` | `void` |
| GET | `/portal/forms/:id` | Lead view form | `id` | `Forms` |
| PUT | `/portal/forms/:id/page/:pageId/response` | Lead fill form | `id`, `pageId`, `custom_fields` | `Forms` |
| PUT | `/portal/forms/:id/sign` | Lead sign form | `id` | `void` |
| POST | `/api/template/:id/submitting` | Submit public form | `template_id`, `visitor_id`, `data[]` | `void` |
| GET | `/api/template/:id/download` | Download template PDF | `id` | `TemplatePdf[]` |

### 3.1 CreateNewTemplateRequestParams
```typescript
{
  name: string;
  type: 'pdf' | 'form' | 'document' | 'email' | 'sms' | 'note' | 'quick_reply' | string;
  subject?: string;
  content: any;
  custom_fields: CustomField[];
  status: 'draft' | string;
  template_ids?: number[];      // For batch templates
}
```

### 3.2 Custom Field Types (TypeCustomField)
```typescript
type TypeCustomField =
  | "textarea"          // Multi-line text
  | "checkbox"          // Checkbox
  | "dropdown"          // Dropdown select
  | "signature"         // Signature field
  | "initials"          // Initials field
  | "date"              // Date picker
  | "firstname"         // First name (auto-fill from lead)
  | "lastname"          // Last name (auto-fill from lead)
  | "emailaddress"      // Email (auto-fill from lead)
  | "phonenumber"       // Phone (auto-fill from lead)
  | "dateofbirth"       // Date of birth (auto-fill from lead)
  | "customfield"       // Custom field
  | "address"           // Address (auto-fill from lead)
```

### 3.3 CustomField
```typescript
{
  id: number;
  type: TypeCustomField;
  title: string;
  description?: string;
  default_value?: string;
  position_x: number;           // X coordinate on PDF page
  position_y: number;           // Y coordinate on PDF page
  width: number;                // Field width
  height: number;               // Field height
  required?: boolean;
  placeholder?: string;
  answers?: any;                // For dropdown/checkbox options
  signatory?: {                 // Assign field to signatory
    signable_id?: number;
    signable_type: 'Staff' | 'Lead';
  };
  color?: string;               // Field color
  font_size?: number;
  order: number;                // Field order
}
```

### 3.4 Template
```typescript
{
  id: number;
  name: string;
  type: string;
  pages: TemplatePages[];
  status?: string;
  favorite: boolean;
  require_authentication?: boolean;
  subject?: string;
}
```

### 3.5 TemplatePages
```typescript
{
  id: number;
  order: number;
  custom_fields: CustomField[];
  file: {
    mime_type: string;          // 'application/pdf'
    contents: string;           // Base64 PDF content
  };
  width: number;                // Page width
  height: number;               // Page height
  title?: string;
  description?: string;
}
```

### 3.6 Form
```typescript
{
  id: number;
  template_id?: number;
  template?: Template;
  lead_id: number;
  lead?: Lead;
  status: string;
  signing_status: string;
  signatories?: Signatory[];
  pages?: TemplatePages[];
  pending_lead_signature: boolean;
  pending_staff_signature: boolean;
  completed_at?: number;
  due_date?: number;
}
```

### 3.7 Signatory
```typescript
{
  id: number;
  form_id: number;
  signable_type: 'Staff' | 'Lead';
  signable_id: number;
  signable?: Staff | Lead;
  signature_status: string;
  signature_status_updated_at: Date;
  custom_fields?: CustomField[];
}
```

---

## 4. Lưu ý kỹ thuật

### 4.1 PDF Processing
- Upload PDF → Convert to base64
- Store in `file.contents` field
- Each page stored separately
- Support multi-page PDFs

### 4.2 Custom Fields Overlay
- Fields positioned với absolute coordinates (x, y)
- Coordinates relative to PDF page dimensions
- Fields rendered on top of PDF
- Support drag-drop repositioning

### 4.3 Auto-fill Fields
- Special field types auto-fill from lead data:
  - `firstname`, `lastname` → lead.first_name, lead.last_name
  - `emailaddress` → lead.email
  - `phonenumber` → lead.phone
  - `dateofbirth` → lead.date_of_birth
  - `address` → lead.address

### 4.4 Signature Flow
```
1. Create form from template
2. Assign signatories (lead + staff nếu cần)
3. Send to lead (email/SMS/portal)
4. Lead fills fields → saves as draft
5. Lead completes all required fields
6. Lead signs → signature_status = 'signed'
7. If require staff signature:
   - Notify staff
   - Staff reviews form
   - Staff signs
8. All signatories signed → form completed
9. Generate final signed PDF
```

### 4.5 Public Forms
- Route: `/:projectId/template/:id`
- No authentication required (nếu `require_authentication: false`)
- Visitor ID tracking
- Submit directly to template

### 4.6 Client Portal Forms
- Route: `/client/portal/:uuidProject/forms/:id`
- Require lead login
- Auto-fill lead information
- Save progress

---

## 5. Component Structure

```
pages/apps/forms.tsx
└── containers/apps/forms/
    ├── wrapper/
    ├── sidebar/              # Templates list
    ├── main/                 # Template editor / Form viewer
    └── form-nav/             # Navigation (pages, fields)
```

---

## 6. Template Types

### 6.1 Form Templates
### 6.2 Document Templates
### 6.3 Communication Templates
- Email templates
- SMS templates
- Quick reply templates
- Note templates

---

## 7. Permissions
- **All roles:** Có thể view forms và fill forms
- **Staff:** View forms assigned, sign forms
- **Admin/Owner:** Full access, create/edit/delete templates

