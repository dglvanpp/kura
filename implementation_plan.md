# Architectural Blueprint & Consolidation Plan: Project VERA / KURA

Enterprise-grade consolidation of 5-6 hackathon codebases (`AgentBuyer`, `clearwave`, `Hackatoons-NextWaveHackathon`, `nextwave-2026`, `nextwave-hackathon`, `hackatonyuno`) into a unified, modular, autonomous backend monorepo (`/vera-kura-core`).

## User Review Required

> [!IMPORTANT]
> **Zero External Telecom / SMS Dependency (Eliminating Twilio):**
> - All SMS-based 2FA is replaced by **RFC 6238 TOTP** (Google/Microsoft Authenticator compatible) and **Native WebAuthn / FIDO2** (Passkeys, TouchID, FaceID, Android `BiometricPrompt`).
> - High-value approvals are routed through secure, interactive Webhook bots (**Telegram Bot API / WhatsApp Cloud API**) using signed cryptographic callback payloads.

> [!IMPORTANT]
> **Strict Fail-Closed Architecture (ISO/IEC 27001 Alignment):**
> - Every authorization check (biometrics, spending budget, merchant policy, semantic firewall) defaults to `REJECT` / `ABORT`.
> - If any subsystem (biometrics timeout, signature mismatch, token expiry, anomalous rate drop) encounters an error, the transaction terminates immediately without partial commits.

> [!IMPORTANT]
> **Microsoft Fabric / DP-700 Audit Ledger:**
> - Structured JSONL audit logs designed with Delta Lake / Parquet ingestion schema (ISO 27001 A.12.4 compliance), containing immutable event IDs, cryptographic hashes, tenant IDs, and delegation signatures.

---

## 1. Executive Summary & Dual-Branch Vision

**VERA / KURA** bridges autonomous AI agency with enterprise payment governance:

```mermaid
graph TD
    subgraph Client Surfaces
        WEB[🌐 Web Console]
        MOB_AND[🤖 Android Native App]
        MOB_IOS[🍏 iOS Native App]
        BOT[💬 Telegram / WhatsApp Bot]
    end

    subgraph API Gateway
        API[⚡ FastAPI Async REST & Webhooks]
        MW[🛡️ Security Middleware & Rate Limiting]
    end

    subgraph Branch 1: Saturday (B2C Agency)
        SAT[🤖 Saturday Autonomous Agent]
        VOLTA[🎙️ Volta Realtime Procurement Engine]
        NAUTA[📊 Self-Explaining Operational Interface]
        FIREWALL[🔥 Semantic Commerce Firewall]
    end

    subgraph Branch 2: Kura (B2B Infrastructure)
        DELEG[📜 Cryptographic Delegation & Ed25519 Tokens]
        PASSKEY[🔑 WebAuthn / FIDO2 & RFC 6238 TOTP]
        DLP[💳 Scoped Virtual Tokens PCI-DSS DLP]
        CENTINEL[📡 Centinel Multidimensional Diagnostics]
    end

    subgraph Shared Core & Enterprise Governance
        FAILCLOSED[🛑 Fail-Closed Decision Engine]
        AUDIT[🏛️ Immutable Audit Ledger DP-700 Ready]
        ORCH[🔀 Multi-Gateway Orchestrator Stripe/Yuno]
    end

    WEB & MOB_AND & MOB_IOS & BOT --> API
    API --> MW
    MW --> FAILCLOSED
    FAILCLOSED --> SAT & DELEG
    SAT --> FIREWALL & VOLTA & NAUTA
    DELEG --> PASSKEY & DLP & CENTINEL
    FIREWALL & CENTINEL --> ORCH
    FAILCLOSED & SAT & DELEG & ORCH --> AUDIT
```

---

## 2. Source Repositories & Extracted Capabilities

| Repository | Extracted Capabilities & Architecture Contributions | Target Module in `/vera-kura-core` |
|---|---|---|
| **`AgentBuyer`** (Saturday) | Autonomous purchase loop, Zero-Trust 4-step wizard, Ed25519 mandate issuance, DLP virtual tokenization, SMTP/IMAP notifications, Android & iOS bindings | `src/agents/saturday`, `src/security/tokens.py`, `src/security/notifications.py` |
| **`clearwave`** | Payment anomaly telemetry, ground-truth transaction simulation, evaluation engine, signed checkout boundaries | `src/orchestrator/telemetry.py`, `src/orchestrator/evaluator.py` |
| **`Hackatoons`** (Centinel) | Multidimensional payment diagnosis loop, root-cause segmentation (issuer/market/method), revenue at risk calculation | `src/orchestrator/centinel.py`, `src/orchestrator/diagnostics.py` |
| **`nextwave-2026`** (Marketline / Volta) | Deterministic procurement policy engine, Pareto ranking of multi-provider offers, immutable state machine | `src/agents/volta/`, `src/core/models/market.py` |
| **`nextwave-hackathon`** (Donald / Nauta) | "Interface that builds itself", live agent explanation pipeline, operational timeline event streaming | `src/agents/nauta/`, `src/api/routes/timeline.py` |
| **`hackatonyuno`** | Multi-gateway routing patterns, SDK interfaces, Supabase/SQL database schemas | `src/orchestrator/routing.py`, `src/core/db.py` |

---

## 3. Modular Monorepo Architecture (`/vera-kura-core`)

We will build the repository at `C:\Users\Angy2\.gemini\antigravity\scratch\vera-kura-core` with the following clean architecture:

```text
/vera-kura-core
├── README.md
├── pyproject.toml / requirements.txt
├── .env.example
├── /src
│   ├── /core
│   │   ├── config.py                 # Pydantic v2 settings (Azure, Keys, JWT, Env)
│   │   ├── db.py                     # Async SQLAlchemy database session engine
│   │   ├── models/                   # Core domain models
│   │   │   ├── mandate.py            # Delegation mandates, constraints, validity
│   │   │   ├── transaction.py        # Autonomous checkout attempts & ledger
│   │   │   └── audit.py              # ISO 27001 / DP-700 audit event schemas
│   │   └── audit_ledger.py           # Immutable JSONL + Hash-chained event logger
│   │
│   ├── /security
│   │   ├── fail_closed.py            # Strict Fail-Closed evaluation gatekeeper
│   │   ├── webauthn.py               # FIDO2 / WebAuthn platform biometrics verifier
│   │   ├── totp.py                   # RFC 6238 TOTP generator and strict validator
│   │   ├── signatures.py             # Ed25519 cryptographic token signing & verifying
│   │   ├── token_vault.py            # Scoped Virtual Token PCI DLP vault
│   │   └── bot_approval.py           # Interactive Telegram/WhatsApp approval webhooks
│   │
│   ├── /orchestrator
│   │   ├── router.py                 # Smart payment gateway routing engine
│   │   ├── centinel.py               # Multidimensional failure & approval rate diagnosis
│   │   ├── delegation_rules.py       # Corporate employee & bot spend limit enforcement
│   │   └── connectors/               # Pluggable payment processors (Stripe, Yuno mock)
│   │       ├── base.py
│   │       ├── stripe_connector.py
│   │       └── yuno_connector.py
│   │
│   ├── /agents
│   │   ├── saturday/                 # Branch 1: Autonomous consumer purchasing agent
│   │   │   ├── loop.py               # Search -> Validate -> Firewall -> Execute loop
│   │   │   ├── semantic_firewall.py  # LLM dark pattern & hidden fee detector
│   │   │   └── search_provider.py    # Multi-merchant search aggregator
│   │   ├── volta/                    # Deterministic procurement & Pareto bid evaluator
│   │   │   └── policy.py
│   │   └── nauta/                    # Self-explaining agent activity timeline generator
│   │       └── timeline.py
│   │
│   └── /api
│       ├── main.py                   # FastAPI async app factory & CORS
│       ├── middleware/               # Zero-Trust auth & rate limiting
│       │   └── zero_trust.py
│       └── routes/                   # Clean RESTful API endpoints
│           ├── mandates.py           # B2B & B2C delegation mandate lifecycle
│           ├── auth.py               # WebAuthn registration/login & TOTP validation
│           ├── transactions.py       # Checkout execution & approval flow
│           ├── agent.py              # Saturday autonomous run triggers & streaming
│           ├── diagnostics.py        # Centinel payment health & incident monitoring
│           ├── webhooks.py           # Telegram/WhatsApp interactive bot callbacks
│           └── audit.py              # DP-700 / Fabric exportable audit logs
│
└── /tests
    ├── conftest.py                   # Shared fixtures & async test client
    ├── test_fail_closed.py           # Security stress tests proving Fail-Closed default
    ├── test_crypto_delegation.py     # Ed25519 token issuance & tampering detection
    ├── test_totp_webauthn.py         # RFC 6238 and FIDO2 authentication flow
    ├── test_centinel_diagnostics.py  # Anomaly detection & revenue at risk checks
    └── test_saturday_agent.py        # End-to-end autonomous purchase loop
```

---

## 4. Implementation Steps & Milestones

### Phase 1: Environment, Core Domain & Database Models
- Initialize `/vera-kura-core` with Python 3.11+, Pydantic v2, FastAPI, Cryptography, PyNaCl (Ed25519), PyOTP (RFC 6238), WebAuthn.
- Setup `src/core/config.py`, `src/core/db.py`, and domain models (`Mandate`, `Transaction`, `AuditEvent`).
- Implement `src/core/audit_ledger.py` with DP-700 compliant structured JSONL records and hash chaining.

### Phase 2: Security & Autonomous Authentication (Zero Third-Party SMS)
- Build `src/security/totp.py` with RFC 6238 time-based OTP generation, URI QR provisioning, and drift tolerance.
- Build `src/security/webauthn.py` implementing FIDO2 assertion and attestation challenges.
- Build `src/security/signatures.py` with Ed25519 public/private key delegation tokens.
- Build `src/security/fail_closed.py` ensuring transactions fail immediately if any factor fails.
- Build `src/security/bot_approval.py` for Telegram/WhatsApp webhook signing and interactive approval callbacks.

### Phase 3: Orchestration, Delegation & Centinel Diagnostics
- Port Centinel's multidimensional diagnosis engine (`src/orchestrator/centinel.py`) to detect approval rate drops and segment anomalies by gateway, currency, and merchant.
- Implement corporate delegation rules in `src/orchestrator/delegation_rules.py`.
- Build pluggable payment connectors (Stripe / Yuno mock) in `src/orchestrator/connectors/`.

### Phase 4: Autonomous Agents (Saturday + Volta + Nauta)
- Consolidate Saturday's agent loop (`src/agents/saturday/loop.py`) and Semantic Firewall (`src/agents/saturday/semantic_firewall.py`).
- Integrate Volta's Pareto optimization and Nauta's self-explaining timeline event generator (`src/agents/nauta/timeline.py`).

### Phase 5: FastAPI REST Endpoints & Webhooks
- Implement all routes in `src/api/routes/` (`mandates`, `auth`, `transactions`, `agent`, `diagnostics`, `webhooks`, `audit`).
- Configure OpenAPI documentation, Swagger UI, and middleware.

### Phase 6: Automated Verification & Testing Suite
- Run comprehensive pytest suite verifying:
  1. Fail-Closed security behavior under network/timeout/tampering faults.
  2. Ed25519 cryptographic mandate delegation.
  3. TOTP generation & verification.
  4. Centinel diagnostic segmentation.
  5. Saturday autonomous agent execution.

---

## 5. Verification Plan

### Automated Tests
- Command: `pytest tests/ -v`
- Test cases:
  - `tests/test_fail_closed.py`: Proves that any tampered signature, expired TOTP, or missing biometric confirmation rejects immediately with HTTP 401/403.
  - `tests/test_crypto_delegation.py`: Validates Ed25519 signed mandate issuance, verification, and tamper-resistance.
  - `tests/test_totp_webauthn.py`: Verifies RFC 6238 TOTP codes and WebAuthn credential assertion.
  - `tests/test_centinel_diagnostics.py`: Evaluates synthetic transaction cubes and verifies that anomaly detection identifies the localized problem segment and revenue at risk.
  - `tests/test_saturday_agent.py`: Executes the autonomous agent loop from search to semantic firewall check and approval receipt.

### Manual Verification & Inspection
- Launch FastAPI locally: `uvicorn src.api.main:app --reload --port 8000`
- Access interactive documentation: `http://127.0.0.1:8000/docs`
- Inspect DP-700 audit log exports: `http://127.0.0.1:8000/audit/events`
