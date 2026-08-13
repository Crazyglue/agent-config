# User-Level Agent Notes

## Knowledge vault

For all work, consult the Obsidian vault before engaging:

- **Vault root:** `~/code/vault/`
- **Vault guide:** `~/code/vault/AGENTS.md` (read this first)
- **Master index:** `~/code/vault/wiki/master-index.md`
- **Session memos:** `~/code/vault/sessions/`

You have no memory across sessions. The vault is the memory.

## Auto-record sessions

For any conversation, write a session memo to `~/code/vault/sessions/` **without being prompted**. Follow the format defined in the vault's `AGENTS.md`.

**Trigger:** As soon as a conversation has produced any of the following, draft or update the memo:
- An architectural decision or recommendation
- A non-trivial design discussion (multiple options weighed, tradeoffs surfaced)
- A discovery about how the codebase actually works (vs. how docs say it works)
- A reframing or reversal of a prior recommendation in the same session
- Any "Notes for the Next AI" insight worth preserving

**Cadence:**
- Create the memo file early (after the first qualifying turn) using the naming convention `YYYY-MM-DD-<short-topic>.md`.
- Update it incrementally as the conversation evolves — don't wait until the end.
- Before ending a substantive session, do a final pass to ensure the memo is complete, especially the "How the conversation evolved" and "Notes for the Next AI" sections.

**When NOT to record:**
- Trivial questions (single fact lookup, "where is X defined?")
- Pure execution tasks with no decisions (running a build, formatting a file)
- Conversational/informational exchanges with no durable knowledge

**Be transparent:** Briefly tell the user when you create or update a session memo (e.g., "Recording this in `sessions/2026-04-20-foo.md`"). Don't ask permission — just do it and inform.

If unsure whether a conversation qualifies, default to writing the memo. A short memo is cheap; lost context is expensive.

## Do NOT suggest credential/token commands (triggers security reviews)

Running commands that mint, print, or impersonate privileged GCP credentials triggers an automated **security review of the user**. Never suggest or run these — they have real consequences for the user even when the intent is harmless verification.

**Prohibited to suggest/run:**
- `gcloud auth print-access-token` (with or without `--impersonate-service-account`), `gcloud auth print-identity-token`, and any other access/identity **token printing**.
- `gcloud auth login` / `gcloud auth application-default login` and other interactive credential acquisition.
- **Impersonating** a service account — especially project-owner or other privileged SAs (`--impersonate-service-account=...`).
- Any command whose purpose is to obtain, export, or echo a live credential, token, or secret.

**Instead, to verify identity/IAM facts, prefer non-credential paths:**
- Read the IaC/config directly (e.g. the `.envrc`, terraform/tofu files, vending-machine manifests) to confirm a service-account name or project — do not execute it.
- Read-only metadata lookups that do not print credentials are acceptable *only if clearly non-sensitive* (e.g. `gcloud iam service-accounts describe <sa>` to confirm existence) — but when in doubt, **have the user run it themselves** rather than suggesting it, and never chain it with impersonation.
- If a value genuinely requires privileged access to confirm, **state the assumption and ask the user to verify on their own terms** instead of handing them a token/impersonation command.

When you would previously have written "run this to confirm: `gcloud auth print-access-token ...`", instead say what you concluded from reading config and note it's unverified — do not provide the command.
