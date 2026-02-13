# Mantto — Production Architecture & Roadmap

## Executive Summary

Mantto is a CMMS (Computerized Maintenance Management System) for commercial buildings in Mexico. Current state: functional prototype. This document outlines the path to production.

**Target Market:** Property managers of commercial buildings (offices, mixed-use) in CDMX and major Mexican cities. Initial focus: 10-50 floor buildings with 5-20 maintenance staff.

**Revenue Model:** Per-building SaaS subscription
- Small (≤10 floors): $199 USD/mo
- Medium (11-30 floors): $399 USD/mo  
- Large (31+ floors): $799 USD/mo
- Enterprise (multi-building): Custom pricing

---

## Architecture Proposal

### Option A: Supabase-First (Recommended for Speed)

```
┌─────────────────────────────────────────────────────────────┐
│                        FRONTEND                              │
│  Next.js 14 (App Router) + Tailwind + shadcn/ui             │
│  - Admin Dashboard (web)                                     │
│  - Technician PWA (mobile-first)                            │
│  - Tenant Portal (web)                                       │
│  - Vendor Portal (web)                                       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                       SUPABASE                               │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│  │    Auth     │ │  PostgreSQL │ │  Realtime   │           │
│  │  (RLS)      │ │  (DB + RLS) │ │  (WebSocket)│           │
│  └─────────────┘ └─────────────┘ └─────────────┘           │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│  │   Storage   │ │    Edge     │ │    Cron     │           │
│  │  (files)    │ │  Functions  │ │   (pg_cron) │           │
│  └─────────────┘ └─────────────┘ └─────────────┘           │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     INTEGRATIONS                             │
│  - Twilio (WhatsApp Business API)                           │
│  - Resend (transactional email)                             │
│  - Stripe (billing)                                          │
│  - OpenAI (AI predictions, optional)                        │
└─────────────────────────────────────────────────────────────┘
```

**Why Supabase:**
- Built-in auth with Row Level Security (multi-tenant out of the box)
- Real-time subscriptions (technician tracking, live updates)
- PostgreSQL = robust, scalable, familiar
- Edge Functions for custom logic (WhatsApp webhooks, etc.)
- Fast to build, low ops overhead
- ~$25/mo to start, scales with usage

**Frontend Stack:**
- **Next.js 14** — Server components, API routes, easy deployment
- **Tailwind + shadcn/ui** — Matches our prototype aesthetic, fast iteration
- **PWA** — Technician app works offline, installable on phone
- **Hosting:** Vercel (free tier to start, auto-scaling)

### Option B: Custom Backend (More Control)

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Next.js App   │────▶│  Node/Express   │────▶│   PostgreSQL    │
│   (Frontend)    │     │  or FastAPI     │     │   + Redis       │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                               │
                        ┌──────┴──────┐
                        ▼             ▼
                   Socket.io     Bull Queue
                   (realtime)    (jobs)
```

**When to choose this:**
- Need complex business logic
- Want full control over infrastructure
- Plan to raise VC and need "real" architecture
- Team has backend experience

**Trade-off:** 2-3x longer to build, more ops work, but more flexible long-term.

### Recommendation

**Start with Option A (Supabase).** Move to Option B only if/when:
- Supabase costs exceed $500/mo
- Need features Supabase can't handle
- Raising serious funding and need to show "enterprise" stack

---

## Database Schema (Core Tables)

```sql
-- Multi-tenant via organization_id on all tables
organizations (
  id, name, slug, plan, floors, address, 
  whatsapp_number, logo_url, settings,
  created_at, stripe_customer_id
)

users (
  id, organization_id, email, name, role,
  phone, avatar_url, is_active,
  -- roles: admin, manager, technician, tenant, vendor
)

work_orders (
  id, organization_id, 
  title, description, category, priority, status,
  floor, location, reported_by, assigned_to,
  due_date, completed_at, resolution_notes,
  source, -- web, whatsapp, tenant_portal, qr_scan
  created_at, updated_at
)

work_order_comments (
  id, work_order_id, user_id, 
  comment, attachments, created_at
)

assets (
  id, organization_id,
  name, category, location, floor,
  qr_code, serial_number, install_date,
  warranty_expires, maintenance_interval_days,
  last_maintenance, next_maintenance
)

vendors (
  id, organization_id,
  name, category, contact_name, email, phone,
  rating, avg_response_time, total_jobs,
  is_verified
)

rfqs (
  id, organization_id,
  title, description, category, location,
  budget_min, budget_max, deadline,
  status, -- draft, sent, responses, awarded, closed
  created_by, created_at
)

rfq_invites (
  id, rfq_id, vendor_id, 
  sent_at, viewed_at, responded_at
)

quotes (
  id, rfq_id, vendor_id,
  price, timeline_days, warranty_months,
  includes, notes, attachments,
  is_selected, submitted_at
)

preventive_maintenance (
  id, organization_id, asset_id,
  title, description, interval_days,
  last_completed, next_due, assigned_to
)

compliance_items (
  id, organization_id,
  title, description, authority,
  due_date, status, -- pending, submitted, approved, expired
  attachments, notes
)
```

---

## User Roles & Portals

### 1. Admin Dashboard (Web)
**Users:** Building managers, property administrators
**Features:**
- Full work order management
- Staff assignment & tracking
- Vendor management & RFQs
- Reporting & analytics
- Asset management
- Compliance tracking
- Settings & billing

### 2. Technician App (PWA)
**Users:** Maintenance staff
**Features:**
- My assigned tasks (prioritized list)
- Task details + location + history
- Update status, add notes/photos
- QR scan to view asset / create ticket
- Real-time notifications
- Offline mode (sync when connected)

### 3. Tenant Portal (Web)
**Users:** Office tenants, building occupants
**Features:**
- Submit maintenance request
- Track request status
- View building announcements
- Contact info

### 4. Vendor Portal (Web)
**Users:** External service providers
**Features:**
- View RFQ invitations
- Submit quotes
- Track awarded jobs
- Invoice submission
- Performance dashboard

---

## Key Integrations

### WhatsApp Business API (Critical)
Mexican businesses live on WhatsApp. Two integration points:

1. **Inbound:** Tenants/staff report issues via WhatsApp
   - Webhook receives messages
   - AI extracts: category, location, urgency
   - Creates work order automatically
   - Confirms with user

2. **Outbound:** Notifications to technicians, vendors, tenants
   - Task assigned → WhatsApp to technician
   - Quote received → WhatsApp to admin
   - Task completed → WhatsApp to requester

**Provider:** Twilio WhatsApp Business API (~$0.05/message)

### Email (Resend)
- Quote requests to vendors
- Weekly reports to management
- Compliance deadline reminders

### Stripe (Billing)
- Subscription management
- Invoice generation
- Usage-based billing (optional)

---

## Development Roadmap

### Phase 0: Foundation (Week 1-2)
**Goal:** Project setup, design system, auth

- [ ] Initialize Next.js 14 + Tailwind + shadcn/ui
- [ ] Set up Supabase project (dev + prod)
- [ ] Design database schema, create migrations
- [ ] Implement auth (email + magic link)
- [ ] Create organization onboarding flow
- [ ] Build design system (port prototype components)
- [ ] Set up CI/CD (Vercel)

**Deliverable:** Empty app with auth, can create organization

### Phase 1: Core MVP (Week 3-6)
**Goal:** Replace prototype with real functionality

- [ ] Work order CRUD (create, list, detail, update)
- [ ] Work order filters & search
- [ ] Staff management (invite, roles)
- [ ] Task assignment
- [ ] Basic dashboard with stats
- [ ] Activity feed
- [ ] Mobile-responsive (technician view)

**Deliverable:** Usable work order system for pilot customer

### Phase 2: Technician Experience (Week 7-8)
**Goal:** Mobile app for field staff

- [ ] PWA setup (manifest, service worker)
- [ ] My Tasks view (assigned to me)
- [ ] Task detail with actions
- [ ] Photo upload for completions
- [ ] Offline support (queue actions)
- [ ] Push notifications
- [ ] GPS tracking (optional, for live map)

**Deliverable:** Technicians can work entirely from phone

### Phase 3: Vendor & RFQ System (Week 9-11)
**Goal:** Full quote workflow

- [ ] Vendor directory (CRUD)
- [ ] RFQ creation modal → real data
- [ ] Vendor portal (view RFQs, submit quotes)
- [ ] Quote comparison view
- [ ] Award quote → create work order
- [ ] Vendor performance tracking

**Deliverable:** End-to-end quote workflow

### Phase 4: WhatsApp Integration (Week 12-13)
**Goal:** Inbound + outbound messaging

- [ ] Twilio WhatsApp Business setup
- [ ] Webhook for inbound messages
- [ ] AI message parsing (category, location)
- [ ] Auto-create work orders from WhatsApp
- [ ] Outbound notifications (task assigned, completed)
- [ ] Conversation history on work orders

**Deliverable:** Staff/tenants can report issues via WhatsApp

### Phase 5: Assets & Preventive Maintenance (Week 14-15)
**Goal:** Proactive maintenance

- [ ] Asset registry (CRUD)
- [ ] QR code generation + printing
- [ ] QR scan → asset detail / create ticket
- [ ] Preventive maintenance schedules
- [ ] Auto-generate PM work orders
- [ ] Maintenance history per asset

**Deliverable:** Full asset lifecycle management

### Phase 6: Reporting & Polish (Week 16-18)
**Goal:** Production-ready

- [ ] Monthly report generation (PDF)
- [ ] Cost tracking & analysis
- [ ] Technician performance metrics
- [ ] Compliance tracker
- [ ] Building floor map visualization
- [ ] Dark mode
- [ ] Settings & customization
- [ ] Billing integration (Stripe)
- [ ] Onboarding flow polish

**Deliverable:** Ready for paying customers

---

## Estimated Costs (Monthly)

### Development Phase
| Service | Cost |
|---------|------|
| Supabase Pro | $25 |
| Vercel Pro | $20 |
| Domain | ~$1 |
| Twilio (testing) | ~$20 |
| **Total** | **~$66/mo** |

### Production (10 customers)
| Service | Cost |
|---------|------|
| Supabase Pro | $25-75 |
| Vercel Pro | $20 |
| Twilio WhatsApp | ~$100 |
| Resend | $20 |
| OpenAI (AI features) | ~$50 |
| **Total** | **~$215-265/mo** |

### Break-even
At $399/mo average price, break-even at **1 customer**.
10 customers = ~$4K/mo revenue, ~$265 costs = **93% margin**.

---

## Go-to-Market

### Pilot Customer
Isaac's friend managing Torre Reforma 115 (30 floors). 
- Free pilot for 3 months
- In exchange: feedback, testimonial, referrals
- Success metric: 80% of work orders created in system

### Early Sales Strategy
1. **Warm intros** — Isaac's real estate network
2. **Property management companies** — One contract = multiple buildings
3. **AMPI/AMPPI** (Mexican RE associations) — Speak at events
4. **Content marketing** — "Moderniza tu edificio" guides

### Competitive Positioning
- **vs. UpKeep/Fiix/Limble:** Those are US-focused, expensive ($150+/user/mo), no Spanish, no WhatsApp
- **vs. spreadsheets/WhatsApp groups:** "You're already using WhatsApp — now it creates tickets automatically"
- **vs. custom development:** 10x cheaper, 10x faster, maintained for you

---

## Risk & Mitigations

| Risk | Mitigation |
|------|------------|
| Pilot doesn't convert | Get commitment upfront, involve in design |
| WhatsApp API complexity | Start with outbound only, add inbound later |
| Supabase limitations | Architecture allows migration to custom backend |
| Market too small | Expand to residential, hospitality, industrial |
| Competitors enter MX | Move fast, build relationships, local presence |

---

## Next Steps

1. **Validate pricing** — Ask pilot customer what they'd pay
2. **Confirm scope** — What's MVP vs nice-to-have for Torre Reforma?
3. **Decision:** Build in-house or hire dev help?
4. **If go:** Start Phase 0 immediately

---

## Appendix: Tech Decisions

### Why Next.js over React + Vite?
- Server components = faster initial load
- API routes = no separate backend for simple things
- Vercel deployment = zero config
- App Router = modern patterns, better DX

### Why shadcn/ui?
- Copy-paste components (own the code)
- Tailwind-based (matches prototype)
- Highly customizable
- No vendor lock-in

### Why PWA over React Native?
- Single codebase
- No app store approval
- Instant updates
- Good enough for CRUD app
- Can always add native later

### Why PostgreSQL?
- Industry standard
- Complex queries (reporting)
- JSONB for flexible fields
- PostGIS for location features (future)
- Supabase = managed Postgres
