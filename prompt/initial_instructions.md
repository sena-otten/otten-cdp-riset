
# 📘 CRM Otten Coffee
## B2B & B2C Sales Pipeline, Customer Segmentation & Messaging

---

## 1. Overview & Objective

Dokumen ini mendefinisikan **CRM terpadu untuk Otten Coffee** yang mencakup:
- Pengelolaan **sales pipeline end-to-end**
- Pemisahan jelas **B2B vs B2C**
- Integrasi **WhatsApp, Email, dan Push Notification**
- Struktur data konsisten untuk **Sales, CS, Product, Data, dan Engineering**

CRM ini berfungsi sebagai **single source of truth** untuk seluruh aktivitas sales, customer interaction, dan retention.

---

## 2. Customer Grouping

### 2.1 Customer Group

| Customer Group | Segment |
|---|---|
| **B2B** | Cafe, Restaurant, Hotel, Office |
| **B2C** | Personal |

**Aturan:**
- 1 customer = 1 customer group
- Semua deal mewarisi `customer_group` dan `segment`
- Pipeline & messaging berbeda antara B2B dan B2C

---

## 3. Source of Customer

### 3.1 Source Method
- Canvassing
- Event
- CSO / Academy
- Referral

### 3.2 Source App
- B2B App
- Retail App
- Customer Service (CS)

---

## 4. Sales Pipeline

```
Lead
→ Qualified
→ Discovery
→ Proposal
→ Negotiation
→ Won / Lost
→ Retention
```

**Aturan pipeline:**
- Won / Lost wajib reason
- Retention hanya untuk deal Won
- Setiap stage memiliki PIC & SLA

---

## 5. Activity Recording

Activity recording dari berbagai sumber untuk tracking interaksi dengan customer.

### 5.1 Activity Source & Types

**B2B Sales Follow Up:**
- call_outbound
- visit
- meeting
- email_sent
- whatsapp_sent
- proposal_sent
- sample_sent
- demo_scheduled
- negotiation
- contract_sent

**Customer Service:**
- call_inbound
- inquiry
- complaint
- order_support
- technical_support
- feedback_received
- ticket_created
- ticket_resolved
- escalation

**AI Automation:**
- auto_followup_reminder
- auto_email_sent
- auto_whatsapp_sent
- lead_score_updated
- churn_risk_alert
- reorder_reminder
- birthday_greeting
- campaign_triggered
- sentiment_analyzed

### 5.2 Activity Fields

**Core Fields:**
- activity_id
- activity_type
- activity_source (b2b_sales / customer_service / ai_automation / system)
- customer_id (FK)
- location_id (FK)
- deal_id (FK)

**Transaction Relation (Polymorphic):**
- transaction_id
- transaction_ref_table (ecommerce_orders / pos_transactions / b2b_orders)
- transaction_channel (retail_app / pos / marketplace)
- transaction_amount

**Execution & Follow Up:**
- performed_by
- performed_at
- duration_minutes
- outcome
- next_action
- next_action_date
- notes
- attachments[]

**Contoh Transaction Relation:**
| Field | Contoh Value |
|-------|-------------|
| transaction_id | ORD-2024-001234 |
| transaction_ref_table | ecommerce_orders |
| transaction_channel | mobile_app |

### 5.3 Outcome Options
- completed
- no_answer
- rescheduled
- interested
- not_interested
- follow_up_needed
- escalated

### 5.4 AI Automation Triggers

#### B2B Triggers

| Trigger | Condition | Action |
|---------|-----------|--------|
| Reorder Reminder | last_order > 30 days & monthly_coffee_kg > 0 | WhatsApp to PIC |
| Churn Risk Alert | no_activity > 60 days & deal_status = won | Notify Sales PIC |
| Follow Up Reminder | next_action_date = today | Push to Sales App |
| Contract Renewal | contract_end_date - 30 days | Email + Notify Sales |
| Lead Score Update | activity_count changed | Recalculate score |
| PIC Birthday | pic_birthday = today | Auto WhatsApp |
| New Location Detected | brand.total_locations increased | Notify Sales for upsell |
| Payment Overdue | invoice_due_date < today & unpaid | Email + Notify Finance |
| Consumption Drop | monthly_order < avg_monthly * 0.5 | Alert Sales PIC |

#### B2C Triggers

| Trigger | Condition | Action |
|---------|-----------|--------|
| Welcome Series | new_customer & first_order_completed | Email + Push sequence |
| Reorder Reminder | last_order > 14 days & has_consumable | Push Notification |
| Cart Abandonment | cart_items > 0 & no_checkout > 1 hour | Push + Email |
| Birthday Promo | birthday = today OR birthday - 7 days | Push + Email with voucher |
| Membership Upgrade | total_spent > tier_threshold | Push + Email congrats |
| Inactive Win-back | no_order > 30 days | Email with promo |
| Review Request | order_delivered + 3 days | Push to review |
| Wishlist Price Drop | wishlist_item.price decreased | Push Notification |
| Back in Stock | notify_stock_item.stock > 0 | Push + Email |

---

## 6. Product Dimension

- Coffee Beans
- Syrup
- Powder
- Machine
- Tools
- Training
- Maintenance

Satu deal dapat memiliki lebih dari satu product dimension.

---

## 7. Messaging Integration

### 7.1 Channel
- WhatsApp
- Email
- Push Notification (**hanya untuk contact yang memiliki app**)

### 7.2 Eligibility Rules

```
Push       → has_app = true AND push_token exists
WhatsApp  → phone valid AND whatsapp_opt_in = true
Email     → email valid AND email_opt_in = true
```

Fallback otomatis:
1. Push
2. WhatsApp
3. Email

---

## 8. Contact Global Fields

- primary_phone
- primary_email
- has_app
- app_type (b2b / retail / both)
- app_user_id
- push_tokens
- whatsapp_opt_in
- email_opt_in
- preferred_channel
- timezone
- last_contacted_at

---

## 9. Field Design

### 9.1 Global Field

- customer_id
- customer_group
- segment
- customer_name
- source_method
- source_app
- pic_owner
- current_stage
- estimated_deal_value
- product_dimension
- created_at
- last_activity_at

### 9.2 B2B Entity Structure

**Aturan:** Nama bisnis dapat berbeda dengan nama legal. Satu legal entity bisa memiliki banyak brand/location.

```
Legal Entity (1) → Brand (N) → Location (N)
```

**Contoh:**
| Legal Entity | Brand | Segment | Locations |
|---|---|---|---|
| PT Fore Kopi Indonesia Tbk | Fore Coffee | Cafe | Sudirman, Kemang, +50 |
| PT Marriott Indonesia | JW Marriott | Hotel | Jakarta, Bali, Surabaya |
| PT Bank Central Asia Tbk | BCA | Office | Menara BCA, Wisma BCA |

#### Legal Entity Fields
- legal_entity_id
- legal_name
- tax_id (NPWP)
- business_license
- billing_address
- billing_email
- payment_term
- credit_limit

#### Brand Fields
- brand_id
- brand_name
- legal_entity_id (FK)
- segment
- business_scale
- total_locations
- pic_name
- pic_role

#### Location Fields (Core)
- location_id
- location_name
- brand_id (FK)
- is_active

#### Location Detail Fields (dari Existing B2B App)

**Alamat:**
- address_pinpoint (GPS coordinate)
- province
- city
- district (kecamatan)
- postal_code
- address_detail

**Kontak & PIC:**
- phone
- pic_name
- pic_role (owner/manager/staff)
- instagram

**Media:**
- photos[] (min 3: front, exterior, interior)
- notes

#### B2B Deal Fields
- decision_maker
- contract_term
- sla_requirement
- deal_level (brand/location)

**Aturan Entity B2B:**
- Legal Entity = entitas hukum untuk kontrak & invoice
- Brand = nama bisnis yang dikenal publik
- Location = cabang/lokasi fisik untuk delivery & service
- Deal bisa di-attach ke level Brand atau Location
- Invoice selalu ke Legal Entity

#### B2B Segment Extension (Location Level)

**Cafe / Restaurant**
- total_locations
- monthly_coffee_kg
- monthly_coffee_budget
- coffee_notes_preference
- roasting_profile (light/medium/medium_dark/dark)
- sample_available (yes/no)
- existing_machine_brand
- seating_capacity

**Hotel**
- number_of_rooms
- breakfast_volume
- banquet_event_usage
- monthly_coffee_kg
- monthly_coffee_budget
- existing_machine_brand

**Office**
- employee_count
- pantry_model
- monthly_coffee_kg
- monthly_coffee_budget
- existing_machine_brand

#### Reference Values

**PIC Role:**
- Owner
- Manager
- Staff

**Roasting Profile:**
- Light
- Medium
- Medium Dark
- Dark

### 9.3 B2C – Personal
- full_name
- skill_level
- brew_preference
- budget_range
- home_usage

---

## 10. Messaging Strategy

| Channel | B2B | B2C |
|---|---|---|
| WhatsApp | Primary | Support |
| Email | Primary | Optional |
| Push | Jika punya app | Primary |

---

## 11. Role & Ownership

### Product
- Pipeline & rules
- Segment & field definition
- Messaging use case & template

### Data Team
- ERD & standardisasi data
- Funnel & conversion metrics
- Dashboard & attribution

### Engineering
- CRM backend & UI
- Messaging service & provider
- Consent & audit log

### Sales
- Pipeline execution
- Discovery & negotiation
- Won / Lost reason

### Customer Service
- Inbound lead
- Post-sales support
- Retention & service case

---

## 12. Expected Outcome

- Pipeline visibility real-time
- Forecast revenue B2B
- Retention & upsell terstruktur
- Siap untuk automation & AI CRM
