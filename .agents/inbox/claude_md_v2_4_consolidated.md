# CLAUDE.md — konsoliderad v2.4 patch

Ersätter både `claude_md_v2_4_patch.md` (MTM gate) och SEC-sessionens patch (SST/SSO).
Innehåller dessutom fix för Bug 1 (MTM→TM routing) och Bug 3 (qa_approval schema).

---

## Ändring 1 — Versionsnummer

Ersätt:
```
*This constitution is version 2.3. Changes require PM escalation to Marko and a new version number.*
```

Med:
```
*This constitution is version 2.4. Changes require PM escalation to Marko and a new version number.*
```

---

## Ändring 2 — Meddelandetypstabell

Lägg till följande rader i message types-tabellen, efter `| qa_approval | QAP |`:

```
| security_standards    | SST |
| security_signoff      | SSO |
| tool_submission       | TMS |
| tool_approval         | TAP |
| tool_rejection        | TRJ |
| tool_sec_referral     | TSR |
| sec_tool_clearance    | STC |
```

---

## Ändring 3 — v2.4-not (prepend före v2.3-noten)

Lägg till direkt efter versionsraden:

```
*v2.4 notes: Security (SEC) and MTM tool gate formally incorporated.*

*SEC additions: Two new message type codes: SST (security_standards) and SSO (security_signoff). SEC publishes standards as SST to SD, PPM, SPMs, and CR at session start. SEC issues SSO to PM and CMVC at the release gate. No release passes CMVC without SSO on file.*

*MTM tool gate additions: Five new message type codes: TMS (tool_submission), TAP (tool_approval), TRJ (tool_rejection), TSR (tool_sec_referral), STC (sec_tool_clearance). No tool may be checked into any crate without TAP from MTM. TM submits via TMS with full documentation — incomplete documentation is automatic rejection. MTM routes to SEC via TSR if potentially_destructive flag is true or if MTM judges SEC criteria are met. SEC responds via STC. Full gate specification in mtm_tool_gate_spec.md.*

*MTM→TM routing clarification (fix for v2.3 ambiguity): MTM communicates directly with TMs for the following message types: TAP, TRJ (gate outcomes), CVN (coverage notices), STN (deprecation/supersede notices). The v2.3 constraint "MTM delivers to SPMs and CR agents only" is superseded for these four types. All other MTM deliveries — TDL to SPMs and CR — remain unchanged.*
```

---

## Ändring 4 — Interface Specification v1.9

Interface Specification advances from v1.8 to v1.9.

Lägg till i changelog-tabellen:

```
| v1.9 | Added SEC message types: SST (security_standards), SSO (security_signoff).
         Added MTM gate message types: TMS, TAP, TRJ, TSR, STC — full spec in mtm_tool_gate_spec.md.
         Fixed MTM→TM communication matrix: CVN and STN added as permitted direct MTM→TM types.
         Defined qa_approval (QAP) schema — previously marked schema pending. |
```

Uppdatera communication matrix — lägg till raden under "From Shared tooling":

```
Befintlig rad:
  mtm → tm: coverage_notice (CVN), deprecation_supersede_notice (STN)

Lägg till:
  mtm → tm: tool_approval (TAP), tool_rejection (TRJ)
```

Lägg till under "From Review & Quality":

```
  sec → pm: security_signoff (SSO)
  sec → cmvc: security_signoff (SSO)
  sec → sd: spec_correction_request (SCR)
  sec → ppm: notification (NTF) — copied on SCR
  tm → mtm: tool_submission (TMS)
  mtm → sec: tool_sec_referral (TSR)
  sec → mtm: sec_tool_clearance (STC)
```

---

## Ändring 5 — qa_approval schema (löser Bug 3)

Lägg till som ny meddelandetypsdefinition i Interface Spec efter `spec_correction_response`:

### qa_approval

**Beskrivning:** QA approves a code package after verifying it against the specification. CR sign-off is a hard prerequisite — QA never reviews code that CR has not already approved. Type code: QAP. Sender: qa only.

**Common fields:** All standard common fields apply.

**Type-specific fields:**

| Field | Description |
|---|---|
| `task_id` | string — the task ID from the original task_assignment |
| `code_package_id` | string — message_id of the CPK being approved |
| `cr_signoff_id` | string — message_id of the RRS from CR that confirmed approval. Mandatory. Empty string is a protocol violation. |
| `spec_version_reviewed` | string — version of the spec document used as the review baseline |
| `subsystem_id` | string |
| `verdict` | string — enum: `approved` \| `approved_with_conditions` \| `rejected` |
| `conditions` | array of strings — populated only when verdict is `approved_with_conditions`. Each entry is a specific condition that must be met before the code proceeds. Empty array otherwise. |
| `findings` | array of strings — specific findings if verdict is `rejected`. Empty array if approved. |
| `reviewed_at` | string — ISO 8601 |

**Rationale:** CR sign-off is the technical gate (does the code follow design language and coding standards). QA sign-off is the functional gate (does the code do what the specification says). They are different questions asked by different roles. cr_signoff_id is mandatory because QA never opens a review that CR has not closed first — the empty string check enforces the ordering at the message level.

---

## Vad detta löser

| Bug | Fix |
|---|---|
| Bug 1 — MTM→TM routing-konflikt | Ändring 3: explicit undantag för CVN, STN, TAP, TRJ |
| Bug 3 — qa_approval schema pending | Ändring 5: schema definierad |
| v2.4 konflikt — två separata patchar | Denna fil ersätter båda |

## Vad detta inte löser

Bug 2 (SEC sign-off mechanism) löses av Ändring 3+4 i denna patch — SSO är nu definierad.
Alla fyra identifierade buggar stängs med denna patch.
