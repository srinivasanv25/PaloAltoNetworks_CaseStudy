# PaloAltoNetworks_CaseStudy

TechGlobal is a fictional global enterprise software company. Its Accounts Payable (AP) organization processes thousands of supplier invoices across multiple business units, legal entities, and geographical regions.
Suppliers submit invoices through shared email addresses such as invoices@techglobal.example.
These emails may contain PDF invoices, scanned documents, image attachments, multiple invoices, or other supplier correspondence. Invoice layouts and formats vary significantly between suppliers.
Today, AP specialists manually perform several activities:
Invoice intake: Monitor shared mailboxes, identify invoices, retrieve attachments, and determine whether an invoice has already been received.
Data extraction: Read invoice documents and extract supplier details, invoice numbers, dates, currency, amounts, taxes, and line items.
Data enrichment: Retrieve additional information from enterprise systems, including supplier master data, purchase orders, organizational information, and payment terms.
Transformation and validation: Normalize extracted values, apply business rules, reconcile amounts, and verify purchase-order eligibility.
Exception management: Investigate missing information, ambiguous supplier identities, mismatched purchase orders, and other discrepancies.
Invoice submission: Prepare and submit validated invoice records into the company's enterprise Procure-to-Pay (P2P) platform.
Tracking and audit: Monitor processing status, resolve downstream errors, and retain evidence of invoice-processing decisions.
This process is labor-intensive, inconsistent across teams, difficult to scale during high-volume periods, and dependent on manual interpretation of documents and enterprise data.
TechGlobal wants to develop Invoice Flow, an AI-assisted invoice-processing platform that can automate routine invoice processing while maintaining financial accuracy, explainability, and appropriate human oversight.
The long-term goal is to support straight-through processing for invoices that can be reliably validated, while routing ambiguous or noncompliant invoices to AP specialists for review.
3. Your Task
Design the technical architecture and build a working prototype demonstrating the core execution flow for Invoice Flow.
Your solution should address how the platform can:
Receive and process invoice submissions originating from a shared email mailbox.
Identify relevant invoice documents and extract structured information from varying document formats.
Normalize invoice data into a consistent internal representation.
Enrich extracted information using existing enterprise systems.
Apply deterministic accounting, supplier, and purchase-order validation rules.
Determine whether an invoice can proceed automatically or requires human intervention.
Capture human corrections and resume processing without losing previously completed work.
Submit validated invoices into a downstream enterprise P2P platform.
Handle duplicate events, partial failures, retries, and uncertain downstream outcomes.
Preserve sufficient evidence, workflow history, and data lineage to explain how each invoice was processed.
Your design should demonstrate clear separation between AI-assisted interpretation, deterministic business logic, enterprise data access, workflow orchestration, and downstream system mutations.
There is no prescribed AI agent framework, orchestration product, document-processing provider, database, or integration platform.
Choose the architecture and technologies you believe are appropriate, and explain the trade-offs.

4. Existing Enterprise Systems and Business Rules
Assume TechGlobal already operates several enterprise systems that Invoice Flow must integrate with.
You do not need access to real enterprise systems. Create representative mock APIs, services, or datasets for your prototype.
4.1 Supplier Master
The Supplier Master is the authoritative source for approved suppliers.
Representative fields include:
Field
Description
supplier_id
Unique enterprise supplier identifier
legal_name
Registered supplier name
aliases
Known alternate or trading names
tax_id
Registered tax identifier
status
Active, inactive, or blocked
legal_entities
TechGlobal entities authorized to transact with the supplier
payment_terms
Approved supplier payment terms
default_currency
Default transaction currency

Business rules:
Every invoice must resolve to an approved and active supplier before submission.
Supplier names on invoices may differ from registered legal names.
Matching by name alone may be insufficient when multiple supplier records are plausible.
Inactive, blocked, or ambiguous suppliers must not be automatically submitted.
Extracted supplier information must not silently overwrite authoritative supplier master data.
4.2 Purchase Order System
The Purchase Order system maintains approved purchasing commitments.
Representative fields include:
Field
Description
po_number
Unique purchase order identifier
supplier_id
Associated enterprise supplier
legal_entity
Purchasing legal entity
currency
Approved purchase currency
status
Open, closed, canceled
line_items
Approved goods or services, quantities, and unit prices
remaining_balance
Amount available for additional invoicing

Business rules:
PO-backed invoices must reference an existing, open purchase order.
The invoice supplier, legal entity, and currency must match the purchase order.
Invoice line items must be reconciled against the corresponding PO lines.
Unit-price variances greater than 2% require human review.
An invoice subtotal exceeding the PO's available balance must be flagged.
A closed or canceled PO cannot be used for automatic submission.
Missing or ambiguous PO-line matches must not be silently resolved by the AI model.
For this exercise, assume PO available balances and price tolerances are evaluated before tax. Candidates may document additional assumptions around freight, discounts, and partial invoicing.
4.3 Invoice Registry
The Invoice Registry tracks invoices previously received or submitted through enterprise AP processes.
Representative fields include:
Enterprise supplier ID
Legal entity
Normalized invoice number
Invoice date
Invoice amount
Processing status
Downstream invoice reference
Business rules:
The combination of supplier ID, legal entity, and normalized invoice number is the business duplicate key for this exercise.
Duplicate email events and repeated attachments must not produce duplicate downstream invoice creation.
A previously submitted invoice must not be resubmitted merely because the original request was retried.
An existing invoice record may require reconciliation with the downstream system before processing continues.
4.4 Invoice Validation and Transformation
TechGlobal requires invoice data to be represented using a canonical internal schema before downstream submission.
At minimum, that representation should support:
Supplier identification
Invoice number and invoice date
Purchase order reference, when applicable
Legal entity
Currency
Invoice line items, quantities, and prices
Subtotal, discounts, shipping, tax, and total
Payment terms
Source document reference
Validation status and exceptions
Business rules:
Extracted invoice amounts must reconcile with line-item totals and applicable adjustments.
A rounding difference of up to $1.00 in invoice currency is permitted for this exercise.
Original extracted values must remain traceable when transformations or corrections occur.
Currency, date, and numeric-format normalization must not silently alter the financial meaning of a value.
Information retrieved from authoritative enterprise systems should take precedence over unsupported document-derived assumptions.
Missing required fields must be resolved or escalated rather than filled with fabricated values.
Candidates should explain how their design distinguishes original document content, extracted values, normalized values, enriched data, and human corrections.
4.5 Non-PO Invoices
Some suppliers submit legitimate invoices without purchase-order references.
Business rules:
Non-PO invoices may enter the same processing workflow.
They must be associated with a valid supplier and legal entity.
They require an authorized AP approval before downstream submission.
The system must preserve the approval decision, reviewer identity, and relevant reasoning.
The AI system must not independently authorize a non-PO invoice.
4.6 Downstream P2P Platform
The enterprise P2P platform is responsible for creating and managing official invoice records.
For this assignment, assume an integration interface supports:
Creating an invoice
Looking up a previously submitted invoice
Retrieving submission status
Returning validation errors and enterprise invoice identifiers
The downstream integration may be fully mocked.
However, the solution must address what happens when the downstream platform successfully accepts an invoice but the response is lost or times out.
A retry must not blindly create another invoice.

5. Human Review and Exception Management
Not every invoice can or should be processed automatically.
Invoice Flow must distinguish between invoices that are eligible for straight-through processing and invoices requiring further investigation or approval.
Examples include:
Scenario
Expected behavior
Required fields extracted reliably, supplier and PO valid, all rules pass
Eligible for automatic submission
Low-confidence or missing invoice field
Request human verification or correction
Multiple plausible supplier matches
Require supplier resolution
PO price or balance mismatch
Flag discrepancy for review
Missing PO on a non-PO invoice
Require authorized AP approval
Duplicate invoice already submitted
Prevent duplicate submission and reconcile status
Downstream response is uncertain
Recover or reconcile safely before retrying

The architecture should explain:
How extraction uncertainty and business-rule failures are represented.
How the system determines whether to proceed, retry, request clarification, or require approval.
How an AP specialist views the relevant invoice information and supporting evidence.
How a reviewer corrects extracted data or resolves an exception.
How corrections are validated before processing resumes.
How reviewer actions, overrides, and approvals are recorded.
How the processing workflow resumes without restarting unrelated successful steps.
Important: Extraction confidence alone is not sufficient to authorize invoice submission. A highly confident extraction may still violate financial or procurement business rules.
The AI system may propose values, matches, or resolutions, but must not bypass required deterministic validations or human approvals.
A production-quality review interface is not required for the prototype. A simple API, command-line interaction, or minimal UI is acceptable.




---

## Folder Structure

```
dev-os/
├── CLAUDE.md                     # Project instructions & stage-gated workflow rules
├── docs/
│   ├── design.md                 # Brand design system (colors, typography, spacing, components)
│   ├── PAN_CaseStudy_PRD.md         # Product Requirements Document for ContractIQ
├── skills/                       # Claude Code custom skills (slash commands)
│   ├── engineering-planner/
│   │   └── SKILL.md              # Stage 1: PRD → engineering docs
│   ├── implementation-specs/
│   │   └── SKILL.md              # Stage 2: Engineering docs → granular specs + SQL
│   ├── security-foundation/
│   │   └── SKILL.md              # Stage 3: Specs → security controls + RLS policies
│   ├── frontend-setup/
│   │   └── SKILL.md              # Stage 4: Scaffold Next.js 14 App Router project
│   └── design-system/
│       └── SKILL.md              # Applied during all frontend work — enforces design.md
```

> Files and folders under `docs/engineering/`, `docs/specs/`, `docs/security/`, and `.env.example` are generated outputs — they are created during the build workflow and do not exist until their respective stage runs.

---

## The Build Workflow

Each stage must be completed and approved before the next begins.

| Stage | Skill | Input | Output |
|---|---|---|---|
| 1 — Engineering Plan | `/engineering-planner` | PRD | `docs/engineering/engineering-doc.md` + `docs/engineering/implementation-specs.md` |
| 2 — Implementation Specs | `/implementation-specs` | Stage 1 docs | `docs/specs/*.md` + `docs/specs/supabase-schema.sql` + `.env.example` |
| 3 — Security Foundation | `/security-foundation` | Stages 1–2 docs | `docs/security/security-plan.md` + `supabase/rls-policies.sql` + `src/lib/security/` |
| 4 — Frontend Setup | `/frontend-setup` | Stages 2–3 docs | Scaffolded Next.js 14 app |
| 5 — Feature Implementation | *(manual, one feature at a time)* | `docs/specs/*.md` | Feature code |

---

## Skills

Skills live in `skills/<name>/SKILL.md` and are invoked as slash commands in Claude Code.

### `/engineering-planner`
Transforms a PRD into two engineering documents. Asks clarifying questions (auth strategy, DB, LLM provider, user roles) before generating anything. Output is the authoritative reference for all downstream stages.

### `/implementation-specs`
Reads the approved engineering docs and generates granular, runnable specs — one file per concern. Always produces `supabase-schema.sql` (paste-and-run SQL) and `.env.example` (all environment variables grouped by service).

### `/security-foundation`
Reviews all engineering and spec documents, identifies every security surface, and implements controls before any feature code is written. Covers auth, rate limiting, prompt injection, token limits, file upload validation, RLS policies, and environment variable protection.

### `/frontend-setup`
Scaffolds a complete Next.js 14 (App Router) project — `package.json`, `next.config.mjs`, `app/layout.jsx`, `app/globals.css`, `app/page.jsx` — and runs the dev server.

### `/design-system`
Enforces the brand design system defined in `docs/design.md` on all frontend code. Applied automatically whenever any UI component, page, or style is written. Uses Inter for body text, JetBrains Mono for contract content, and `#112E81` as the primary brand color.

---

## Key Rules

- **Never skip a stage.** Each stage depends on the output of the previous one.
- **Never proceed without user approval.** Claude stops after every stage and waits.
- **Never write code before specs exist.** Implementation only starts after Stage 2 is approved.
- **Always apply the design system** when writing any frontend code.

---

## Invoice Flow: Intelligent Invoice Processing

 An AI-assisted invoice-processing platform that can automate routine invoice processing while maintaining financial accuracy, explainability, and appropriate human oversight.
 The PRD is at `docs/InvoiceFlow_PRD.md`.

Tech stack (fixed):
- **Frontend:** Next.js 14 (App Router)
- **Backend:** Next.js API Routes
- **Database:** Supabase (PostgreSQL + Auth + Storage)
- **AI:** Anthropic Claude API
- **UI:** Lucide React icons, Inter + JetBrains Mono fonts
