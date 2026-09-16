# NextAppStore - System Architecture & AI Harness (Root Level)

This document defines the overarching architectural invariants, security rules, and verification loops for all AI coding agents and human developers working across the NextAppStore organization.

---

## 1. System Invariants & Architectural Boundaries

---

## 2. Shared Contracts (Single Source of Truth)

- **REST API (Backend <-> Frontend):**
  - Contract: `backend/openapi.json`
  - The Vue frontend consumes endpoints defined in this schema. Endpoints and TypeScript types must be generated/aligned against this file.

---

## 3. Security & Zero-Trust Guardrails

- **Zero Secret Persistence:**
  - Never commit credentials, private SSH keys, Moodle client secrets, or `clouds.yaml` files to version control.

---

## 4. Cascaded Sub-Harness Guidelines

Agents must navigate to the target sub-repository and respect its local `AGENTS.md` before generating code:

- **Backend (`backend/AGENTS.md`):**

- **Frontend (`frontend/AGENTS.md`):**

- **Worker (`worker/AGENTS.md`):**

- **Deployment (`deployment/AGENTS.md`):**

---

## 5. Agentic Verification & Self-Correction Loop
