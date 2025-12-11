# Client Portal Module - Patient Portal

## 1. Giới thiệu
Client Portal (Patient Portal) là portal riêng cho leads/patients để xem và quản lý thông tin của họ, không cần login vào dashboard chính.

**Routes:**
- `/client-portal` - Portal dashboard (internal view)
- `/client/portal/:uuidProject` - Client portal (public - for patients)

**Scope:** Patient self-service portal với authentication, view appointments, forms, invoices, payments.

---

## 2. Portal Features

### 2.1 Authentication Flow
```
1. Patient visits /client/portal/:uuidProject
2. Login screen (phone + verification code)
3. Verify client info (first name, last name, DOB, phone)
4. Access portal home
```

**Steps:**
- `login` - Phone verification
- `verify` - Client info confirmation
- `home` - Portal dashboard

![Client Portal Login](./images/image.png)

---

### 2.2 Portal Tabs

#### Tab 1: Home
**Features:**
- Overview dashboard
- Quick actions
- Upcoming appointments
- Pending forms
- Recent invoices

![Portal Home Dashboard](./images/image-1.png)

#### Tab 2: Appointments
**Features:**
- View upcoming appointments
- View past appointments
- Appointment details (date, time, location, procedures)
- Appointment stages:
  - `upcoming` - Confirmed future appointments
  - `in-progress` - Currently happening
  - `previous` - Past appointments

**API:**
- GET `/portal/events` - Get appointments
  - `between[from]` - Start timestamp
  - `between[to]` - End timestamp

![Appointments List](./images/image-2.png)

#### Tab 3: Forms
**Features:**
- View forms to complete
- Fill forms
- View completed forms
- Form count badge (pending forms)

**API:**
- GET `/portal/forms` - Get forms list
- GET `/portal/forms/:id` - Get form detail
- PUT `/portal/forms/:id/page/:pageId/response` - Fill form
- PUT `/portal/forms/:id/sign` - Sign form
- GET `/portal/forms/count-to-complete` - Get pending count

![Forms List](./images/image-3.png)

#### Tab 4: Billing & Payments
**Features:**
- View invoices
- View payment history
- Pay invoices (Stripe)
- View/update payment method (credit card)
- Download invoice PDF

**API:**
- GET `/portal/invoices` - Get invoices
- GET `/portal/payments` - Get payment history
- GET `/portal/billing/card` - Get card info
- PUT `/portal/billing/card` - Update card
- POST `/portal/invoices/:id/pay/card` - Pay invoice
- POST `/portal/invoices/:id/pdf-payment` - Download PDF

**Payment Flow:**
```
1. View invoice
2. Click "Pay Now"
3. Enter/select payment method (Stripe Elements)
4. Process payment
5. Show receipt
6. Send receipt email
```

![Invoice Payment](./images/image-4.png)

#### Tab 5: Personal Information
**Features:**
- View personal info
- Update profile (first name, last name, DOB, phone, email, address)

**API:**
- GET `/portal/client-info` - Get client info
- PUT `/portal/client-info` - Update client info

![Personal Information](./images/image-5.png)

#### Tab 6: Files
**Features:**
- View uploaded files
- Download files
- File types: documents, images, PDFs

**API:**
- GET `/portal/files` - Get files list (limit: 100)

---

## 3. API Endpoints

| Method | Endpoint | Mục đích | Auth Required |
|--------|----------|----------|---------------|
| POST | `/portal/login/:leadId` | Login với phone + code | No |
| GET | `/portal/client-info` | Lấy client info | Yes |
| PUT | `/portal/client-info` | Update client info | Yes |
| GET | `/portal/events` | Lấy appointments | Yes |
| GET | `/portal/forms` | Lấy forms list | Yes |
| GET | `/portal/forms/:id` | Chi tiết form | Yes |
| PUT | `/portal/forms/:id/page/:pageId/response` | Fill form | Yes |
| PUT | `/portal/forms/:id/sign` | Sign form | Yes |
| GET | `/portal/forms/count-to-complete` | Count pending forms | Yes |
| GET | `/portal/invoices` | Lấy invoices | Yes |
| GET | `/portal/payments` | Lấy payment history | Yes |
| GET | `/portal/billing/card` | Lấy card info | Yes |
| PUT | `/portal/billing/card` | Update card | Yes |
| POST | `/portal/invoices/:id/pay/card` | Pay invoice | Yes |
| POST | `/portal/invoices/:id/pdf-payment` | Download invoice PDF | Yes |
| GET | `/portal/files` | Lấy files | Yes |
| GET | `/portal/widget-uuid` | Get widget UUID (for booking) | Yes |

---

## 4. Authentication

### 4.1 Login Flow
```typescript
POST /portal/login/:leadId
{
  phone: string;
  code: string;        // Verification code sent via SMS
}
```

**Process:**
1. Patient enters phone number
2. System sends verification code via SMS
3. Patient enters code
4. System verifies code
5. Create session
6. Return client info

### 4.2 Client Info Verification
After login, if client info incomplete:
- Required fields: `first_name`, `last_name`, `date_of_birth`, `phone`
- Show verification screen
- Patient confirms/updates info
- Proceed to portal

---

## 5. Lưu ý kỹ thuật

### 5.1 Project UUID
- Portal URL uses `uuidProject` instead of numeric ID
- Format: `/client/portal/:uuidProject`
- UUID maps to project in database

### 5.2 Stripe Integration
- Uses Stripe Elements for payment form
- Connected account support (multi-tenant)
- Stripe account ID from `clientInfo.project.owner.stripe_account.stripe_id`
- Load Stripe with connected account:
```typescript
loadStripe(STRIPE_KEY, {
  stripeAccount: connectedAccountId
})
```

### 5.3 Session Management
- Portal uses separate authentication from main dashboard
- Session stored in cookies/localStorage
- Auto-logout after inactivity

### 5.4 Responsive Design
- Mobile-first design
- Works on phones, tablets, desktops
- Touch-friendly UI

### 5.5 Branding
- Portal uses project's logo (`projectInfo.logo_url`)
- Project name
- Custom colors (if configured)



---

## 6. Component Structure

```
pages/apps/client-portal.tsx
└── containers/apps/client-portal/
    ├── wrapper/
    └── main/
        ├── index.tsx             # Main portal logic
        ├── home/                 # Home tab
        ├── appointments/         # Appointments tab
        ├── forms/                # Forms tab
        ├── billing-payments/     # Billing tab (21 components)
        ├── personal-information/ # Personal info tab
        └── invoice-main-view.tsx # Invoice detail view
```

---

## 7. Portal States

### 7.1 Loading States
- `isLoadingHomePage` - Initial portal load
- `isLoadingFormPortal` - Forms loading
- `isLoadingEventPortal` - Appointments loading
- `isLoadingInvoicePortal` - Invoices loading
- `isLoadingFilePortal` - Files loading
- `loadingGetCard` - Card info loading

### 7.2 Tab States
```typescript
type TabActive = 
  | "home" 
  | "appointments" 
  | "forms" 
  | "billings" 
  | "personal-info" 
  | "files";
```

### 7.3 Step States
```typescript
type Step = "login" | "verify" | "home";
```

---

## 8. Data Fetching

### 8.1 Appointments
```typescript
// Upcoming appointments (next 6 months)
getEventPortalByLeadIDApi({
  project_id: projectId,
  "between[from]": moment().unix(),
  "between[to]": moment().add(6, "months").unix()
})

// Past appointments
getEventPortalByLeadIDApi({
  project_id: projectId,
  "between[from]": 1,
  "between[to]": moment().unix()
})
```

**Appointment Stages:**
- `in-progress` - Event started but not ended
- `upcoming` - Future confirmed events
- `previous` - Past events

### 8.2 Forms
```typescript
getFormPortalByLeadIDApi({
  project_id: projectId,
  page?: number
})
```

**Pagination:**
- Load more forms on scroll
- Append to existing list

### 8.3 Invoices & Payments
```typescript
// Invoices
getInvoicesPortalByLeadIDApi({
  project_id: projectId
})

// Payment history
getPaymentPortalByLeadIDApi({
  project_id: projectId
})
```

---