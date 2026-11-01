# Controlled AI Outreach Validation and Provider Comparison Automation

**HBA Phase 1 — Controlled Core (DRAFT)**

n8n workflow for controlled validation of AI-powered lead research, scoring, pathway selection, and outreach draft generation — **before** any real sending path is connected.

It supports two locked modes:

- **Synthetic mode** → 6 controlled fixture records (qualified, below-threshold, suppressed, duplicate, malformed)
- **Historical comparison mode** → 20-record pack (zero-spend dry-run **or** gated paid provider comparison with Perplexity + OpenAI)

All runs write evidence to Google Sheets and enforce hard **no-send** boundaries.

---

## Workflow Overview

![Full Workflow](assets/Full%20View.png)

### Part 1 – Trigger & Mode Selection
![Part 1](assets/part%201.png)

### Part 2 – Synthetic Path + Paid Controller Start
![Part 2](assets/part%202.png)

### Part 3 – Synthetic Evidence Writing + Paid Loop
![Part 3](assets/part%203.png)

### Part 4 – Paid Provider Calls & Finalization
![Part 4](assets/part%204.png)

---

## Key Features

- **Synthetic fixtures** – Deterministic 6-record pack covering success and failure paths
- **Historical comparison** – Validates a 20-record input pack; paid path is gated and canary-controlled
- **Cost & call controls** – Configurable `ai_call_cap`, `cost_cap_usd`, and batch limits
- **Pre-call checkpoints** – Durable state written before any paid provider call
- **Evidence trail** – Ledger upserts, immutable proof events, and run summaries
- **No-send safety** – Explicit asserts that `pushed_ghl`, `pushed_instantly`, and sending flags remain disabled
- **Identity & DNC handling** – Email routing, suppression, and DNC evaluation logic

---

## Architecture
Manual Trigger
└── Select Locked Run Mode
├── Synthetic Mode
│       ├── Synthetic Fixture Pack (6 records)
│       ├── Read Ledger / Runs / Proof Contracts
│       ├── Controlled Core State Machine (pure JS)
│       └── Emit + Upsert Ledger / Proof / Run Evidence
│
└── Comparison Mode
├── Dry-run (20 records, zero spend)
└── Paid path (gated)
├── Initialize Paid Comparison Controller
├── Next Action → Pre-call Checkpoint
├── Perplexity (research) or OpenAI (score/draft)
├── Validate & Apply Result
└── Final Results + Safety Asserts


The core state machine is pure JavaScript and performs no I/O. Persistence and external calls are handled by surrounding n8n nodes.

---

## Modes

| Mode                              | Purpose                                      | Spend     | Sending   |
|-----------------------------------|----------------------------------------------|-----------|-----------|
| `internal-synthetic-integration`  | Full state-machine coverage with mocks       | None      | Disabled  |
| Historical dry-run                | Validate 20-record pack & contracts          | None      | Disabled  |
| `approved-paid-comparison`        | Live Perplexity + OpenAI comparison          | Gated + canary | Disabled |

Paid mode requires explicit authorization flags, a canary record, and human release after canary success.

---

## Safety Guarantees

- External push flags (`pushed_ghl`, `pushed_instantly`) are forced `false` and asserted
- Sending is never enabled in these paths
- Phone material is rejected or treated as non-callable in Phase 1 (email-only)
- Duplicate / historic / suppressed identities are blocked
- Call and spend caps are enforced before every provider invocation
- Malformed provider responses quarantine the record instead of proceeding

---

## Google Sheets Contracts

The workflow expects a spreadsheet with (at minimum) these sheets:

- `ledger` – per-lead checkpoint state
- `runs` – run-level evidence
- `proof_events` – immutable per-input outcomes
- Comparison input / result sheets (for the 20-record path)

Header contracts are validated at runtime; mismatches refuse to write.

---

## Configuration Notes

- Workflow is **inactive** by default in the export
- Paid-provider nodes exist but do **not** authorize spend by themselves
- Synthetic mode uses embedded mocks only
- Prompt version and evidence class are recorded on every run

---

## Requirements

- n8n instance with Google Sheets OAuth2 credentials
- (Paid path only) Perplexity and OpenAI credentials
- Access to the HBA Outreach Ledger Google Spreadsheet

---

## Status

**Draft / Phase 1 Controlled Core**  
Synthetic and comparison validation paths are implemented.  
No production outreach sending path is connected.

---

## License

Private / internal use. Update this section as needed before publishing.
