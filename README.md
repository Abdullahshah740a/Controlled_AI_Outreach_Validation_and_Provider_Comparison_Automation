# Controlled AI Outreach Validation and Provider Comparison Automation

**HBA Phase 1 — Controlled Core (DRAFT)**

n8n workflow for controlled validation of AI-powered lead research, scoring, pathway selection, and outreach draft generation — **before** any real sending path is connected.

It supports two locked modes:

- **Synthetic mode** → 6 controlled fixture records (qualified, below-threshold, suppressed, duplicate, malformed)
- **Historical comparison mode** → 20-record pack (zero-spend dry-run **or** gated paid provider comparison with Perplexity + OpenAI)

All runs write evidence to Google Sheets and enforce hard **no-send** boundaries.

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
