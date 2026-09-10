# Permaconn HubSpot Build — Key Decisions & Build Notes (Security Division)

**Source:** 5 Fireflies meeting transcripts
**Participants:** Patrick McAuliffe, Aimee Engelmann, Prachi Singh (Myrtl) · Hayden Lee, Charles Ridler (Permaconn)
**Portals in play:** Permaconn UK (Prod + Sandbox), Permaconn AU (Prod + Sandbox)

| # | Meeting | Date | Focus |
|---|---------|------|-------|
| 1 | Hubspot Permaconn | 10 Sep 2026 | Products/SKUs, Service desk, Zoom Phone, Sugar migration |
| 2 | Product and Product Assumption for Permaconn | — | Revenue model, average unit value, reporting |
| 3 | Product assumption in Permaconn UK/AU | — | Product Assumption vs. standard Product |
| 4 | Permaconn product assumption – fix workflow / set Average Unit value | 11 Aug 2026 | Workflow bug fix, price books |
| 5 | Amending the installation schedule for Permaconn deals | — | Deal Activation records, date logic, data gaps |

---

## 1. Headline decisions

| Decision | Detail | Status |
|---|---|---|
| **Keep go-live simple** | Only SKUs relevant to raising an opportunity go into the product list. HubSpot-as-billing-system is a later phase, not go-live. | Agreed (Hayden + Amy) |
| **Wipe & replace SKUs** | Pull every SKU currently in HubSpot, wipe, re-import Hayden's reviewed file (sourced from MYOB billing). Safe — sandbox holds no customer data. | Agreed |
| **Two revenue streams kept separate** | Deal `amount` (line items) = Total Contract Value for hardware. Deal Activation records = Total Actual Unit Revenue. Combined via a **new calculated property**. | Agreed |
| **Product Assumption object retained** | Average Unit Value stays sourced from the custom Product Assumption object, not standard Product. | Agreed (see §3.2) |
| **Product becomes mandatory** | Sales must set Product before a deal can move to *Actively Deploying* — avoids retro-fixing 265+ imported deals. | Built; awaiting Amy's approval |
| **Zoom Phone stays 1 portal : 1 account** | No cross-portal call logging. Cross-region calls get manual tickets. | Agreed (Charles + Hayden) |
| **Single service pipeline** | New → Open → Escalated → Resolved, shared across all channels/regions. | Built |

---

## 2. Products, SKUs & price books

### Build decisions
- **Product list rebuilt from MYOB.** Hayden ran a full review out of the MYOB billing system and re-grouped/re-categorised. Only opportunity-relevant SKUs are included — no dumping the full catalogue.
- **Import method:** HubSpot CSV import wizard (Products → Import → advanced objects → field mapping). Patrick can bypass via CLI if bulk push is faster, but the wizard is preferred for Hayden's muscle memory and repeatability.
- **Field creation is Patrick's task.** Hayden to send the file; Patrick creates any missing corresponding properties (~5 min). Headings in Hayden's file:
  - Product Cloud
  - Region
  - Product Line (Security / IoT / Both)
  - Product Type (Hardware / Accessory / Data Pack / VPN)
  - Description
  - Price
  - Charge Type
  - Billing Frequency *(added by Patrick — confirmed as an existing property)*
  - Currency
- **Price book model:** one product library, **two price books (AU / NZ)**, selected by rule from a **Region dropdown on the deal**.
  - Explicitly **not** driven by account business address or deal currency — AU customers operate in NZ and vice versa, so the rep picks the region on the opportunity.
  - Rationale: one product record with multiple price points beats duplicating the whole catalogue per country.
- **Finn to assist** with the product upload and specifically the price book association (Amy's call, to accelerate delivery).

---

## 3. The revenue model (the core architectural decision)

### 3.1 Two values, deliberately not merged

```
Deal.amount              = Σ line items            → Total Contract Value (hardware, one-off)
Deal Activation records  = Avg Unit Value × Units  → Total Actual Unit Revenue (recurring-like)
                                    ↓
        NEW calculated property = TCV + Total Actual Unit Revenue
        (working name: "Permaconn Total Revenue")
```

**Why they are separate — build rationale (Patrick):**
> "Whenever you're playing with stuff that comes out of the box with HubSpot, it's already pre-configured to work a certain way and to get it to work a different way normally means breaking something else. We could have had line items that indicated total actual unit revenue, but then we would have lost the ability to have total contract value in the amount property."

**Decisions:**
- Line items may **not** be re-purposed to drive unit revenue — that breaks the standard `amount` field.
- A **separate calculated property** sums the two. Display it on the deal record (and on the deal card if it can be squeezed in).
- **Expectation management required:** the team must understand HubSpot's default `amount` is *only* the line items, not full contract value.
- **Commercial rationale:** Craig's sales team care about **units**, not hardware. Each unit behaves like a subscription (ongoing revenue/commission at each installed site); hardware is a one-off sale.

### 3.2 Product Assumption vs. standard Product — resolved

Both objects are used, for different jobs:

| Object | Used for | Why |
|---|---|---|
| **Product + Line Items** | Hardware TCV → `amount` | HubSpot-native, pre-built for TCV/ARR/MRR |
| **Product Assumption** (custom) | Average Unit Value → Total Actual Unit Revenue | Only reliable way to set avg unit value on a deal post-close |

**Decision:** keep Product Assumption and copy the existing workflow. Patrick's reasoning — moving to standard Product "sounds like the right thing to do", but it can't be validated in the time available, and *"Charles isn't going to care whether we've used standard Product or Product Assumption."* Acknowledged as slightly janky; it was originally built that way on Claude's recommendation.

### 3.3 Average Unit Value — record structure

- **Only two "box" products trigger Average Unit Value: Secure Link and Rapid Link.**
- Amy's instruction: **delete all other branches/records** (Secure Link SIM data packs, RomTech were incorrectly included). Patrick deleted the extra Product Assumption record on the call.
- **One Product Assumption record per box product** — Rapid Link and Secure Link cannot share an average (e.g. Rapid Link ~£100, Secure Link ~£200).
- **UK values in GBP.** AU sandbox has **no** Product Assumption records or GBP values set up yet.
- Current values are **placeholders** — real figures pending from Charles.
- **Permissions:** hide the default Average Unit Value property on the deal; **admin-edit only**, not editable by all sales users.
- **Layout:** the admin-only deal card holding Average Unit Value was removed and **must be re-added** — otherwise closed/actively-deploying deals show no value once forecasts start.

### 3.4 Workflow bug — Average Unit Value not populating

- **Symptom:** Average Unit Value was not being set consistently on applicable deals.
- **Root cause (working theory):** deals created **via import** often don't trigger the workflow. *Needs further validation.*
- **Trigger:** deal created **AND** product is one of the signalling products (Secure Link / Rapid Link). Branch logic confirmed correct.
- **Resolution:** fixed and tested in **UK**; applied to **both orgs** and the product-specific property card; missing values back-filled on existing deals. **Not yet tested in Sandbox AU** (no Product Assumption records / GBP values there yet).
- Workflow itself was unchanged — the fix was in trigger/data conditions.

---

## 4. Installation schedule & Deal Activation records

### Creation rule
Child **Deal Activation** records are generated when:
```
Product ∈ {Secure Link, Rapid Link}  AND  Deal Stage = "Actively Deploying"  (= Closed Won)
```
Only actively-deployed deals need them. Imported deals sitting in Prospecting etc. correctly have none.

### Date logic — redesigned
Replaced the old 30/60/90-day approach with a months-out model:

| Property | Lives on | Purpose |
|---|---|---|
| **Activation Start Date** | Deal | Anchor date, set by sales |
| **Months Out** | Deal Activation | Sequence number, 1 → 24 |
| **Activation Month Calculation** | Deal Activation (calculated) | Start Date + N months, stamped onto Activation Month |
| **Activation Month** | Deal Activation | The stamped, fixed date |

**Build note — must be hidden:** *Activation Month Calculation* must **not** appear on any page layout. Once the record is created and the date stamped, the calculated value recalculates and displays incorrectly.

**Noted alternative (not taken):** each node could reference the previous node's field recursively (May → May+1 → May+2). Current approach uses the calculated property instead.

### Data quality issue — 265 deals with no product
- After the deal import, **265 deals** have no Product assigned (source file from Debbie, plus one from James; Mark and Prachi both confirmed **no product indicator exists in the source data** — only "preferred panel", which is something else).
- **Decision proposed to Amy:** make Product a **required property** on the pipeline stage rule for *Actively Deploying*, rather than retro-setting Product on 100+ deals.
- Patrick implemented the stage rule (Deal Settings → Pipelines → Edit → Add rule → Product → Required) and removed the old "open existing logic". **Awaiting Amy's confirmation/approval.**

### Cleanup flags
- **"Migration Product" field** — added on Claude's advice, purpose never established. Candidate for removal.
- Several **contract date fields** likely removable — hold until after the final training session so the team can confirm what they actually want.
- Deleting test deals requires deleting the **child Deal Activation records** first.

---

## 5. Installed-units data feed (interim → data lake)

- **Weekly installed-units spreadsheet** comes from **Simon Liddy** (via Craig). This is the interim process until Permaconn's data lake is live.
- **Matching key: `Atlas Reference Project`** — a unique identifier, so imports don't rely on name-based matching. Amy to ask Simon to include the Atlas ID in the XLS.
- **Restrict `Atlas Reference Project`** to admin/Craig edit only — user overrides would break matching. (Note: there are hundreds of swap-outs, so free-text editing is a real risk.)
- **Process:** Prachi runs the manual import for the first couple of weeks, then writes a **process document** and hands over. Superseded when the data lake goes live.

---

## 6. Reporting

- Two **sample reports for Charles**, one per product:
  `units installed (this month) × average unit value (per product type) = revenue`
- **Do not put on a dashboard yet** — no real data in the fields. Sample reports only.
- Owner: Prachi. Priority: low / later in the week.
- Audience note: this view matters to **Charles and Tom** (revenue thinkers). Front-line sales users (Jason, Debbie, Raj) only care about units.

---

## 7. Service / Help Desk build

### Channels in scope
| Channel | Portal | Note |
|---|---|---|
| UK Support | UK | |
| ANZ Security | AU | Covers AU **and** NZ |
| RomTech | — | Address corrected to **service@romtechgrid.com** (was support@…). Not going live now, configured for later |
| Patriot | — | Inbound is mainly **web form**, then email, then phone |
| Global Finance / Global Sales | — | **Excluded** for now |

### Decisions
- **One single ticket pipeline for both regions:** New → Open → Escalated → Resolved.
- Email channels connect by OAuth; licences confirmed with Steph. Fake inbound emails seeded in **Sandbox AU** for demo/testing.
- Ticket assignment to teams to be automated off the inbound channel.
- **Constraint:** Permaconn has the **basic workflow automation tool** — no custom-code workflow actions. All service automation must fit that ceiling.

### Zoom Phone integration
- **Hard constraint:** a Zoom Phone account maps **1:1 to a single HubSpot portal**. It cannot feed both UK and AU.
- **Decision (Charles + Hayden): don't solve it perfectly.** UK agents covering AU overnight will log into the AU HubSpot portal with Zoom simply not connected, and **create tickets manually**. Cross-region is the exception, not the rule (~1–2 UK calls/week vs ~10 the other way).
- Fail-safe confirmed: the connection simply won't establish in the second portal, so agents can't misfile calls.
- **What the integration gives:** call activity record, inbound/outbound + comment values, and **AI voice-to-text transcription**, auto-associated to the contact by AI.
- **Build task:** push the AI call transcription/notes into the **ticket description** via workflow (Patrick has done something similar before — must fit the basic workflow tool).
- Permaconn has **Zoom Phone, not Zoom Contact Centre** — so no true channel import and no IVR automation.

### Unknown-caller / ticket creation — stripped back for go-live
- Inbound calls do **not** auto-populate a ticket create form.
- **Decision (Amy + Hayden): strip the create form right back.** No mandatory email address. A contact can be created on **first name + phone number** alone.
  - Rationale: 3 tech support agents, queues of up to ~19 calls; asking for an email address kills adoption and slows the call. Speed beats data completeness at go-live; build up from there.
- **Alternative/complementary approach:** a **pre-built AI assistant prompt** with 2–3 preset inputs that creates the contact, creates the ticket, and summarises the call transcription — replacing the form entirely. Patrick to build one to demo.
- **Race condition identified:** call logged before the contact exists. Mitigation — workflow: on activity created, wait ~5 minutes, then auto-associate the activity to the record matching that number. Patrick to validate what Zoom Phone actually does here.
- **Self-healing path:** callers usually email later; the email channel creates the contact and records can be **merged**, pulling call + ticket history together.

### Templates, snippets, macros (Amy)
- **Templates:** any CRM property can drop in as a personalisation token (first name, ticket number, created date, reference number, company). Use the ticket **owner's first name only** — no surnames.
- **Macros:** combine 2–3 steps into one button (Send & Close, Send & Escalate, Send & Follow-up). Example escalation macro: notify person → set high priority → send acknowledgement.
- **Not required for launch** but recommended — high time-saver, works before any AI tools are enabled.
- Hayden's angle: the 3 security/IoT tech support staff report pain points daily; he wants to build macros himself as they surface.

### Change management
- Optional short **preview videos** of Help Desk before training (Amy's suggestion — pre-empt the "it's different to Outlook" reaction; show split view vs. table view, movable AI summary).
- **Hayden's assessment: low need for Security/IoT.** The 3 tech support staff haven't used Outlook for cases in 5–6 years — they work exclusively in Salesforce/Sugar. HubSpot is a clear step up; they won't want to go back.
- Walkthrough sessions run separately: **service/support stakeholders** (leaders only, not full team) and **sales**.

---

## 8. Sugar CRM migration — field decisions

| Field | Decision |
|---|---|
| **Photo URL** | **Drop** — no value ("what would we put there, the company logo?") |
| **Account hierarchy** | **Do not import** — likely wrong, out of date, or misinterpreted by the sales team |
| **Atlas ID / Rapid Link IDs** (Sugar v7, absent from v8) | **Not migrated in phase 1** — sourced from the portal integration and will come via the data lake |
| **`vertical_c`** | **Migrate as-is.** Permaconn division: Fire, Security, IoT, FHIR. Doubles as marketing segmentation |
| **Attrition + Attrition Flag** | **Migrate both**, even where empty, so the mapping exists |
| **ABN / ACN** | Fields exist; **external IDs deferred** pending Farzan's external mapping work → likely data lake |

**Convention:** any Sugar field **without** `_c` is out-of-the-box; anything **with** `_c` is a Hayden-built custom field.

**Data handling note:** Patrick exported and **randomised** the data set for modelling — no real customer data held on local/cold storage.

---

## 9. Environments, access & permissions

- **Permaconn UK Production was built first** (early rollout). The UK **sandbox was deleted and refreshed** to get a clean, current copy — the recommended pattern for any stale sandbox (also flagged for Bravo's).
- Prachi was given **Claude tooling access** to port custom fields, properties, layouts and workflows from the UK portal into **Sandbox AU**.
- **Sequence:** bring Sandbox AU to parity → **turn Claude off** → run a **test data import from Sugar**.
- **Permissions gap:** no teams/permissions configured in Sandbox AU. Required:
  - **Sales User** permission set
  - **Sales Leader** permission set → **Tom** and **Hayden**
  - A test **salesperson** user for sandbox testing
  - **Hayden = admin** in the sandbox
- **Views:** currently **default only** — everyone sees the same view in UK (contacts/companies/deals) and Sandbox AU. Role-specific views not yet built.
- **Leads object is not enabled** — lead imports go in as **deals at Prospecting stage**.
- **Sequencing:** UK is close to handover (final sales training + marketing assets with Finn outstanding). **AU is the large remaining piece.**

---

## 10. Ways of working

- **Recap emails after every meeting** listing everything discussed and everything Patrick will action — deliberate practice to create a written record of what was asked for.
- Completed tasks logged back into the **Teams channel** with links, so Patrick and Amy can review work as it goes.
- Known issue: **quote template download error** in HubSpot (also fails via Print-to-PDF) — support ticket raised.

---

## 11. Open items / owners

| Item | Owner | Blocking |
|---|---|---|
| Average Unit Value figures for **Secure Link** and **Rapid Link** (GBP for UK; AU TBC) | **Charles** (via Amy) | Unit revenue calc, reports |
| Send final product/SKU CSV for wipe-and-replace | **Hayden** | Product import, price books |
| Approve **mandatory Product before Actively Deploying** | **Amy** | 265 deals with no product |
| UK installed-units XLS **with Atlas ID** | **Simon Liddy** (via Amy) | Weekly units import |
| Create the **TCV + Unit Revenue calculated property**; validate feasibility | **Patrick** | Total revenue display |
| Re-add **admin-only deal card** with Average Unit Value to layout | **Prachi / Patrick** | Forecast visibility |
| Validate: do **import-created records skip workflow triggers**? | **Patrick** | Avg unit value reliability |
| Test Average Unit Value workflow in **Sandbox AU** (needs Product Assumption records + values) | **Prachi** | AU go-live |
| Build **Sales User / Sales Leader** permission sets in Sandbox AU; assign Tom + Hayden | **Prachi** | AU sandbox testing |
| Validate **Zoom Phone unknown-caller** association behaviour + 5-min auto-associate workflow | **Patrick** | Service go-live |
| Build stripped-back **ticket create form** + AI assistant prompt | **Patrick** | Service go-live |
| Two **sample reports** for Charles (units × avg unit value) | **Prachi** | Low priority |
| Price book association setup | **Finn** (with Patrick) | Product import |
| Marketing assets | **Finn** (via Amy) | UK handover |

---

## 12. Risks & watch-items

1. **`amount` ≠ full contract value.** The single biggest expectation-management risk. Any report, dashboard or forecast built on HubSpot's default `amount` understates revenue by the entire unit-revenue component.
2. **Import-created records may not fire workflows.** Unconfirmed root cause. If true, every bulk import needs a follow-up back-fill workflow or a re-enrolment step.
3. **Product Assumption is a non-standard pattern.** Accepted knowingly for delivery speed. It sits outside HubSpot's native product/line-item machinery, so future HubSpot features (quotes, billing, forecasting) won't understand it.
4. **265 deals with no product**, and no product indicator in the source data. The mandatory-field rule stops the bleeding but does not fix history.
5. **Basic workflow tool ceiling** — no custom code actions. Several planned service automations (call transcript → ticket description, 5-minute delayed association) must be proven within that limit.
6. **Zoom Phone 1:1 portal limit** is a permanent structural constraint, not a bug. Cross-region overnight cover is manual by design.
7. **Atlas Reference Project is free-text and edit-prone**, with hundreds of swap-outs. Locking it to admin/Craig is essential or weekly matching breaks.
8. **Sandbox AU is behind UK** — no Product Assumption records, no GBP/AUD assumption values, no permission sets, no role views.
