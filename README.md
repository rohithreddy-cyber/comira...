# CoMitra — Smart Collaborative Manufacturing Platform

> **Project Tagline:** One Order. Multiple Manufacturers. One Success.  
> **Target Sector:** B2B Industrial Manufacturing Consortiums & MSME Manufacturing Clusters.  
> **Architecture:** Full-stack Next.js (App Router), TypeScript, Tailwind CSS, Prisma ORM, PostgreSQL / SQLite.

---

## 1. Executive Summary & Problem Statement

Manufacturing enterprises regularly encounter large customer orders that surpass their available shopfloor machinery, skilled labor, or delivery timelines. Historically, single factories have been forced to outright reject lucrative contracts or risk severe delay penalties—even while neighboring certified factories operate with unutilized or idle machine capacity.

**CoMitra** addresses this challenge through a centralized B2B collaborative manufacturing platform. Instead of a single company attempting to produce the entire order alone, the platform enables the lead manufacturer to:
1. Publish bulk contract specifications and in-house capacity limits.
2. Run an explainable, rule-based partner discovery engine to match certified manufacturers with complementary capabilities and excess available capacity.
3. Transact and lock work allocations digitally so cumulative allocations never exceed order limits.
4. Coordinate production milestones, CMM inspection sign-offs, and early delay reporting.
5. Generate digital legal agreements and tax-compliant milestone invoices with simulated escrow payments.

---

## 2. Core Architecture & Technology Stack

```mermaid
graph TD
  User[Browser / Client] -->|HTTPS / Next.js 14 App Router| NextApp[Next.js Frontend & API Handlers]
  NextApp -->|Authentication| JWT[JWT Session & Bcrypt Authorization]
  NextApp -->|Matching Engine| MatchEngine[Deterministic Rule-Based Partner Matcher]
  NextApp -->|Optional AI Copilot| Gemini[Google Gemini 1.5 Flash API]
  NextApp -->|ORM Data Layer| Prisma[Prisma ORM 5.22]
  Prisma -->|Storage| Database[(SQLite Local / PostgreSQL Production)]
  NextApp -->|Audit & Events| AuditTrail[Immutable Audit Event Store]
```

### Stack Components
- **Frontend:** Next.js 14 (App Router), React 18, TypeScript, Tailwind CSS, Lucide Icons, Recharts for industrial telemetry.
- **Backend:** Next.js Server Route Handlers, Zod schema validation, server-side role & ownership guards.
- **Database:** Prisma ORM with normalized relational schema. Compatible with SQLite (zero-config local demo) and PostgreSQL (managed Neon / Supabase cloud deployment).
- **Authentication:** JWT HTTP-only cookie sessions with password hashing via `bcryptjs`.
- **Testing:** Vitest for unit & integration testing across business logic, schemas, matching formulas, and security.

---

## 3. Algorithmic Partner Matching Engine & Formula

The matching engine operates **100% autonomously without external AI dependencies** to guarantee deterministic, auditable decisions. It scores candidate manufacturers across five normalized, weighted factors:

$$\text{Recommendation Score} = 0.35 \times S_{\text{capability}} + 0.25 \times S_{\text{capacity}} + 0.15 \times S_{\text{quality}} + 0.15 \times S_{\text{deadline}} + 0.10 \times S_{\text{trust}}$$

### Factor Definitions:
1. **Manufacturing Capability Compatibility ($S_{\text{capability}}$, Weight: 35%):**  
   Evaluates keyword and process overlap between the order's required capabilities (e.g., CNC Machining, Heat Treatment, Surface Finishing) and the candidate's verified shopfloor capabilities.
2. **Available Capacity Feasibility ($S_{\text{capacity}}$, Weight: 25%):**  
   Ratio of candidate's available uncommitted throughput to the order's remaining unallocated volume. Scores 100% if available capacity $\ge$ required quantity.
3. **Quality Standards & Certifications ($S_{\text{quality}}$, Weight: 15%):**  
   Evaluates regulatory standing (Verified MSME: 90%, Pending: 65%, Unverified: 30%) plus bonus points for ISO 9001:2015, AS9100D, or NADCAP credentials.
4. **Deadline & Lead Time Feasibility ($S_{\text{deadline}}$, Weight: 15%):**  
   Compares candidate's advertised lead times against the customer's delivery milestone window.
5. **Historical Trust & Performance ($S_{\text{trust}}$, Weight: 10%):**  
   Derived from verified collaboration review averages across on-time delivery, communication, and dimensional quality.

### Match Categories:
- **EXCELLENT:** Total Score $\ge 85\%$ with high capability and capacity safety.
- **HIGH:** Total Score $\ge 70\%$.
- **MODERATE:** Total Score $\ge 50\%$.
- **LOW:** Total Score $< 50\%$ or missing core capabilities.

---

## 4. Complete Demonstration Scenario

The database is pre-seeded with an aerospace manufacturing contract:

### Scenario Context:
**Order Ref:** `ORD-2026-10088` — **10,000 Units: Aircraft High-Pressure Hydraulic Actuator Housings**
- **Lead Manufacturer:** *Apex Precision Engineering Pvt Ltd* receives order for 10,000 units.
- **Constraint:** Apex has in-house capacity for only **6,000 units** before the delivery deadline.
- **Resolution via CoMitra:**
  1. Apex self-allocates 6,000 units and opens 4,000 units for collaboration.
  2. The Matching Engine identifies *Bharat ForgeWorks* (Score: 96.5% - EXCELLENT) and *Crestline Heavy Dynamics* (Score: 91.0% - HIGH).
  3. Apex issues an invitation for **2,500 units** to Bharat ForgeWorks and **1,500 units** to Crestline Heavy Dynamics.
  4. Both partners accept. Work allocations are recorded transactionally; agreement `AGR-2026-APEX-BF-001` is drafted and digitally executed.
  5. Shopfloors log progress:
     - Apex: 4,200 / 6,000 units produced.
     - Bharat Forge: 1,800 / 2,500 units produced.
     - Crestline: 950 / 1,500 units produced.
  6. **Cumulative Progress:** **6,950 / 10,000 units (69.5%)** calculated without double-counting.
  7. Milestone tax invoice `INV-2026-BF-00108` for ₹9,55,800 (including 18% GST) is settled via the demonstration payment simulator.

---

## 5. Demonstration Accounts & Persona Switching

The application includes a persistent top bar enabling one-click persona switching:

| Persona | Name | Email | Password | Role | Entity |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Lead Owner** | Rohit Deshmukh | `owner@apex.com` | `Password123!` | OWNER | Apex Precision Engineering (Lead) |
| **Partner 1** | Suresh Patel | `partner@bharatforge.com` | `Password123!` | PARTNER | Bharat ForgeWorks (2,500 units) |
| **Partner 2** | Ananya Rao | `partner@crestline.com` | `Password123!` | PARTNER | Crestline Heavy Dynamics (1,500 units) |
| **Partner 3** | Karthik Subramanian | `partner@deltamicro.com` | `Password123!` | PARTNER | Delta MicroTech (Additive labs) |
| **Admin** | Dr. Vikram Sethi | `admin@comitra.gov.in` | `AdminSecret123!` | ADMIN | Regulatory & Audit Directorate |

---

## 6. Installation & Execution Guide

### Prerequisites
- Node.js version 18.x or 20.x or 24.x
- Git

### Setup Steps
```bash
# 1. Clone the repository
git clone <repository-url>
cd comira

# 2. Install dependencies
npm install

# 3. Environment configuration
# The .env file defaults to SQLite for zero-config demonstration:
# DATABASE_URL="file:./dev.db"

# 4. Initialize database schema
npx prisma generate
npx prisma db push

# 5. Populate demonstration records
npm run prisma:seed

# 6. Run automated test suite
npm test

# 7. Start the development server
npm run dev
```

Open your browser at `http://localhost:3000`.

---

## 7. PostgreSQL Production Deployment (Neon / Supabase / Railway)

To deploy to PostgreSQL:
1. In `prisma/schema.prisma`, update the datasource provider:
   ```prisma
   datasource db {
     provider = "postgresql"
     url      = env("DATABASE_URL")
   }
   ```
2. In `.env`, supply your PostgreSQL connection string:
   ```env
   DATABASE_URL="postgresql://user:password@host:5432/comitra?sslmode=require"
   ```
3. Run migrations and seed:
   ```bash
   npx prisma db push
   npm run prisma:seed
   ```
4. Deploy to Vercel or any Node.js container host.

---

## 8. API Reference Summary

| Endpoint | Method | Description | Guard |
| :--- | :--- | :--- | :--- |
| `/api/auth/register` | POST | Register new user and manufacturing business | Public |
| `/api/auth/login` | POST | Authenticate and issue HTTP-only session cookie | Public |
| `/api/auth/logout` | POST | Revoke session cookie | Public |
| `/api/auth/me` | GET | Retrieve authenticated profile, capabilities, and capacity | Authenticated |
| `/api/orders` | GET / POST | List bulk orders or publish new order with in-house capacity checks | Authenticated |
| `/api/orders/[id]` | GET / PATCH / DELETE | Order specifications, progress breakdown, cancel order | Authenticated |
| `/api/orders/[id]/recommendations` | GET | Run algorithmic matching engine with custom factor weights | Authenticated |
| `/api/collaboration/requests` | GET / POST | Manage invitations; server enforces available capacity limits | Authenticated |
| `/api/collaboration/requests/[id]` | PATCH | Accept, reject, or withdraw collaboration invitation | Business Owner / Partner |
| `/api/production/updates` | GET / POST | Log completed quantities, CMM inspection passes, delay reports | Allocation Holder |
| `/api/capacity` | GET / PATCH | Retrieve throughput; enforces limit $\ge$ committed orders | Authenticated |
| `/api/agreements` | GET / POST | Subcontract documents, digital signing, and revision tracking | Contract Parties |
| `/api/invoices` | GET / POST | Issue GST milestone invoices, simulate escrow settlement | Contract Parties |
| `/api/businesses` | GET | Filterable catalog of certified manufacturing units | Public / Authenticated |
| `/api/reviews` | GET / POST | Submit peer ratings restricted to verified collaborators | Verified Collaborators |
| `/api/analytics` | GET | Industrial volume telemetry, utilization rates, CSV export | Authenticated |
| `/api/admin` | GET / POST | Verification approval, account suspension, full audit logs | ADMIN Only |
| `/api/ai/insights` | POST | Optional Google Gemini executive summary (safe fallback) | Authenticated |

---

## 9. Security & Governance Implementation

- **Role-Based Access Control (RBAC):** Server-side guards distinguish `OWNER`, `PARTNER`, and `ADMIN`. Ordinary users cannot view or manipulate administrative queues or other factories' private records.
- **Capacity Integrity:** Atomic database transactions ensure a factory cannot reduce its total capacity below active order obligations.
- **Non-Double Counting:** Aggregated order completion percentages are derived strictly from active non-cancelled subcontracts.
- **Immutable Audit Logging:** Every registration, order modification, allocation sign-off, delay report, and invoice payment records an `AuditEvent` with actor ID, timestamp, and target metadata.

---

## 10. Known Limitations & Future Roadmap

- **Simulated Financial Settlement:** Invoices include precise GST arithmetic and reference IDs; real-world banking integration (Razorpay / Stripe B2B Escrow) can be connected to the existing payment interface.
- **CAD File 3D Viewer:** Currently, orders store rich technical specifications; future revisions could incorporate WebGL CAD `.STEP` / `.STL` mesh rendering.
- **ERP Webhooks:** Direct connector hooks for SAP / Oracle NetSuite to sync shopfloor machine sensors with CoMitra milestone feeds.
