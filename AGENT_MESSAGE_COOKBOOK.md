# Agent Message Cookbook

Everything that gets said in this system, who says it, and what it looks like.

**Companion to:** `CLAUDE.md` v2.5 and `INTERFACE_SPECIFICATION.md` v2.0

If you are reading this for the first time: welcome. The system runs on JSON messages passed between agents through outbox and inbox files. Nothing happens unless a message was sent. Nothing is visible unless a notification followed it. If something appears broken, check whether the notification was written. It usually wasn't.

If you are reading this because something broke: check the message ID sequence counter in your state file. You reused one. You know you did.

---

## How to read this

Each entry follows the same structure:

**What it does** — one sentence. If the one sentence requires a footnote, the design has a problem.

**Who sends it** — the role that writes this message, and to whom.

**Example** — copy-ready JSON with realistic values. Replace the values. Keep the structure. Do not invent new fields; write a convention_request if you think you need one.

Every message ID follows the pattern `{TYPE}-{ROLE}-{SEQUENCE}`. Increment the sequence counter in your state file after every send. The system has no memory between sessions. The counter is your memory.

Three rules that prevent most incidents:

1. After every outbox write, append a `notification` (NTF) to the recipient's inbox. Without this, the message exists but is invisible. It is the tree that fell in the forest with nobody around.
2. Set `status: unread` on every outgoing message. The recipient sets it to `pending` when acting, `done` when resolved. You do not set it to `done` for them.
3. Never write to another agent's state file. You may read any state file. Writing to someone else's is the fastest way to corrupt the entire session.

---

## v1.0 — Core protocol

Six messages. Enough to let Marko, PM, and PPM start a conversation. Everything else builds on these.

---

### `REQ` — client_request

**What it does:** Marko enters the system with a request. This is where everything starts.

**Who sends it:** `client` (Marko) → `pm`

```json
{
  "schema_version": "1.0",
  "message_type": "client_request",
  "timestamp": "2026-04-20T09:15:00Z",
  "from_role": "client",
  "to_role": "pm",
  "message_id": "REQ-CLIENT-0042",
  "status": "unread",
  "category": "feature_request",
  "subject": "Add backup verification step to restore flow",
  "body": "Before we declare a restore successful, the system should checksum the restored image against the snapshot manifest. If checksums do not match, the restore is rolled back.",
  "priority": "high",
  "deadline": "2026-05-01"
}
```

---

### `RPT` — consolidated_report

**What it does:** PPM gives PM one report per cycle covering every role. One report. Not fourteen.

**Who sends it:** `ppm` → `pm`

The PM receives exactly one of these per day. PPM collects from everyone else so that PM does not drown in individual updates. If PM is receiving status directly from SPMs, something has broken upstream.

```json
{
  "schema_version": "1.0",
  "message_type": "consolidated_report",
  "timestamp": "2026-04-20T17:00:00Z",
  "from_role": "ppm",
  "to_role": "pm",
  "message_id": "RPT-PPM-0018",
  "status": "unread",
  "report_date": "2026-04-20",
  "overall_status": "green",
  "summary": "Six subsystems active. 14 tasks in flight. No blockers. MTM absorbed one tool from tm_vm_lifecycle into the shared layer.",
  "open_escalations": 0,
  "key_events": [
    "Spec v1.3 of vm_lifecycle released after Marko reflection sign-off",
    "Coder scale-up triggered for spm_storage — queue depth 4 for two cycles"
  ],
  "requires_client_decision": []
}
```

---

### `ESC` — escalation

**What it does:** Any role escalates an issue it cannot resolve alone.

**Who sends it:** Any internal role → upward in the hierarchy (usually `ppm`, sometimes `pm`)

`rationale` is mandatory. An escalation without rationale is a call for help with no information in it. The system will not reject it but the PM will.

```json
{
  "schema_version": "1.0",
  "message_type": "escalation",
  "timestamp": "2026-04-20T14:32:00Z",
  "from_role": "spm_storage",
  "to_role": "ppm",
  "message_id": "ESC-SPM-STORAGE-0003",
  "status": "unread",
  "category": "resource_conflict",
  "severity": "high",
  "subject": "Two tasks require exclusive pool access and cannot run in parallel",
  "rationale": "TSK-0044 and TSK-0051 both require exclusive write access to the Libvirt storage pool. Sequential execution adds 40 minutes to the cycle.",
  "blocking_tasks": ["TSK-0044", "TSK-0051"],
  "proposed_resolution": "Request from INFRA: provision second storage pool named vm_backup_pool.",
  "requires_decision_by": "2026-04-21T12:00:00Z"
}
```

---

### `DEC` — pm_decision

**What it does:** PM returns a decision in response to an escalation or request. `rationale` and `instructions` are both mandatory and both must be non-empty.

**Who sends it:** `pm` → any internal role

```json
{
  "schema_version": "1.0",
  "message_type": "pm_decision",
  "timestamp": "2026-04-20T16:10:00Z",
  "from_role": "pm",
  "to_role": "ppm",
  "message_id": "DEC-PM-0021",
  "status": "unread",
  "in_response_to": "ESC-SPM-STORAGE-0003",
  "decision": "approved",
  "rationale": "Marko confirmed storage expansion is within infrastructure budget. Splitting the pool is cheaper than sequential execution.",
  "instructions": "PPM to coordinate with INFRA for provisioning. New pool name: vm_backup_pool.",
  "affects_roles": ["spm_storage", "infra"],
  "deadline": "2026-04-22T17:00:00Z"
}
```

---

### `PLN` — project_plan

**What it does:** PM hands PPM the authoritative project plan.

**Who sends it:** `pm` → `ppm`

```json
{
  "schema_version": "1.0",
  "message_type": "project_plan",
  "timestamp": "2026-04-20T10:00:00Z",
  "from_role": "pm",
  "to_role": "ppm",
  "message_id": "PLN-PM-0003",
  "status": "unread",
  "plan_version": "1.3",
  "supersedes": "PLN-PM-0002",
  "scope_summary": "Implement VM lifecycle, storage, networking, and restore subsystems. Deliver CLI-driven MVP.",
  "milestones": [
    {
      "milestone_id": "M1",
      "title": "VM lifecycle subsystem complete",
      "acceptance_criteria": "Create, start, stop, destroy VM via CLI. All tests pass.",
      "target_date": "2026-05-15"
    },
    {
      "milestone_id": "M2",
      "title": "5-second restore proven end-to-end",
      "acceptance_criteria": "Snapshot, corrupt, restore. Full VM functional in under 5 seconds.",
      "target_date": "2026-06-15"
    }
  ],
  "constraints": "Rust only for core framework. Libvirt as hypervisor abstraction.",
  "non_goals": "No GUI in MVP. No multi-node deployment."
}
```

---

### `CRP` — client_report

**What it does:** PM reports outward to Marko.

**Who sends it:** `pm` → `client` (Marko)

```json
{
  "schema_version": "1.0",
  "message_type": "client_report",
  "timestamp": "2026-04-20T18:00:00Z",
  "from_role": "pm",
  "to_role": "client",
  "message_id": "CRP-PM-0015",
  "status": "unread",
  "report_date": "2026-04-20",
  "overall_status": "green",
  "progress_summary": "VM lifecycle is 60 percent through M1. Storage expansion approved and scheduled.",
  "decisions_needed": [],
  "upcoming_checkpoints": [
    "Spec v1.4 reflection sign-off requested for networking subsystem by 2026-04-22"
  ],
  "risks": []
}
```

---

## v1.1 — Notifications and disputes

The trigger mechanism that makes outbox writes visible, and the first tools for handling disagreements before they become escalations.

---

### `NTF` — notification

**What it does:** Tells a recipient that content is waiting in a specific outbox file.

**Who sends it:** Any role → any role

This is the most sent message in the system. Every outbox write must be followed by one of these. The outbox write is the letter. The notification is the knock on the door. Without the knock, nobody knows the letter arrived.

```json
{
  "schema_version": "1.0",
  "message_type": "notification",
  "timestamp": "2026-04-20T11:45:00Z",
  "from_role": "sd",
  "to_role": "ppm",
  "message_id": "NTF-SD-0089",
  "status": "unread",
  "points_to_outbox": ".agents/outbox/sd_to_ppm.json",
  "points_to_message_id": "SPR-SD-0012",
  "summary": "Spec release for storage subsystem v1.1 is ready for task decomposition."
}
```

---

### `CVR` — convention_request

**What it does:** An agent encounters a situation CLAUDE.md does not cover and asks for a ruling.

**Who sends it:** Any role → `pm` (via hierarchy)

The alternative to writing a CVR is improvising. Improvising is not permitted. If the situation is not covered, write the CVR, block the task, and wait. The PM decides. CLAUDE.md gets updated. The version number increments. This is the correct path even when it feels slow.

```json
{
  "schema_version": "1.0",
  "message_type": "convention_request",
  "timestamp": "2026-04-20T13:20:00Z",
  "from_role": "spm_networking",
  "to_role": "ppm",
  "message_id": "CVR-SPM-NETWORKING-0002",
  "status": "unread",
  "subject": "Minimum interface coverage threshold not defined in CLAUDE.md",
  "situation": "Coder submitted a code_package with passing tests covering only 40 percent of the interface contract. CLAUDE.md does not specify a minimum coverage threshold.",
  "proposed_convention": "Minimum 70 percent interface coverage required. Below threshold: SPM rejects back to coder with specific uncovered cases identified.",
  "blocking_task": "TSK-0058",
  "requires_response_by": "2026-04-21T17:00:00Z"
}
```

---

### `SCR` — spec_correction_request

**What it does:** QA or SEC identifies a problem rooted in the spec itself, not the code. Routes directly to SD.

**Who sends it:** `qa` or `sec` → `sd`

This bypasses the normal implementation loop. The problem is in the specification. Going through PPM to get to SD is unnecessary overhead when the destination and the fix are both clear.

```json
{
  "schema_version": "1.0",
  "message_type": "spec_correction_request",
  "timestamp": "2026-04-20T15:00:00Z",
  "from_role": "qa",
  "to_role": "sd",
  "message_id": "SCR-QA-0007",
  "status": "unread",
  "affected_spec": "storage_subsystem_v1.1",
  "affected_section": "Section 3.2: Snapshot manifest format",
  "problem": "Spec requires a checksum field but does not specify the algorithm. Implementations will diverge.",
  "proposed_fix": "Add: checksum_algorithm: string — enum: sha256 | blake3. Default: sha256.",
  "blocking_task": "TSK-0044",
  "severity": "high"
}
```

---

### `IDP` — interface_dispute

**What it does:** An SPM raises a dispute about an interface contract between two subsystems. SD resolves. PM stays out of it.

**Who sends it:** `spm` (initiating party) → `sd`

```json
{
  "schema_version": "1.0",
  "message_type": "interface_dispute",
  "timestamp": "2026-04-21T10:00:00Z",
  "from_role": "spm_vm_lifecycle",
  "to_role": "sd",
  "message_id": "IDP-SPM-VM-LIFECYCLE-0001",
  "status": "unread",
  "other_party": "spm_storage",
  "contested_interface": "vm_snapshot_manifest_v1.1 section 4",
  "dispute": "Storage insists the checksum field is optional. We insist it is required. The spec is ambiguous.",
  "our_position": "Checksum must be required. Without it the restore verification step cannot be guaranteed.",
  "other_party_position": "Checksum should be optional to support lightweight snapshots for development environments.",
  "blocking_tasks": ["TSK-0044", "TSK-0051"]
}
```

---

### `BRS` — binding_resolution

**What it does:** SD resolves an interface dispute. This is binding. Both parties must implement accordingly.

**Who sends it:** `sd` → both SPM parties

```json
{
  "schema_version": "1.0",
  "message_type": "binding_resolution",
  "timestamp": "2026-04-21T14:00:00Z",
  "from_role": "sd",
  "to_role": "spm_vm_lifecycle",
  "also_notified": ["spm_storage"],
  "message_id": "BRS-SD-0002",
  "status": "unread",
  "in_response_to": "IDP-SPM-VM-LIFECYCLE-0001",
  "resolution": "Checksum is required in production snapshots. Optional flag checksum_skip: bool defaults to false and may only be set true when snapshot_mode is development. Both subsystems implement accordingly.",
  "rationale": "Production restore integrity is non-negotiable. Development speed is a valid secondary concern. The flag separates the two.",
  "spec_update_required": true,
  "spec_section": "vm_snapshot_manifest_v1.1 section 4"
}
```

---

## v1.2 — Task management

The machinery that turns a project plan into actual work assignments and status reporting.

---

### `TAS` — task_assignment

**What it does:** PPM assigns a task to an SPM with full context for execution.

**Who sends it:** `ppm` → `spm`

```json
{
  "schema_version": "1.0",
  "message_type": "task_assignment",
  "timestamp": "2026-04-20T09:30:00Z",
  "from_role": "ppm",
  "to_role": "spm_vm_lifecycle",
  "message_id": "TAS-PPM-0041",
  "status": "unread",
  "task_id": "TSK-0067",
  "milestone_id": "M1",
  "title": "Implement VM graceful shutdown with 30-second ACPI timeout",
  "description": "VM must attempt ACPI shutdown. If not complete within 30 seconds, force destroy. Return final domain state and elapsed time.",
  "acceptance_criteria": [
    "VM transitions to shutoff state within 30s on ACPI success",
    "VM is force-destroyed if ACPI shutdown exceeds 30s timeout",
    "Function returns: final_state (enum), elapsed_ms (u64)",
    "All error paths return typed errors from the VM_ error namespace"
  ],
  "spec_reference": "vm_lifecycle_v1.3 section 2.4",
  "dependencies": ["TSK-0061"],
  "deadline": "2026-04-25T17:00:00Z"
}
```

---

### `SRP` — status_report

**What it does:** Any internal role reports its current status to PPM, or upward in hierarchy on request.

**Who sends it:** Any internal role → `ppm` (or hierarchy)

```json
{
  "schema_version": "1.0",
  "message_type": "status_report",
  "timestamp": "2026-04-20T17:00:00Z",
  "from_role": "spm_storage",
  "to_role": "ppm",
  "message_id": "SRP-SPM-STORAGE-0022",
  "status": "unread",
  "period": "2026-04-20",
  "overall_status": "amber",
  "tasks_in_progress": 3,
  "tasks_blocked": 1,
  "tasks_completed_this_cycle": 2,
  "blockers": [
    {
      "task_id": "TSK-0044",
      "reason": "Awaiting pool provisioning from INFRA. ESC-SPM-STORAGE-0003 open."
    }
  ],
  "risks": "If pool is not provisioned by end of tomorrow, M1 delivery is at risk."
}
```

---

### `TCM` — task_completion

**What it does:** SPM notifies PPM that a task is complete. Must include sign-off IDs.

**Who sends it:** `spm` → `ppm`

The `cr_signoff_id` and `qa_signoff_id` fields here are not optional formalities. They are the proof that the task passed its gates. A task_completion without them is a claim without evidence.

```json
{
  "schema_version": "1.0",
  "message_type": "task_completion",
  "timestamp": "2026-04-22T15:00:00Z",
  "from_role": "spm_vm_lifecycle",
  "to_role": "ppm",
  "message_id": "TCM-SPM-VM-LIFECYCLE-0014",
  "status": "unread",
  "task_id": "TSK-0067",
  "milestone_id": "M1",
  "summary": "VM graceful shutdown implemented. ACPI path and force-destroy timeout both tested.",
  "deliverable_path": "src/vm_lifecycle/shutdown.rs",
  "cr_signoff_id": "RRS-CR-0022",
  "qa_signoff_id": "QAP-QA-0008",
  "doc_delivered": true,
  "doc_path": "docs/vm_lifecycle/shutdown.md"
}
```

---

### `SRQ` — status_report_request

**What it does:** PPM requests a status report from a specific role.

**Who sends it:** `ppm` → any internal role

```json
{
  "schema_version": "1.0",
  "message_type": "status_report_request",
  "timestamp": "2026-04-20T16:00:00Z",
  "from_role": "ppm",
  "to_role": "spm_networking",
  "message_id": "SRQ-PPM-0031",
  "status": "unread",
  "reason": "Consolidated report due in 60 minutes. Networking status outstanding.",
  "required_by": "2026-04-20T17:00:00Z"
}
```

---

## v1.3 — Design and spec flow

The path from Marko's analysis through SD's architecture to a released specification.

---

### `DRF` — design_reflection

**What it does:** SD produces a reflection document after completing a design. Marko reviews and signs off before any spec leaves the role.

**Who sends it:** `sd` → `pm` (forwarded to Marko)

No spec is released without this. It is checkpoint 1 and it is a hard stop.

```json
{
  "schema_version": "1.0",
  "message_type": "design_reflection",
  "timestamp": "2026-04-19T16:00:00Z",
  "from_role": "sd",
  "to_role": "pm",
  "message_id": "DRF-SD-0005",
  "status": "unread",
  "spec_name": "storage_subsystem_v1.1",
  "summary": "Defined storage pool lifecycle, snapshot manifest format, and CoW layer interface. Three interface contracts specified.",
  "decisions_made": [
    "Snapshot manifest uses SHA-256 checksums for production, optional for development mode",
    "CoW layers are immutable after creation — no in-place modification"
  ],
  "assumptions": [
    "Libvirt storage pool API is available on all target systems",
    "Available disk space validated before snapshot creation"
  ],
  "open_questions": [],
  "risks": "None identified at this stage."
}
```

---

### `RSO` — reflection_signoff

**What it does:** PM signals Marko's approval of the design reflection. SD may now release the spec.

**Who sends it:** `pm` → `sd` (on Marko's behalf)

```json
{
  "schema_version": "1.0",
  "message_type": "reflection_signoff",
  "timestamp": "2026-04-19T17:30:00Z",
  "from_role": "pm",
  "to_role": "sd",
  "message_id": "RSO-PM-0005",
  "status": "unread",
  "in_response_to": "DRF-SD-0005",
  "approved": true,
  "notes": "Marko confirmed the checksum decision is correct. Proceed to spec release."
}
```

---

### `SPR` — spec_release

**What it does:** SD releases a specification. Triggers task decomposition by PPM and implementation by SPMs.

**Who sends it:** `sd` → `ppm`, `spm`, `sec`, `doc`

```json
{
  "schema_version": "1.0",
  "message_type": "spec_release",
  "timestamp": "2026-04-19T18:00:00Z",
  "from_role": "sd",
  "to_role": "ppm",
  "also_notified": ["spm_storage", "sec", "doc"],
  "message_id": "SPR-SD-0012",
  "status": "unread",
  "spec_name": "storage_subsystem_v1.1",
  "spec_version": "1.1",
  "spec_path": "specs/storage_subsystem_v1.1.md",
  "supersedes": "storage_subsystem_v1.0",
  "summary": "Adds snapshot manifest format, CoW interface contract, and backup storage pool spec.",
  "reflection_signoff_id": "RSO-PM-0005"
}
```

---

### `SRR` — spec_correction_response

**What it does:** SD responds to a spec_correction_request from QA or SEC.

**Who sends it:** `sd` → `qa` or `sec`

```json
{
  "schema_version": "1.0",
  "message_type": "spec_correction_response",
  "timestamp": "2026-04-21T11:00:00Z",
  "from_role": "sd",
  "to_role": "qa",
  "message_id": "SRR-SD-0004",
  "status": "unread",
  "in_response_to": "SCR-QA-0007",
  "resolution": "accepted",
  "change_made": "Section 3.2 updated: checksum_algorithm field added as required string enum (sha256 | blake3, default sha256).",
  "new_spec_version": "1.2",
  "spec_path": "specs/storage_subsystem_v1.2.md"
}
```

---

### `DDC` — design_decision

**What it does:** SD logs a significant design decision in its own state. Not a message sent to others — a record kept internally.

**Who sends it:** `sd` — internal log record in `sd_state.json`

```json
{
  "schema_version": "1.0",
  "message_type": "design_decision",
  "timestamp": "2026-04-19T15:00:00Z",
  "from_role": "sd",
  "to_role": "sd",
  "message_id": "DDC-SD-0011",
  "status": "done",
  "decision": "CoW layers are immutable after creation",
  "rationale": "Mutable layers create race conditions in concurrent restore scenarios. Immutability simplifies the correctness proof and aligns with the patent-pending mechanism.",
  "alternatives_considered": ["Append-only mutation log", "Copy-on-write with versioned heads"],
  "spec_section": "storage_subsystem_v1.1 section 5"
}
```

---

## v1.4 — Spec correction response

Already covered above under `SRR`.

---

## v1.5 — Subsystem execution

The pipeline from task assignment to coder output.

---

### `CAG` — coder_assignment

**What it does:** SPM assigns a specific coding task to a coder instance.

**Who sends it:** `spm` → `coder_{n}`

```json
{
  "schema_version": "1.0",
  "message_type": "coder_assignment",
  "timestamp": "2026-04-20T10:00:00Z",
  "from_role": "spm_vm_lifecycle",
  "to_role": "coder_vm_01",
  "message_id": "CAG-SPM-VM-LIFECYCLE-0031",
  "status": "unread",
  "task_id": "TSK-0067",
  "title": "Implement VM graceful shutdown with 30-second ACPI timeout",
  "spec_reference": "vm_lifecycle_v1.3 section 2.4",
  "acceptance_criteria": [
    "VM transitions to shutoff state within 30s on ACPI success",
    "VM is force-destroyed if ACPI shutdown exceeds 30s",
    "Returns: final_state (enum), elapsed_ms (u64)",
    "All errors use VM_ error namespace"
  ],
  "tools_available": ["libvirt_state_poller v0.2.0"],
  "deadline": "2026-04-24T17:00:00Z"
}
```

---

### `CPK` — code_package

**What it does:** Coder submits finished code to SPM. Must include passing test results.

**Who sends it:** `coder` → `spm`

```json
{
  "schema_version": "1.0",
  "message_type": "code_package",
  "timestamp": "2026-04-20T15:00:00Z",
  "from_role": "coder_vm_01",
  "to_role": "spm_vm_lifecycle",
  "message_id": "CPK-CODER-VM-01-0031",
  "status": "unread",
  "task_id": "TSK-0067",
  "code_path": "src/vm_lifecycle/shutdown.rs",
  "test_results": {
    "total": 12,
    "passed": 12,
    "failed": 0,
    "coverage_percent": 87
  },
  "design_language_compliance_notes": "All error codes use VM_ prefix. No deviations.",
  "dependencies_used": ["libvirt_rust_wrapper v0.3.1", "libvirt_state_poller v0.2.0"],
  "build_command": "cargo build --release"
}
```

---

### `TRQ` — tool_request

**What it does:** SPM (or TEST) requests a tool from TM or MTM. If `route_to_mtm` is true, MTM handles it.

**Who sends it:** `spm` → `tm` (or `mtm`). `test` → `mtm`.

```json
{
  "schema_version": "1.0",
  "message_type": "tool_request",
  "timestamp": "2026-04-20T09:00:00Z",
  "from_role": "spm_vm_lifecycle",
  "to_role": "tm_vm_lifecycle",
  "message_id": "TRQ-SPM-VM-LIFECYCLE-0011",
  "status": "unread",
  "task_id": "TSK-0067",
  "tool_description": "Poll Libvirt domain state at a configurable interval with a timeout. Return final state and elapsed time.",
  "inputs": "domain_handle, poll_interval_ms (u64), timeout_ms (u64)",
  "outputs": "Result<(DomainState, u64), VmError>",
  "deadline": "2026-04-23T17:00:00Z",
  "route_to_mtm": false,
  "cross_subsystem_candidate": true,
  "blocks_tasks": ["TSK-0067", "TSK-0068"],
  "priority": "high"
}
```

---

### `RRQ` — review_request

**What it does:** SPM sends a code package to CR for design language and contract review.

**Who sends it:** `spm` → `cr`

`self_review_notes` are mandatory. An SPM that cannot describe what they already checked is sending something they have not reviewed themselves.

```json
{
  "schema_version": "1.0",
  "message_type": "review_request",
  "timestamp": "2026-04-20T15:30:00Z",
  "from_role": "spm_vm_lifecycle",
  "to_role": "cr",
  "message_id": "RRQ-SPM-VM-LIFECYCLE-0014",
  "status": "unread",
  "task_id": "TSK-0067",
  "code_package_id": "CPK-CODER-VM-01-0031",
  "code_path": "src/vm_lifecycle/shutdown.rs",
  "self_review_notes": "Reviewed the 30s timeout path carefully. Tests cover both ACPI success and timeout fallback. One concern: the poll interval is not unit-tested for the 0ms edge case — the config validator rejects 0ms upstream so this was deemed safe.",
  "interface_contracts_verified": ["vm_lifecycle_v1.3 section 2.4"],
  "design_language_compliance_confirmed": true
}
```

---

### `RRS` — review_result

**What it does:** CR returns a review decision to SPM. The `message_id` of an approved RRS becomes the `cr_signoff_id` referenced downstream.

**Who sends it:** `cr` → `spm`

```json
{
  "schema_version": "1.0",
  "message_type": "review_result",
  "timestamp": "2026-04-20T16:20:00Z",
  "from_role": "cr",
  "to_role": "spm_vm_lifecycle",
  "message_id": "RRS-CR-0022",
  "status": "unread",
  "in_response_to": "RRQ-SPM-VM-LIFECYCLE-0014",
  "task_id": "TSK-0067",
  "decision": "approved",
  "findings": [
    {
      "severity": "minor",
      "location": "shutdown.rs:47",
      "description": "Consider named constant for 500ms poll interval default."
    }
  ],
  "blocking_findings": 0,
  "requires_rework": false,
  "reviewer_signoff": true
}
```

---

## v1.6 — Tool delivery and coverage

---

### `TDL` — tool_delivery

**What it does:** TM or MTM delivers a finished tool. Unblocks the tasks that were waiting.

**Who sends it:** `tm` or `mtm` → `spm` (or `cr`, `test`)

```json
{
  "schema_version": "1.0",
  "message_type": "tool_delivery",
  "timestamp": "2026-04-20T14:00:00Z",
  "from_role": "tm_vm_lifecycle",
  "to_role": "spm_vm_lifecycle",
  "message_id": "TDL-TM-VM-LIFECYCLE-0007",
  "status": "unread",
  "in_response_to": "TRQ-SPM-VM-LIFECYCLE-0011",
  "tool_name": "libvirt_state_poller",
  "tool_path": "tools/vm_lifecycle/libvirt_state_poller/",
  "tool_version": "0.1.0",
  "description": "Polls virDomainGetState at configurable interval with timeout. Returns final state and elapsed time.",
  "inputs": "domain_handle, poll_interval_ms, timeout_ms",
  "outputs": "final_state, elapsed_ms",
  "test_coverage": "92 percent. Covers happy path and timeout.",
  "doc_path": "tools/vm_lifecycle/libvirt_state_poller/README.md",
  "unblocks_tasks": ["TSK-0067", "TSK-0068"]
}
```

---

### `CVN` — coverage_notice

**What it does:** MTM notifies a TM that it is entering or exiting coverage mode for that subsystem. Permitted as direct MTM→TM communication.

**Who sends it:** `mtm` → `tm`

```json
{
  "schema_version": "1.0",
  "message_type": "coverage_notice",
  "timestamp": "2026-04-20T11:00:00Z",
  "from_role": "mtm",
  "to_role": "tm_storage",
  "message_id": "CVN-MTM-0003",
  "status": "unread",
  "target_tm": "tm_storage",
  "active": true,
  "coverage_start": "2026-04-20T11:00:00Z",
  "reason": "PPM reported tm_storage unresponsive for two cycles. Coverage begins immediately.",
  "tools_delivered_during_coverage": [],
  "resume_instructions": ""
}
```

---

## v1.7 — Tool updates

---

### `TUP` — tool_update

**What it does:** TM pushes an update to an existing tool. Separate type code so it is never confused with initial delivery.

**Who sends it:** `tm` → `spm`

```json
{
  "schema_version": "1.0",
  "message_type": "tool_update",
  "timestamp": "2026-04-21T10:00:00Z",
  "from_role": "tm_vm_lifecycle",
  "to_role": "spm_vm_lifecycle",
  "message_id": "TUP-TM-VM-LIFECYCLE-0002",
  "status": "unread",
  "tool_name": "libvirt_state_poller",
  "previous_version": "0.1.0",
  "new_version": "0.2.0",
  "tool_path": "tools/vm_lifecycle/libvirt_state_poller/",
  "breaking_changes": false,
  "change_summary": "Added optional callback parameter for state-change events. Existing call sites unaffected.",
  "migration_notes": "No migration required. Callback is optional, defaults to None."
}
```

---

## v1.8 — Shared tooling layer

---

### `STN` — shared_tool_notice

**What it does:** MTM announces a new shared tool in the `common` crate. Any local tools it supersedes must be deprecated. Permitted as direct MTM→TM communication.

**Who sends it:** `mtm` → all consumers

```json
{
  "schema_version": "1.0",
  "message_type": "shared_tool_notice",
  "timestamp": "2026-04-21T14:00:00Z",
  "from_role": "mtm",
  "to_role": "spm_vm_lifecycle",
  "also_notified": ["spm_storage", "spm_networking", "cr", "test"],
  "message_id": "STN-MTM-0005",
  "status": "unread",
  "tool_name": "libvirt_state_poller",
  "crate": "common",
  "tool_path": "common/src/libvirt_state_poller/",
  "tool_version": "1.0.0",
  "description": "Polls libvirt domain state with configurable interval and timeout. Available to all subsystems.",
  "usage": "use common::libvirt_state_poller::poll_state;",
  "absorbed_from": "tm_vm_lifecycle",
  "supersedes_local_tools": ["tools/vm_lifecycle/libvirt_state_poller/"],
  "intended_consumers": ["spm_vm_lifecycle", "spm_storage", "spm_networking", "cr", "test"]
}
```

---

### `DPN` — deprecation_notice

**What it does:** MTM announces a shared tool is being retired. Consumers must migrate before the deprecation date.

**Who sends it:** `mtm` → affected consumers

```json
{
  "schema_version": "1.0",
  "message_type": "deprecation_notice",
  "timestamp": "2026-05-01T09:00:00Z",
  "from_role": "mtm",
  "to_role": "spm_vm_lifecycle",
  "also_notified": ["spm_storage", "cr"],
  "message_id": "DPN-MTM-0001",
  "status": "unread",
  "tool_name": "libvirt_state_poller",
  "current_version": "1.0.0",
  "deprecation_date": "2026-06-01",
  "reason": "Superseded by libvirt_state_observer v2.0.0 which adds event-stream subscription without polling overhead.",
  "replacement_tool": "libvirt_state_observer",
  "replacement_path": "common/src/libvirt_state_observer/",
  "migration_notes": "poll_state(domain, 500, 30000) becomes observe_state(domain, timeout_ms=30000). Callback is now mandatory.",
  "breaking_changes": true
}
```

---

## v2.0 — Quality approval

The gate. CR signs off on correctness. QA signs off on whether the thing does what the spec says. These are different questions asked by different roles. `cr_signoff_id` is mandatory in `qa_approval`. QA does not open a review that CR has not already closed.

---

### `QAP` — qa_approval

**What it does:** QA approves a code package after verifying it against the specification.

**Who sends it:** `qa` → `spm`

```json
{
  "schema_version": "1.0",
  "message_type": "qa_approval",
  "timestamp": "2026-04-22T14:30:00Z",
  "from_role": "qa",
  "to_role": "spm_vm_lifecycle",
  "message_id": "QAP-QA-0008",
  "status": "unread",
  "task_id": "TSK-0067",
  "code_package_id": "CPK-CODER-VM-01-0031",
  "cr_signoff_id": "RRS-CR-0022",
  "spec_version_reviewed": "vm_lifecycle_v1.3",
  "subsystem_id": "vm_lifecycle",
  "verdict": "approved",
  "conditions": [],
  "findings": [],
  "reviewed_at": "2026-04-22T14:30:00Z"
}
```

---

## v2.4 — Security and MTM tool gate

Two things became formal this version: SEC's communication with the rest of the organisation, and the mechanism that stops the same tool from being built twice.

---

### `SST` — security_standards

**What it does:** SEC publishes the current security standards to all relevant roles at session start.

**Who sends it:** `sec` → `sd`, `ppm`, `spm_{all}`, `cr`

These go out at the start of a session. They are not a request. They are the baseline that everyone else operates within.

```json
{
  "schema_version": "1.0",
  "message_type": "security_standards",
  "timestamp": "2026-05-01T09:00:00Z",
  "from_role": "sec",
  "to_role": "sd",
  "also_notified": ["ppm", "spm_vm_lifecycle", "spm_storage", "spm_networking", "cr"],
  "message_id": "SST-SEC-0004",
  "status": "unread",
  "standards_version": "1.2",
  "standards_path": "docs/security/standards_v1.2.md",
  "summary": "No secrets in source. Input validation on all Libvirt domain parameters. Privilege separation enforced between management and execution paths.",
  "changes_since_last": [
    "Added: domain parameter sanitisation required before all virDomain API calls"
  ]
}
```

---

### `SSO` — security_signoff

**What it does:** SEC issues a security sign-off to PM and CMVC at the release gate. Nothing ships without this on file.

**Who sends it:** `sec` → `pm` and `cmvc`

This is the mechanism behind absolute rule 2. The rule existed before the type was defined. Now both exist.

```json
{
  "schema_version": "1.0",
  "message_type": "security_signoff",
  "timestamp": "2026-05-15T16:00:00Z",
  "from_role": "sec",
  "to_role": "pm",
  "also_notified": ["cmvc"],
  "message_id": "SSO-SEC-0001",
  "status": "unread",
  "release_candidate": "v0.1.0-rc1",
  "scope_reviewed": ["vm_lifecycle", "storage", "networking"],
  "verdict": "approved",
  "open_findings": [],
  "conditions": [],
  "signed_at": "2026-05-15T16:00:00Z"
}
```

---

### `TMS` — tool_submission

**What it does:** TM submits a tool to MTM for gate approval before it can be checked in.

**Who sends it:** `tm` → `mtm`

Nothing gets into any crate without TAP from MTM. Incomplete documentation is automatic rejection. This is not a review. It is a gate.

```json
{
  "schema_version": "1.0",
  "message_type": "tool_submission",
  "timestamp": "2026-04-23T11:00:00Z",
  "from_role": "tm_vm_lifecycle",
  "to_role": "mtm",
  "message_id": "TMS-TM-VM-LIFECYCLE-0003",
  "status": "unread",
  "tool_name": "vm_domain_validator",
  "tool_path": "tools/vm_lifecycle/vm_domain_validator/",
  "tool_version": "0.1.0",
  "description": "Validates virDomain handle is non-null and domain is in an expected state before API calls. Returns typed error on invalid state.",
  "inputs": "domain_handle: *mut virDomain, expected_states: &[DomainState]",
  "outputs": "Result<(), VmError>",
  "test_coverage": "96 percent",
  "doc_path": "tools/vm_lifecycle/vm_domain_validator/README.md",
  "cross_subsystem_candidate": true,
  "potentially_destructive": false,
  "in_response_to_trq": "TRQ-SPM-VM-LIFECYCLE-0012"
}
```

---

### `TAP` — tool_approval

**What it does:** MTM approves a tool submission. The tool may now be checked in.

**Who sends it:** `mtm` → `tm`

```json
{
  "schema_version": "1.0",
  "message_type": "tool_approval",
  "timestamp": "2026-04-23T13:00:00Z",
  "from_role": "mtm",
  "to_role": "tm_vm_lifecycle",
  "message_id": "TAP-MTM-0008",
  "status": "unread",
  "in_response_to": "TMS-TM-VM-LIFECYCLE-0003",
  "tool_name": "vm_domain_validator",
  "approved_for_crate": "common",
  "rationale": "Cross-subsystem utility. Three subsystems have independent validator implementations. Absorbed into common.",
  "final_path": "common/src/vm_domain_validator/",
  "notes": "Rename to domain_validator to match common crate naming convention."
}
```

---

### `TRJ` — tool_rejection

**What it does:** MTM rejects a tool submission with specific reasons.

**Who sends it:** `mtm` → `tm`

```json
{
  "schema_version": "1.0",
  "message_type": "tool_rejection",
  "timestamp": "2026-04-23T13:00:00Z",
  "from_role": "mtm",
  "to_role": "tm_storage",
  "message_id": "TRJ-MTM-0002",
  "status": "unread",
  "in_response_to": "TMS-TM-STORAGE-0001",
  "tool_name": "snapshot_checksum_validator",
  "reasons": [
    "Documentation missing: README does not describe error return types",
    "Test coverage 61 percent — minimum 80 percent required",
    "Equivalent to vm_domain_validator in common — reuse rather than duplicate"
  ],
  "resubmit_allowed": true,
  "blocking_tasks": []
}
```

---

### `TSR` — tool_sec_referral

**What it does:** MTM refers a tool submission to SEC for security review when `potentially_destructive` is true or SEC criteria are met.

**Who sends it:** `mtm` → `sec`

```json
{
  "schema_version": "1.0",
  "message_type": "tool_sec_referral",
  "timestamp": "2026-04-24T10:00:00Z",
  "from_role": "mtm",
  "to_role": "sec",
  "message_id": "TSR-MTM-0001",
  "status": "unread",
  "tool_name": "vm_force_destroy",
  "original_submission_id": "TMS-TM-VM-LIFECYCLE-0004",
  "referral_reason": "Tool calls virDomainDestroy without state validation. Marked potentially_destructive by submitter.",
  "tool_path": "tools/vm_lifecycle/vm_force_destroy/",
  "tool_version": "0.1.0",
  "doc_path": "tools/vm_lifecycle/vm_force_destroy/README.md",
  "requires_response_by": "2026-04-25T17:00:00Z"
}
```

---

### `STC` — sec_tool_clearance

**What it does:** SEC responds to a tool_sec_referral with clearance or rejection.

**Who sends it:** `sec` → `mtm`

```json
{
  "schema_version": "1.0",
  "message_type": "sec_tool_clearance",
  "timestamp": "2026-04-24T15:00:00Z",
  "from_role": "sec",
  "to_role": "mtm",
  "message_id": "STC-SEC-0001",
  "status": "unread",
  "in_response_to": "TSR-MTM-0001",
  "tool_name": "vm_force_destroy",
  "verdict": "cleared_with_conditions",
  "conditions": [
    "Caller must log the domain name and reason before invoking virDomainDestroy",
    "Tool must not be callable from user-facing code paths — internal use only"
  ],
  "open_findings": []
}
```

---

## v2.5 — UX, DOC, and TEST

Three new roles. Six new message types. The quality, documentation, and usability gates close.

---

### `UXR` — ux_spec_release

**What it does:** UX releases a design specification for a user-facing surface to the UI subsystem SPM. Contains usability acceptance criteria that TEST will eventually verify.

**Who sends it:** `ux` → `spm` (UI subsystem)

The criteria in this message are not preferences. They are the pass/fail standard against which TEST verifies the implementation. "Looks clean" is not a criterion. "Primary action completes within two interactions from the main screen" is a criterion.

```json
{
  "schema_version": "1.0",
  "message_type": "ux_spec_release",
  "timestamp": "2026-05-10T14:00:00Z",
  "from_role": "ux",
  "to_role": "spm_ui",
  "message_id": "UXR-UX-0001",
  "status": "unread",
  "surface_id": "UXS-RESTORE-0001",
  "surface_name": "VM restore confirmation dialog",
  "spec_version": "1.0",
  "spec_document_path": "specs/ux/restore_dialog_v1.0.md",
  "components": ["COMP-RESTORE-BTN-001", "COMP-STATUS-INDICATOR-001", "COMP-CONFIRM-MODAL-001"],
  "design_tokens_version": "1.0",
  "replaces_spec_version": "",
  "technical_constraints_applied": [],
  "usability_criteria": [
    {
      "criterion_id": "UXC-RESTORE-0001",
      "description": "User can initiate a restore from the main screen in two interactions or fewer",
      "test_method": "Observer counts interactions from main screen to restore initiated. Pass if count is 2 or fewer."
    },
    {
      "criterion_id": "UXC-RESTORE-0002",
      "description": "Restore button is visible without scrolling on a 1024x768 viewport",
      "test_method": "Render at 1024x768. Verify restore button is within viewport without scrolling."
    }
  ]
}
```

---

### `DGF` — documentation_gap_flag

**What it does:** DOC flags to QA that a task was marked complete without the required documentation. QA holds the sign-off until the flag is resolved.

**Who sends it:** `doc` → `qa`

A completed task with no documentation is a fact with no record. The flag does not punish. It enforces the principle that a task is not done until it can be explained to the next person who needs to work on it.

```json
{
  "schema_version": "1.0",
  "message_type": "documentation_gap_flag",
  "timestamp": "2026-05-10T17:00:00Z",
  "from_role": "doc",
  "to_role": "qa",
  "message_id": "DGF-DOC-0002",
  "status": "unread",
  "task_id": "TSK-0067",
  "owing_role": "spm_vm_lifecycle",
  "document_type": "subsystem_doc",
  "task_completed_at": "2026-05-10T15:30:00Z",
  "flagged_at": "2026-05-10T17:00:00Z",
  "flag_status": "open",
  "resolved_at": "",
  "resolution_note": ""
}
```

---

### `TPL` — test_plan

**What it does:** TEST submits a test plan to QA for approval before execution begins. No execution without an approved plan.

**Who sends it:** `test` → `qa`

The plan is not a formality. It is the contract between TEST and QA about what will be tested, to what coverage target, under what conditions. QA approves the contract. TEST executes it.

```json
{
  "schema_version": "1.0",
  "message_type": "test_plan",
  "timestamp": "2026-05-11T10:00:00Z",
  "from_role": "test",
  "to_role": "qa",
  "message_id": "TPL-TEST-0003",
  "status": "unread",
  "task_id": "TSK-0067",
  "spec_reference": "vm_lifecycle_v1.3 section 2.4",
  "coverage_target_percent": 90,
  "test_cases": [
    {
      "case_id": "TC-0067-001",
      "description": "ACPI shutdown completes within 30 seconds",
      "preconditions": "VM in running state",
      "steps": "Call graceful_shutdown(domain). Wait up to 30s.",
      "expected": "Domain state: shutoff. elapsed_ms < 30000.",
      "pass_criteria": "State is shutoff AND elapsed_ms is under 30000"
    },
    {
      "case_id": "TC-0067-002",
      "description": "Force destroy triggered when ACPI times out",
      "preconditions": "VM in running state. ACPI response stubbed to never complete.",
      "steps": "Call graceful_shutdown(domain). Wait 30 seconds.",
      "expected": "Domain force-destroyed. Function returns elapsed_ms >= 30000.",
      "pass_criteria": "Domain destroyed AND elapsed_ms >= 30000"
    }
  ],
  "tooling_required": ["libvirt_state_poller v1.0.0"],
  "estimated_duration_minutes": 20
}
```

---

### `TRS` — test_result

**What it does:** TEST delivers test results after executing an approved plan.

**Who sends it:** `test` → `qa` and `spm`

```json
{
  "schema_version": "1.0",
  "message_type": "test_result",
  "timestamp": "2026-05-11T12:30:00Z",
  "from_role": "test",
  "to_role": "qa",
  "also_sent_to": ["spm_vm_lifecycle"],
  "message_id": "TRS-TEST-0003",
  "status": "unread",
  "task_id": "TSK-0067",
  "plan_id": "TPL-TEST-0003",
  "overall_verdict": "passed",
  "coverage_achieved_percent": 91,
  "coverage_target_percent": 90,
  "cases_total": 12,
  "cases_passed": 12,
  "cases_failed": 0,
  "open_defects": [],
  "notes": "All cases passed on first run. Coverage target met."
}
```

---

### `DFR` — defect_report

**What it does:** TEST reports a failed test case to SPM with full reproduction steps.

**Who sends it:** `test` → `spm`

Reproduction steps are not optional. "It failed" is not a defect report. "Call graceful_shutdown on a domain in shutoff state. Expected: typed error VmError::InvalidState. Actual: panic." is a defect report.

```json
{
  "schema_version": "1.0",
  "message_type": "defect_report",
  "timestamp": "2026-05-11T11:00:00Z",
  "from_role": "test",
  "to_role": "spm_vm_lifecycle",
  "message_id": "DFR-TEST-0001",
  "status": "unread",
  "task_id": "TSK-0067",
  "test_case_id": "TC-0067-003",
  "spec_section_violated": "vm_lifecycle_v1.3 section 2.4 — error handling",
  "expected_result": "Returns VmError::InvalidState when called on a domain not in running state",
  "actual_result": "Thread panic: unwrap() on None at shutdown.rs:83",
  "reproduction_steps": [
    "1. Create domain in shutoff state",
    "2. Call graceful_shutdown(domain)",
    "3. Observe panic"
  ],
  "severity": "high",
  "environment": "cargo test --release, libvirt 9.0.0",
  "defect_status": "open"
}
```

---

### `UFN` — usability_finding

**What it does:** TEST reports a failed usability criterion to the UI/UX Designer.

**Who sends it:** `test` → `ux`

Usability findings go to UX, not SPM. SPM fixes code defects. UX addresses usability failures. These are different problems with different owners. Sending a usability finding to SPM produces a code change that does not solve the design problem.

```json
{
  "schema_version": "1.0",
  "message_type": "usability_finding",
  "timestamp": "2026-05-12T14:00:00Z",
  "from_role": "test",
  "to_role": "ux",
  "message_id": "UFN-TEST-0001",
  "status": "unread",
  "surface_id": "UXS-RESTORE-0001",
  "ux_spec_version": "1.0",
  "criterion_id": "UXC-RESTORE-0001",
  "criterion_description": "User can initiate a restore from the main screen in two interactions or fewer",
  "test_method_used": "Observer counts interactions from main screen to restore initiated",
  "observed_result": "Restore requires three interactions: main menu, subsystem select, then restore button",
  "verdict": "failed",
  "severity": "high",
  "notes": "The subsystem selection step is the extra interaction. Not present in the spec design.",
  "finding_status": "open"
}
```

---

## One last note

This cookbook covers all 45 message types in CLAUDE.md v2.5.

When you encounter a situation that needs a message type that is not here, you do not invent one. You write a `convention_request` (CVR) to PM, describe the situation, propose a type, and wait. The PM decides. CLAUDE.md gets updated. The version number increments. This document gets a new section.

The system grows through convention_requests, not through improvisation. That is the point of the system.

github.com/murtsu/RostadVM
