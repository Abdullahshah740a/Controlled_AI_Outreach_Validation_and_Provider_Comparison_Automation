# Controlled AI Outreach Validation and Provider Comparison Automation

**HBA Phase 1 — Controlled Core (DRAFT)**

n8n workflow that validates lead identity handling, AI research, scoring, pathway selection, and outreach draft generation **before** connecting the process to any real outreach sending path.

This project is designed for safe, controlled testing of the core logic using:

- **Synthetic fixtures** (6 records)
- **Historical comparison** (20-record pack)

All runs write full evidence to Google Sheets and enforce hard **no-send** boundaries.

---

## Workflow Screenshots

### Full Workflow View
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

## Functionalities

- **Synthetic fixtures**  
  Includes six records that exercise qualified, below-threshold, suppressed, duplicate, and malformed-input outcomes.

- **Twenty-record comparison**  
  Validates a historical input pack and compares provider outputs through separately gated paid or dry-run paths.

- **Cost and evidence controls**  
  Checks call and spending limits, saves pre-call checkpoints, and writes ledger, proof, and run evidence.

- **No-send boundaries**  
  Keeps external GoHighLevel pushes, Instantly pushes, and sending flags disabled for all comparison runs.

---

## High-Level Workflow

1. Select the locked manual run mode.
2. Read the input and saved evidence contracts.
3. Validate synthetic or comparison records.
4. Run only permitted provider steps with checkpoints.
5. Persist comparison results and assert the no-send boundaries.

**Configuration status:** Inactive in the export. Inactive draft. Paid-provider operations are a separate gated branch; the existence of research and model nodes does not itself authorize spend or sending.

---

## Architecture

```text
Manual Trigger
└── Select Locked Run Mode
    ├── Synthetic Mode
    │   ├── Synthetic Fixture Pack (6 records)
    │   ├── Read Ledger / Runs / Proof Contracts
    │   ├── Controlled Core State Machine (pure JS)
    │   └── Emit + Upsert Ledger / Proof / Run Evidence
    │
    └── Comparison Mode
        ├── Dry-run (20 records, zero spend)
        └── Paid path (gated)
            ├── Initialize Paid Comparison Controller
            ├── Next Action → Pre-call Checkpoint
            ├── Perplexity (research) or OpenAI (score/draft)
            ├── Validate & Apply Result
            └── Final Results + Safety Asserts
```

The core state machine is pure JavaScript and performs no I/O. Persistence and external calls are handled by surrounding n8n nodes.

---

## Modes

| Mode                              | Purpose                                      | Spend          | Sending   |
|-----------------------------------|----------------------------------------------|----------------|-----------|
| `internal-synthetic-integration`  | Full state-machine coverage with mocks       | None           | Disabled  |
| Historical dry-run                | Validate 20-record pack & contracts          | None           | Disabled  |
| `approved-paid-comparison`        | Live Perplexity + OpenAI comparison          | Gated + canary | Disabled  |

Paid mode requires explicit authorization flags, a canary record, and human release after canary success.

---

## Key Features

- Deterministic synthetic test pack (6 records)
- Historical 20-record comparison pack
- Configurable `ai_call_cap`, `cost_cap_usd`, and batch limits
- Pre-call durable checkpoints before any paid API call
- Full evidence trail (ledger, proof events, run summaries)
- Hard no-send safety asserts (`pushed_ghl`, `pushed_instantly`, sending flags)
- Email identity routing + suppression + DNC evaluation
- Pathway selection with tie detection and quarantine
- Output hashing and provenance tracking

---

## Safety Guarantees

- External push flags (`pushed_ghl`, `pushed_instantly`) are forced `false` and asserted at the end of every run
- Sending is never enabled in these paths
- Phone material is rejected or treated as non-callable (Phase 1 is email-only)
- Duplicate / historic / suppressed identities are blocked
- Call and spend caps are enforced **before** every provider invocation
- Malformed provider responses quarantine the record instead of proceeding
- Canary gate prevents the rest of the paid batch from running until the canary passes and is released

---

## Google Sheets Contracts

The workflow expects a spreadsheet with (at minimum) these sheets:

| Sheet            | Purpose                                      |
|------------------|----------------------------------------------|
| `ledger`         | Per-lead checkpoint state                    |
| `runs`           | Run-level evidence                           |
| `proof_events`   | Immutable per-input outcomes                 |
| Comparison sheets| Input pack + result storage (20-record path) |

Header contracts are validated at runtime. Any mismatch causes the workflow to refuse writing.

---

## Tools Used

| Tool              | Role in the project                          |
|-------------------|----------------------------------------------|
| n8n               | Manual mode routing and comparison orchestration |
| Google Sheets     | Ledger, comparison inputs, and evidence storage |
| JavaScript        | State machine, budgets, and result validation |
| Perplexity API    | Research / search provider (paid path only)  |
| OpenAI API        | Scoring and draft-generation provider (paid path only) |

---

## Configuration Notes

- Workflow is **inactive** by default in the export
- Paid-provider nodes exist but do **not** authorize spend by themselves
- Synthetic mode uses embedded mocks only (zero external cost)
- Prompt version and evidence class are recorded on every run
- All paid operations are protected by:
  - Explicit authorization flag
  - Canary record
  - Human release step after canary success

---

## Requirements

- n8n instance
- Google Sheets OAuth2 credentials
- Access to the HBA Outreach Ledger Google Spreadsheet
- (Paid path only) Perplexity and OpenAI credentials

---

## Status

**Draft / Phase 1 Controlled Core**

- Synthetic validation path → Implemented
- Historical comparison (dry-run + gated paid) → Implemented
- Production outreach sending path → **Not connected**

This workflow is intentionally designed as a safe testing and validation layer before any live outreach system is activated.
