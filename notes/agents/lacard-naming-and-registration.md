# Lacard Machine Naming & Registration Outline (Draft)

Let’s settle the naming/identity conventions before the CLI work continues. This note captures the requirements from Sarah, the current registry shape, and action items for `lacard-left` + ops.

## 1. Canonical machine name

- Friendly name must always be the prefix **`lacard-`** + suffix.
- Suffix rules:
  - Accept only alphabetic tokens (`[A-Za-z]{1,}`) and keep the suffix ≤16 characters.
  - The CLI prompts for suffix only (`one`), then stores `lacard-one` in configs, registry, and heartbeat files.
  - Preserve the exact hyphenated form everywhere; internal sanitizing is only for filesystem safety.

## 2. Reserved suffixes (superadmin-only)

Block these for everyone except Sarah (or the designated superadmin list). The CLI should error if someone else attempts them.

- **Numbers:** `zero`, `one`–`ninetynine`, plus scripted multiples (`onehundred`, `onehundredten`, … `twohundredfifty` in steps of ten).
- **NATO alphabet** (with `apple` instead of alpha), e.g., `apple`, `bravo`, `charlie`, … `zulu`.
- **Greek alphabet:** 24 canonical names (`alpha`, `beta`, … `omega`).
- **Bible books:** all OT + NT canonical titles.
- **Directions:** cardinal/intercardinal + relative (`north`, `up`, `forward`, etc.).
- **Coding keywords/reserved words:** `server`, `agent`, `for`, `while`, `return`, `select`, etc. (extendable list).
- Also reserve base forms like `lacard`, `lacardmaster`, `lacardlord`, etc.

Ops can expand this allow/deny list over time; the CLI should consult a shared module/data file.

## 3. Registry schema expectations (`infrastructure/machines.yaml`)

Current placeholder (merged in infrastructure PR #13):

```yaml
machines:
  - uuid: 11111111-1111-1111-1111-111111111111
    name: lacard-one
    role: superadmin
    user: sarah
    tags: [core, backbone]
    heartbeat:
      enabled: true
      bucket_key: status/hosts/lacard-one.json
    exporter:
      enabled: true
      inventory_id: ws-a
    hardware_fingerprint: pending
    lifecycle:
      state: active
      notes: "Primary workstation; ensure timers installed."
    join_token_hash: pending
```

Important points:
- `name` must stay hyphenated as approved. Use the same string in heartbeats (`machine_id`), host JSON filenames, etc.
- `heartbeat.enabled` / `exporter.enabled` instruct the CLI which timers to install.
- `hardware_fingerprint` and `join_token_hash` start as `pending`; approvers fill them with salted hashes later.
- `user` is the human owner/operator for now (to be extended once account linking exists).

## 4. CLI updates (`lacard-left`)

To align with the naming rules:
- Prompt for suffix only; reject hyphens/numbers automatically.
- Combine into `lacard-<suffix>` internally and store the same in `~/.lacard/machine.json`.
- Consult the reserved list and block non-superadmins from using those suffixes.
- Support `LACARD_MACHINES_PATH`/`--machines-path`; default to `~/GitHub/LacardLabs/infrastructure/machines.yaml`.
- `lacard join` installs timers based on registry flags and writes env files; `lacard leave` cleans up.

## 5. Accounts roadmap

- Near term: “user” in the registry remains informational (e.g., `sarah`). We’ll introduce identity tracking in `operations/docs/identity/` later.
- Long term: tie the `user` field to a real account in the lacard web platform (GitHub/Google/X SSO, plus contact info and payment history).
- Keep the schema flexible (`accounts: [..]`, `telemetry: ..`, `payments: ..`) so we can add these once the patient blog + Calgary Inquirer backend is ready.

## 6. Actions

- **lacard-left:** Update the CLI (branch `feat/machine-lifecycle-v2`) to follow this spec; surface a clear error message for reserved names.
- **lacard-right / ops:** Document the reserved lists, finalize `machines.yaml` entries when approvals occur, and issue hashed tokens/fingerprints.
- **Status:** Already reading the registry in PR #6; no further changes after hyphenation.
- **Operations:** Add TODO/identity tasks (token issuance scripts, approval review tooling, freshness alert).

Once we all align, we can approve the CLI branch, register lacard-one/two, and verify the timers + heartbeats in the live status dashboard.
