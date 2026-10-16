# ClaimSpire Admin Prototype — Team Meeting Speaking Script

**Use:** Speak this in a call with your manager / senior / team. Aim ~12–18 minutes + Q&A.  
**Tone:** Natural, confident, honest about POC vs production.  
**Demo ready:** http://127.0.0.1:8000/demo · accounts in `START_HERE_v3.md` · `pytest -q` = 25 tests.

**Related docs:** `08_SENIOR_TALK_TRACK.md` (short), `09_MULTI_TENANT_AND_CUSTOMER_DEMO.md` (click path), `07_COMPLETE_ADMIN_EXPLANATION.md` (depth).

---

## Before you unmute (30 seconds to yourself)

Open the demo. Have Platform Ops, Harbor, and Summit credentials ready. Mentally: **Admin configures ClaimSpire — it does not process claims.**

| Account | Password |
|---|---|
| `platform@claimspire.local` | `DemoAdmin123!` |
| `admin@harbor.local` | `DemoAdmin123!` |
| `admin@summit.local` | `DemoAdmin123!` |

---

## 1. Opening — what is this product? (~90 seconds)

> “I’ll walk through the ClaimSpire Admin prototype I've planned — what it is, who it’s for, how it works end to end, and why we made certain technical choices. I’ll also flag what’s real in the POC versus what still needs client and architecture decisions for production.
>
> ClaimSpire is a claims operations platform. Carriers, TPAs, and independent adjusting firms use it. Before anyone can handle claims in the Web App or Mobile App, the organization has to exist on the platform — with users, teams, configuration, and governance.
>
> **Administration is one of the four product surfaces in this project, it is the control surface.** It establishes and governs the operating environment. It is **not** a second claims-processing application. Desk adjusters adjust in Web. Field adjusters inspect in Mobile. Admin manages the organization, access, setup, and controls around that.”

**Likely cross-question**

- *“So is this replacing Claims Ops Admin?”*  
  > “No. Claims Ops Admin is a business persona for FNOL and claim intake in the Web App. Platform Ops and Tenant IT Admin are the Admin surface personas. Different job, different product surface.”

---

## 2. Who is it for? (~60 seconds)

> “Two Admin personas in the demo.
>
> **Platform Operations** — ClaimSpire’s internal platform administrators. They register organizations, review onboarding, activate tenants, and have cross-tenant visibility for operational administration.
>
> **Tenant IT Administrator** — the organization’s own admin. Harbor’s IT admin only sees Harbor. Summit’s only sees Summit.
>
> Same Admin UI shell for both. The differences are permissions and tenant scope, enforced in the API — not by hiding a menu and hoping nobody calls the endpoint.”

**Likely cross-question**

- *“Why can Platform Ops see all tenants — isn’t that a privacy risk?”*  
  > “In the POC it’s intentional so we can demonstrate platform-level ops. Production needs governed support access, PDP policies, and isolation. We’re not claiming unrestricted production access to every client’s claim data.”

---

## 3. What problem does it solve? (~60 seconds)

> “Imagine Harbor Mutual — a carrier — and Summit — an IA firm. Both use ClaimSpire, differently. Without Admin, how does the platform know which company a user belongs to, what teams exist, what’s enabled, or whether onboarding is done?
>
> That’s the problem. Admin answers four questions: Who’s on the platform? Is the org ready? How is it configured and governed? What changed, and who changed it?”

---

## 4. Organization types — important distinction (~75 seconds)

> “We support two organization classifications: **Carrier/TPA** and **IA Firm**. Those are business categories, not login roles.
>
> Separately we have **operating model** — Hybrid, Field, Desk-only, CAT — how they use ClaimSpire.
>
> And separately again, **persona** — Desk Adjuster, Field Adjuster, QA Reviewer, and so on.
>
> We deliberately did **not** collapse those into one dropdown. Harbor can be Carrier/TPA with Hybrid and employ Desk Adjusters. Summit can be IA Firm with Field and employ Field Adjusters. Same Admin product, different classification.”

**Likely cross-question**

- *“Why not two completely different forms for Carrier vs IA?”*  
  > “Common data — name, contacts, structure, branding — so we used one generic five-section collection with an organization-type field. When design confirms Carrier-specific vs IA-specific mandatory fields, we make sections conditional. We didn’t want to duplicate backends before fields are approved.”

- *“Are we locked to only two org types forever?”*  
  > “No. The catalogue is fixed in the POC for predictable validation. Extending it is a product decision, not a hard architecture limit.”

---

## 5. End-to-end lifecycle — the story (~2.5 minutes)

> “Here’s the journey we implemented, using a new company as the story.
>
> **One — Platform Ops registers the organization.** Name, slug, organization type, operating model. Backend creates a Tenant in **REGISTERED** status, plus a draft Organization Setup. Important: creating a tenant does **not** mean it’s operationally active.
>
> **Two — Platform Ops provisions the first Tenant IT Admin.** Only Platform Ops can grant that privileged role. That admin belongs to that tenant.
>
> **Three — Tenant Admin collects setup.** Five provisional sections: identity, contacts, LOBs and licenses, structure, branding. Save per section. Backend validates.
>
> **Four — Readiness.** Backend says what’s missing. Frontend displays it; we don’t trust the UI alone to decide ‘ready.’
>
> **Five — Assign setup owner, submit for review.** Submit creates an immutable submission snapshot and bumps version. Status goes to In Review.
>
> **Six — Platform Ops approves or returns.** Separation of duties: submitter can’t approve their own submission. Approve does **not** auto-activate.
>
> **Seven — Platform Ops activates.** Explicit activation gate. Then ongoing: users, teams, workforce availability, feature toggles, audit.
>
> So registration, setup approval, and activation are three different gates — that was intentional.”

**Likely cross-questions**

- *“Why isn’t Approve enough to go live?”*  
  > “Approving collected profile info isn’t the same as identity, integrations, retention, and UAT readiness. SOW managed activation is broader. The POC models that with a separate activate step.”

- *“Why snapshots on submit?”*  
  > “So review has a versioned record of what was submitted. Prevents silent edits under a reviewer, and supports concurrency with expected_version.”

---

## 6. Live walkthrough cues (~3–4 minutes)

*(Speak while clicking)*

> “I’ll show isolation first, because this is multi-tenant.
>
> Logging in as Harbor… Overview says I’m scoped to Harbor. Claims, users, features — Harbor only.  
> Summit login… different organization, different data.  
> Platform Ops… I can see all tenants and filter.
>
> Now register a third org — pick Carrier or IA at registration. Status REGISTERED. Create its Tenant Admin. Sign out, log in as that new admin with email and password — no hardcoded account list. Complete setup, submit, approve as Platform Ops, then Activate.
>
> Workforce: temporarily unavailable keeps login; offboarding disables login. Feature toggle with a reason shows in Audit Trail.
>
> Claims, Dispatch, QA, Finance screens are **read-only**. We’re monitoring synthetic data, not paying claims from Admin.”

**Likely cross-question**

- *“Can Harbor pull Summit’s claims via API if they guess the ID?”*  
  > “No — we return 403. The tenant filter in the UI is not the security boundary. We have automated tests for Harbor vs Summit isolation and for suspended tenants blocking Admin access.”

---

## 7. Tech stack and why (~2 minutes)

> “Stack for the prototype: **FastAPI**, **Pydantic**, **SQLAlchemy**, **SQLite**, local **JWT** auth with **PBKDF2** password hashing, and a plain **HTML/CSS/JS** Admin UI served by the same app.
>
> **Why FastAPI?** Typed REST, dependency injection for auth, OpenAPI for free — good for contracts and demos.
>
> **Why Pydantic separate from SQLAlchemy?** Models are storage; schemas are what HTTP may send or return. Frontend can’t dump arbitrary DB fields or read password hashes.
>
> **Why SQLite and no AWS?** We needed a runnable local POC without Cognito, Postgres, or cloud setup — validate journeys first.
>
> **Why local JWT instead of Cognito?** Same reason — prove login, roles, tenant scoping, offboarding blocking tokens. Production identity is Cognito plus PDP per SOW.
>
> **Why plain JS, not React?** Fastest path to a clickable Admin shell. Production Admin should align with the approved React design system later.
>
> **Why command APIs — submit, decide, activate — instead of PATCH status?** So callers can’t jump Draft to Approved. Business transitions carry validation and authorization.”

**Likely cross-questions**

- *“Isn’t SQLite a dead end?”*  
  > “For the POC, no. SQLAlchemy keeps the door open to Postgres. What we don’t pretend is that create_all equals migrations — production needs Alembic and RLS.”

- *“Why one role per user?”*  
  > “Demo simplification. Final permissions need the approved RBAC/ABAC matrix. We already block Tenant Admins from granting other Tenant IT Admins.”

- *“Where does business logic live?”*  
  > “Mostly in route modules today — fine for POC size. Before production we’d extract a service layer so routers stay thin.”

---

## 8. Multi-tenancy — how we handle it (~90 seconds)

> “This is a multi-tenant Admin. Every major record has a tenant_id. Platform Ops can operate across tenants; Tenant Admins are forced to their own.
>
> Helpers: permitted_tenant for lists, assert_resource_access for a specific resource, mutation_tenant when creating under a tenant. Suspended tenants block their Admin from Admin APIs.
>
> We seed Harbor as Carrier and Summit as IA so we can show two organization types sharing one Admin product with isolated data.”

**Likely cross-question**

- *“Do we have Postgres RLS?”*  
  > “Not in this POC. Application-level isolation plus tests. Production defense in depth includes RLS and PDP.”

---

## 9. What’s in scope vs what’s demo vs deferred (~90 seconds)

> “We’re careful about scope language.
>
> **Aligned with SOW Phase 1 thinking:** internal Platform Ops controls, AI/feature governance concepts, managed initial tenant onboarding, auditability.
>
> **In the POC as demonstration:** fourteen nav sections, synthetic claims monitors, local feature toggles, dynamic multi-org registration.
>
> **Explicitly not done / not claimed:** Cognito, real AppConfig kill-switch propagation, payment or QA decisions in Admin, full tenant self-service as a Phase 1 commitment, final Carrier vs IA form fields from design.
>
> A working screen is not the same as a signed Phase 1 deliverable. We use the demo to make decisions concrete.”

---

## 10. What we completed recently (~60 seconds)

> “Stabilization pass: email/password login so new admins can sign in without editing HTML; organization type at registration; REGISTERED versus ACTIVE with explicit activate; privileged role guard; versioned setup submissions; stronger multi-tenant checks; twenty-five automated API tests including isolation and suspend.”

---

## 11. What we’d do next (~60 seconds)

> “First, confirm with product and design: the real five collection flows, who approves setup, and activation checklist.  
> Then production path: Cognito and PDP, service-layer refactor, migrations, real AI control plane with effective-state verification, replace synthetic monitors with read models from owning services.  
> We should not invent production entitlement rules in code before P-07 and client sign-off.”

---

## 12. Closing line (~30 seconds)

> “In short: we built Admin as a platform control and organization-management surface — multi-tenant, two Admin personas, Carrier/TPA and IA classifications, onboarding with review and activation gates, workforce and config with audit — as a local FastAPI prototype so the team can see the journey and argue about the right production boundaries. Happy to take questions or click through any flow again.”

---

## Quick-fire appendix (if they keep asking)

| Question | Short answer |
|---|---|
| Why JSON for setup sections? | Fields still provisional; structured tables later for licenses/docs |
| Why Platform Ops as approver? | Provisional so the workflow is demoable; client decides final authority |
| Why read-only claims in Admin? | Visibility without taking ownership from Claim Core |
| Why PBKDF2 locally? | Avoid plaintext even in a demo; not a Cognito replacement |
| How do I reset the demo? | Stop server, delete `claimspire_admin.db`, restart |
| Where’s the deep write-up? | `lead_documents/07`, `08`, `09` + `START_HERE_v3.md` |

---

## Timing cheat sheet

| Block | Time |
|---|---|
| Opening + who/what | ~3 min |
| Lifecycle story | ~2.5 min |
| Live click-through | ~4 min |
| Tech + multi-tenant | ~3 min |
| Scope + next + close | ~2 min |
| Buffer / Q&A | rest of meeting |
