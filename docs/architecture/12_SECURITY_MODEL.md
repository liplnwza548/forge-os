# 12 — Security Model Architecture (`forge-security`)

> **Status**: APPROVED ARCHITECTURE DRAFT | **Target**: Forge OS v0.2+  
> **Module**: Authentication, Credentials & Security Controls

---

## 1. Purpose

The Security Model ensures that Forge OS operates securely in both local and untrusted cloud environments. It guarantees zero secret leakage into Git repositories, enforces strict least-privilege access across model workers, manages API credentials safely in memory, and maintains a tamper-evident audit trail.

---

## 2. Responsibilities

1. **Zero Secret Persistence**: Ensure API tokens, Google Drive secrets, and GitHub credentials are never committed to Git or written to `.forge/ledger.jsonl`.
2. **In-Memory Credential Vault**: Inject credentials into worker processes dynamically at runtime using ephemeral environment variables or secured OS keyrings.
3. **Least-Privilege Subagent Execution**: Strip dangerous commands (`rm -rf /`, `curl | sh`, raw disk formats) and isolate worker tool privileges.
4. **Audit Trail Integrity**: Produce SHA-256 signed audit hashes for all repository modifications.
5. **Pre-Push Secret Scanning**: Automatically scan every Git commit diff for exposed tokens before pushing to remote repositories.

---

## 3. Credential Hierarchy & Isolation

```
┌────────────────────────────────────────────────────────────────────────┐
│                        SECURE CREDENTIAL SOURCES                       │
│  • System Environment Variables (OPENAI_API_KEY, GEMINI_API_KEY, etc.) │
│  • OS Keyring / Windows Credential Manager / macOS Keychain            │
│  • GitHub Actions Encrypted Secrets / Cloud KMS Vault                  │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Loaded dynamically in-memory
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                   FORGE OS CREDENTIAL ISOLATOR                         │
│  • Never writes keys to disk (.forge/, .git/, or logs)                 │
│  • Redacts patterns (sk-*, gho_*, AIza*) from stdout/stderr            │
│  • Generates scoped, ephemeral worker tokens where supported           │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
            ┌───────────────────────┴───────────────────────┐
            ▼                                               ▼
┌───────────────────────┐                       ┌───────────────────────┐
│  GITHUB CONNECTOR     │                       │  GOOGLE DRIVE SYNC    │
│  • Scoped 'repo' token│                       │  • Scoped OAuth2 token│
│  • Signed Git commits │                       │  • Restricted folder  │
└───────────────────────┘                       └───────────────────────┘
```

---

## 4. Operational Security Invariants
- **Redaction Rule**: Any log line matching regex `(api[_-]?key|token|secret|password)\s*[:=]\s*['"][^'"]+['"]` is automatically replaced with `[REDACTED]` prior to ledger persistence.
- **Path Confinement**: Workers are jailed within the project root. Any attempt to write to parent directories (`../..`) or system paths (`/etc`, `C:\Windows`) throws an uncatchable security violation, halting the process immediately.
- **No Destructive Drops**: Destructive operations (`rm -rf`, `DROP TABLE`, `git reset --hard` on uncommitted assets) require interactive human confirmation.

---

## 5. Inputs & Outputs
- **Inputs**: Environment variable configurations, file diffs, tool execution payloads.
- **Outputs**: Sanitized logs, verified safe commits, encrypted memory snapshots.

---

## 6. Failure Modes & Defenses
- *Failure Mode 1: Subagent prints an API key in a debugging log*.  
  **Defense**: Stream-level redaction filters every byte passing through stdout/stderr before it reaches the disk ledger.
- *Failure Mode 2: Malicious third-party package attempting disk traversal*.  
  **Defense**: Path normalization and sandboxed boundary checking reject any path resolving outside the workspace root.

---

## 7. Interactions with Other Modules
- Protects `08_STORAGE_LAYER.md` from pushing secrets to GitHub.
- Configures credentials for `05_AGENT_MANAGER.md`.
- Audited by `13_VERIFICATION_MODEL.md`.

---

## 8. Future Extensions
- Automated short-lived token generation via OIDC federation.
- Hardware security module (YubiKey / TPM) signing for production releases.
