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

- **Mandatory Sandbox Execution (Strict Host Isolation):**
  - **NEVER** run development, build, test, database, or package-management commands directly on the host machine. This includes: `pytest`, `ruff`, `poetry`, `npm`, `terraform`, `service postgresql`, `psql`, or `python3` (outside of the harness runner).
  - **ALL** code verification, linting, migrations, and test runs MUST be executed inside the sandbox via `harness/sandbox.py` or `harness/run.sh`.
  - Only non-destructive host commands for inspecting git state (e.g. `git status`, `git diff`) are permitted on the host.

---

## 4. Cascaded Sub-Harness Guidelines

Agents must navigate to the target sub-repository and respect its local `AGENTS.md` before generating code:

- **Backend (`backend/AGENTS.md`):**

- **Frontend (`frontend/AGENTS.md`):**

- **Worker (`worker/AGENTS.md`):**

- **Deployment (`deployment/AGENTS.md`):**

---

## 5. Agentic Verification & Sandbox Usage Guide

### How to Execute Commands in the Sandbox
The sandbox runs in an isolated Docker container (`nextappstore-sandbox:latest`) with Python 3.12, Poetry dependencies, PostgreSQL, and Terraform pre-installed. The repository is mounted at `/workspace`.

- **Fast Verification (Linters + Fast Unit Tests, ~8s):**
  ```bash
  python3 harness/sandbox.py --fast
  # or: ./harness/run.sh --fast
  ```

- **Full Verification (All unit & integration tests + templates check):**
  ```bash
  python3 harness/sandbox.py --full
  # or: ./harness/run.sh --full
  ```

- **Targeted Test Execution (Single file, directory, or test name):**
  ```bash
  python3 harness/sandbox.py pytest backend/tests/unit/test_crypto.py
  python3 harness/sandbox.py pytest -k "test_jwt"
  ```

- **Linting & Code Quality:**
  ```bash
  python3 harness/sandbox.py ruff check backend/
  ```

- **Ephemeral Mode (Guarantees zero host file modifications):**
  ```bash
  python3 harness/sandbox.py --ephemeral --fast
  ```

### Self-Correction Loop
Before declaring any task complete:
1. Run `python3 harness/sandbox.py --fast` (or targeted test for the component modified).
2. If the exit code is non-zero, inspect the `stdout` and `stderr` output from the sandbox.
3. Diagnose the root cause, apply targeted fixes, and re-run until all checks exit with `PASSED (exit code: 0)`.
4. Run `python3 harness/sandbox.py --full` before finalizing pull requests or major features.

