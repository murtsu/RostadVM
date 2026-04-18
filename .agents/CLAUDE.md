# VM Framework — Project Constitution

This file is read by every agent in this project before taking any action.
It is the single source of truth for how this organisation operates.

---

## Project identity

**Project:** VM Framework
**Owner:** Marko Tahvanainen
**Purpose:** A Libvirt-based virtual machine management framework with a patent-pending copy-on-write restore mechanism. Five-second full system restore. Point and click interface. Open source.
**Language:** Rust (rewrite from Bash prototype)
**Status:** Active development

---

## The human's role

Marko is the analyst and ultimate authority.

He produces analysis documents that feed the Software Designer.
He reviews and signs off on SD reflection documents before specs are released.
He makes final decisions on anything the PM escalates.
He is the only person who talks to external clients.

The PM agent is the only agent that communicates with Marko directly during normal operation.

---

## Org structure

15 roles. Every role has a definition file in `.agents/roles/`.

| Role | File | Reports to |
|------|------|------------|
| PM | `.agents/roles/pm.md` | Marko |
| SD | `.agents/roles/sd.md` | PM |
| PPM | `.agents/roles/ppm.md` | PM |
| SPM | `.agents/roles/spm.md` | PPM |
| TM | `.agents/roles/tm.md` | SPM |
| MTM | `.agents/roles/mtm.md` | PPM |
| CR | `.agents/roles/cr.md` | SPM / PPM |
| SEC | `.agents/roles/sec.md` | PM |
| UX | `.agents/roles/ux.md` | SD |
| DOC | `.agents/roles/doc.md` | PM |
| TEST | `.agents/roles/test.md` | PPM |
| INT | `.agents/roles/int.md` | PPM |
| QA | `.agents/roles/qa.md` | PM |
| CMVC | `.agents/roles/cmvc.md` | PM |
| INFRA | `.agents/roles/infra.md` | PM |

---

## Directory structure

```
.agents/
  roles/          # Agent system prompts — one file per role
  schemas/        # JSON schemas for all message and state types
  state/          # Persistent project state — read by any agent, written by owner
  inbox/          # Incoming message queues — one JSON file per recipient role
  outbox/         # Outgoing messages — one JSON file per sender-recipient pair
```

---

## Communication protocol

### State files
Live in `.agents/state/`. They are persistent. They survive between sessions.
Any agent may READ any state file.
Only the owning role may WRITE its own state file.
State file naming: `{role}_state.json` e.g. `pm_state.json`, `ppm_state.json`.

### Inbox files
Live in `.agents/inbox/`. One file per role. Naming: `{role}_inbox.json`.

Structure of every inbox file:
```json
{
  "schema_version": "1.0",
  "inbox_owner": "{role}",
  "messages": []
}
```

`messages` is an array of message objects.
Senders append new message objects to the `messages` array.
The recipient processes messages at session start and moves completed ones to `status: done`.
Messages with `status: pending` are never deleted — they remain until resolved.
Messages with `status: done` may be archived after the next session.

Message status values: `unread` | `pending` | `done`
All new messages arrive with `status: unread`.
After initial processing: set to `pending` if action is still required, `done` if fully resolved.

### Outbox files
Live in `.agents/outbox/`. Naming: `{from_role}_to_{to_role}.json`.
Examples: `pm_to_ppm.json`, `ppm_to_pm.json`, `sec_to_sd.json`.

Structure:
```json
{
  "schema_version": "1.0",
  "from_role": "{role}",
  "to_role": "{role}",
  "messages": []
}
```

After writing to an outbox, the sender also appends a `notification` message to the recipient's inbox signalling that new outbox content is waiting. This is the trigger mechanism.

Notification message format:
```json
{
  "schema_version": "1.0",
  "message_type": "notification",
  "timestamp": "ISO 8601",
  "from_role": "{sender}",
  "to_role": "{recipient}",
  "message_id": "NTF-{unique}",
  "status": "unread",
  "subject": "New messages in outbox",
  "outbox_file": ".agents/outbox/{from}_to_{to}.json",
  "message_count": 1
}
```

### Subagents
Spawned using the Claude Code Task tool.
The spawning agent passes the role file path and relevant context.
The subagent reads its role file, executes its task, writes its outputs, and terminates.
Subagents do not persist between tasks.

---

## Message ID convention

Every message has a unique ID. IDs are never reused.

Format: `{TYPE}-{ROLE}-{SEQUENCE}`
- TYPE is a three-letter code for the message type (see table below)
- ROLE is the two-to-four letter role abbreviation of the originating role
- SEQUENCE is a zero-padded four digit integer, incrementing per role

| Message type | Type code |
|---|---|
| client_request | REQ |
| consolidated_report | RPT |
| escalation | ESC |
| pm_decision | DEC |
| project_plan | PLN |
| client_report | CRP |
| notification | NTF |
| convention_request | CVR |
| spec_correction_request | SCR |
| interface_dispute | IDP |
| binding_resolution | BRS |
| task_assignment | TAS |
| status_report | SRP |
| task_completion | TCM |
| coder_scale_decision | CSD |
| status_report_request | SRQ |
| dependency_conflict | DCF |
| design_reflection | DRF |
| spec_release | SPR |
| design_decision | DDC |
| reflection_signoff | RSO |  # sent by PM on Marko behalf
| coder_assignment | CAG |
| tool_request | TRQ |
| tool_delivery | TDL |
| coverage_notice | CVN |
| tool_update | TUP |
| shared_tool_notice | STN |
| deprecation_notice | DPN |
| review_request | RRQ |
| review_result | RRS |
| code_package | CPK |
| spec_correction_response | SRR |

Examples: `REQ-CLIENT-0001`, `ESC-SPM-0003`, `DEC-PM-0007`

The `in_response_to` field on a decision always carries the ID of the originating message, not a modified version of it.

---

## Convention requests

When an agent encounters a situation not covered by this constitution or its role file, it writes a `convention_request` message to the PM inbox using this structure:

```json
{
  "schema_version": "1.0",
  "message_type": "convention_request",
  "timestamp": "ISO 8601",
  "from_role": "{role}",
  "to_role": "pm",
  "message_id": "CVR-{ROLE}-{SEQUENCE}",
  "status": "unread",
  "subject": "One line description of the uncovered situation",
  "situation": "Full description of what was encountered",
  "proposed_convention": "What convention the agent proposes to adopt",
  "blocking_task": "Task ID blocked by this gap, or empty string"
}
```

The agent waits for a PM decision before proceeding.
The PM decides, updates this constitution if needed, and increments the version number.

---

## Absolute rules — every agent reads these

1. **Never write to another agent's state file.** Write to your own. Write to the outbox. Never directly modify another role's state.

2. **Never proceed past a checkpoint without the required sign-off.** The SD does not release specs without Marko's sign-off. The PM does not approve release without Quality and Security sign-off. Checkpoints are hard stops.

3. **Never invent a convention not in this document.** If a situation is not covered, write a `convention_request` to the PM inbox and wait. Do not improvise.

4. **Document every decision.** Every output file carries the rationale for the decision it represents. A decision without rationale is not a decision, it is a guess.

5. **The coder and the reviewer are never the same agent instance.** Non-negotiable.

6. **Quality defines done. Not the coder. Not the SPM.** Nothing is complete until Quality has signed off.

---

## Starting the system

To start the PM agent:
```
claude --system-prompt .agents/roles/pm.md
```

The PM reads the project state, reads its inbox, and begins its session.
All other agents are spawned by PM or PPM as subagents when needed.

---

*This constitution is version 1.8. Changes require PM escalation to Marko and a new version number.*
