# Agent Message Cookbook

Concise examples of every message type in the VM Framework agent system. Copy, adapt, send.

**Companion to:** `CLAUDE.md` v1.8 and `INTERFACE_SPECIFICATION.md`

---

## How to read this

Each entry follows the same structure:

- **What it does** — one sentence
- **Who sends it** — the role that writes this message
- **Example** — copy-ready JSON with realistic values

Every message ID follows the pattern `{TYPE}-{ROLE}-{SEQUENCE}`. Increment the sequence counter in your state file after every send.

---

## v1.0 — Core protocol

The six messages that make Marko, PM, and PPM able to talk at all.

## `REQ` — client_request

**What it does:** Marko (or any external client) enters the system with a request.

**Who sends it:** `client` (Marko) → `pm`

**Example:**

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

## `RPT` — consolidated_report

**What it does:** PPM gives the PM one unified status report per cycle, collected from every internal role.

**Who sends it:** `ppm` → `pm`

**Example:**

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
    "Coder scale-up triggered for spm_storage (queue depth 4 for two cycles)"
  ],
  "requires_client_decision": []
}
```

---

## `ESC` — escalation

**What it does:** Any role escalates an issue upward that it cannot resolve alone.

**Who sends it:** Any internal role → `pm` (or upward in the hierarchy)

**Example:**

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
  "subject": "Two tasks require the same tool and cannot run in parallel",
  "rationale": "Task TSK-0044 and TSK-0051 both require exclusive write access to the Libvirt storage pool. Running them sequentially adds 40 minutes to the cycle. Proposing we split the pool into two named volumes.",
  "blocking_tasks": ["TSK-0044", "TSK-0051"],
  "proposed_resolution": "Request from INFRA: provision second storage pool named vm_backup_pool.",
  "requires_decision_by": "2026-04-21T12:00:00Z"
}
```

---

## `DEC` — pm_decision

**What it does:** PM returns a decision in response to an escalation or a client request.

**Who sends it:** `pm` → any internal role

**Example:**

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
  "decision": "Approved. Proceed with second storage pool.",
  "rationale": "Marko confirmed storage expansion is within infrastructure budget. Splitting the pool is cheaper than sequential task execution.",
  "action_required": "PPM to coordinate with INFRA for provisioning. New pool name: vm_backup_pool.",
  "deadline": "2026-04-22T17:00:00Z"
}
```

---

## `PLN` — project_plan

**What it does:** PM hands PPM the authoritative project plan with milestones and acceptance criteria.

**Who sends it:** `pm` → `ppm`

**Example:**

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
      "acceptance_criteria": "Snapshot, corrupt, restore. Full VM functional in < 5 seconds.",
      "target_date": "2026-06-15"
    }
  ],
  "constraints": "Rust-only for core framework. Libvirt as hypervisor abstraction.",
  "non_goals": "No GUI in MVP. No multi-node deployment."
}
```

---

## `CRP` — client_report

**What it does:** PM reports outward to Marko with a summary of progress and anything that needs his input.

**Who sends it:** `pm` → `client` (Marko)

**Example:**

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

The trigger mechanism, plus the first tools for handling disagreements.

## `NTF` — notification

**What it does:** Tells a recipient to check a specific outbox file. Without this, outbox writes are invisible.

**Who sends it:** Any role → any role

**Example:**

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

## `CVR` — convention_request

**What it does:** An agent hits a situation not covered by CLAUDE.md and asks for a ruling instead of improvising.

**Who sends it:** Any role → `pm` (via hierarchy)

**Example:**

```json
{
  "schema_version": "1.0",
  "message_type": "convention_request",
  "timestamp": "2026-04-20T13:20:00Z",
  "from_role": "spm_networking",
  "to_role": "ppm",
  "message_id": "CVR-SPM-NETWORKING-0002",
  "status": "unread",
  "situation": "Coder submitted a code_package with passing tests but the tests cover only 40 percent of the interface contract. CLAUDE.md does not specify minimum coverage.",
  "proposed_convention": "Minimum 70 percent interface coverage required. Below threshold: SPM rejects back to coder with specific uncovered cases.",
  "impact_if_delayed": "TSK-0058 cannot proceed. Three downstream tasks blocked.",
  "requires_response_by": "2026-04-21T17:00:00Z"
}
```

---

## `SCR` — spec_correction_request

**What it does:** QA or SEC spots a problem rooted in the spec, not in the code. Requests SD to fix the spec.

**Who sends it:** `qa` or `sec` → `sd`

**Example:**

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
  "problem": "Spec requires checksum field but does not specify algorithm. Implementations chose SHA-256 and MD5 respectively. Restore cannot verify across them.",
  "severity": "high",
  "proposed_correction": "Specify SHA-256 explicitly. Add migration note for existing SHA-256 and MD5 manifests.",
  "blocks_tasks": ["TSK-0061", "TSK-0062"]
}
```

---

## `SRR` — spec_correction_response

**What it does:** SD responds to a spec_correction_request within one cycle with accept, partially accept, or reject.

**Who sends it:** `sd` → `qa` or `sec`

**Example:**

```json
{
  "schema_version": "1.0",
  "message_type": "spec_correction_response",
  "timestamp": "2026-04-20T16:30:00Z",
  "from_role": "sd",
  "to_role": "qa",
  "message_id": "SRR-SD-0007",
  "status": "unread",
  "in_response_to": "SCR-QA-0007",
  "decision": "accepted",
  "rationale": "Algorithm ambiguity is a real defect. Specifying SHA-256 only; MD5 manifests must be regenerated. No partial migration.",
  "spec_update": {
    "new_spec_version": "1.2",
    "changed_sections": ["3.2"],
    "spec_path": "docs/specs/storage_subsystem_v1.2.md"
  },
  "ppm_action_required": true,
  "action_description": "Notify affected SPMs to invalidate MD5 manifests and regenerate."
}
```

---

## `IDP` — interface_dispute

**What it does:** Two SPMs cannot agree on an interface contract. They jointly submit to SD for binding resolution.

**Who sends it:** `spm` (initiating) → `sd` (with other SPM as co-author)

**Example:**

```json
{
  "schema_version": "1.0",
  "message_type": "interface_dispute",
  "timestamp": "2026-04-20T14:00:00Z",
  "from_role": "spm_storage",
  "to_role": "sd",
  "message_id": "IDP-SPM-STORAGE-0001",
  "status": "unread",
  "initiating_spm_id": "spm_storage",
  "co_author_spm_id": "spm_networking",
  "interface_name": "StorageNetworkBridge",
  "dispute_subject": "Error propagation direction when storage unreachable over network",
  "spm_storage_position": "Storage should return a retry-after hint to networking and let networking handle timeout.",
  "spm_networking_position": "Storage should fail immediately; networking does not own storage timeouts.",
  "attempted_resolution": "One cycle of direct exchange. No agreement.",
  "blocks_tasks": ["TSK-0063"],
  "requires_resolution_by": "2026-04-22T12:00:00Z"
}
```

---

## `BRS` — binding_resolution

**What it does:** SD's final, binding ruling on an interface dispute. Not negotiable.

**Who sends it:** `sd` → both disputing SPMs

**Example:**

```json
{
  "schema_version": "1.0",
  "message_type": "binding_resolution",
  "timestamp": "2026-04-20T16:00:00Z",
  "from_role": "sd",
  "to_role": "spm_storage",
  "also_notified": ["spm_networking", "ppm"],
  "message_id": "BRS-SD-0001",
  "status": "unread",
  "in_response_to": "IDP-SPM-STORAGE-0001",
  "interface_name": "StorageNetworkBridge",
  "resolution": "Storage fails immediately with specific error code STORAGE_UNREACHABLE. Networking interprets the code and handles retry policy. Retry-after hints are advisory only.",
  "rationale": "Separation of concerns: storage owns storage state, networking owns retry policy. Cross-contamination breaks the subsystem boundary defined in spec v1.1.",
  "spec_reference": "storage_subsystem_v1.2 section 4.1",
  "effective_immediately": true,
  "cag_supersede_required": true
}
```

---

## v1.2 — Task management

How PPM coordinates the daily flow of work.

## `TAS` — task_assignment

**What it does:** PPM hands a task to an SPM. Acceptance criteria are mandatory.

**Who sends it:** `ppm` → `spm`

**Example:**

```json
{
  "schema_version": "1.0",
  "message_type": "task_assignment",
  "timestamp": "2026-04-20T09:30:00Z",
  "from_role": "ppm",
  "to_role": "spm_vm_lifecycle",
  "message_id": "TAS-PPM-0044",
  "status": "unread",
  "task_id": "TSK-0067",
  "title": "Implement VM graceful shutdown via libvirt domain API",
  "description": "Current shutdown is forced. Need ACPI-based graceful shutdown with fallback to force after 30s timeout.",
  "spec_reference": "vm_lifecycle_v1.3 section 2.4",
  "acceptance_criteria": [
    "VM receives ACPI shutdown signal via virDomainShutdown",
    "Timeout is configurable, defaults to 30s",
    "Force shutdown occurs if graceful times out",
    "Unit tests cover both graceful and timeout paths"
  ],
  "priority": "high",
  "dependencies": ["TSK-0061"],
  "target_completion": "2026-04-22"
}
```

---

## `SRP` — status_report

**What it does:** Any internal role reports its current state to PPM each cycle.

**Who sends it:** Any internal role → `ppm`

**Example:**

```json
{
  "schema_version": "1.0",
  "message_type": "status_report",
  "timestamp": "2026-04-20T17:00:00Z",
  "from_role": "spm_vm_lifecycle",
  "to_role": "ppm",
  "message_id": "SRP-SPM-VM-LIFECYCLE-0023",
  "status": "unread",
  "report_date": "2026-04-20",
  "overall_status": "green",
  "summary": "Three tasks in progress, one awaiting CR review.",
  "active_tasks": [
    {"task_id": "TSK-0067", "status": "in_progress", "coder": "coder_vm_01"},
    {"task_id": "TSK-0068", "status": "in_progress", "coder": "coder_vm_02"}
  ],
  "blocked_tasks": [],
  "coder_queue_depth": 2,
  "tool_requests_pending": 0,
  "issues": []
}
```

---

## `TCM` — task_completion

**What it does:** SPM declares a task done. Requires three non-empty sign-off IDs.

**Who sends it:** `spm` → `ppm`

**Example:**

```json
{
  "schema_version": "1.0",
  "message_type": "task_completion",
  "timestamp": "2026-04-20T16:45:00Z",
  "from_role": "spm_vm_lifecycle",
  "to_role": "ppm",
  "message_id": "TCM-SPM-VM-LIFECYCLE-0009",
  "status": "unread",
  "task_id": "TSK-0067",
  "cr_sign_off_id": "RRS-CR-0022",
  "qa_sign_off_id": "NTF-QA-0015",
  "doc_sign_off_id": "NTF-DOC-0008",
  "completion_notes": "Graceful shutdown implemented and tested. 30s timeout default, configurable via config.toml."
}
```

---

## `CSD` — coder_scale_decision

**What it does:** PPM logs a scale-up or scale-down of the coder pool. This is a log record, not an inbox message.

**Who sends it:** `ppm` (log record) + NTF to PM

**Example:**

```json
{
  "schema_version": "1.0",
  "message_type": "coder_scale_decision",
  "timestamp": "2026-04-20T17:15:00Z",
  "decision_id": "CSD-PPM-0004",
  "direction": "scale_up",
  "trigger": "2 SPMs reported coder_queue_depth > 3",
  "affected_subsystems": ["spm_storage", "spm_networking"],
  "coders_added": 2,
  "new_total_coders": 8,
  "consecutive_cycles_at_threshold": 0,
  "notification_message_id": "NTF-PPM-0091"
}
```

---

## `SRQ` — status_report_request

**What it does:** PPM triggers the cycle by asking a role for a status report by a deadline.

**Who sends it:** `ppm` → any internal role

**Example:**

```json
{
  "schema_version": "1.0",
  "message_type": "status_report_request",
  "timestamp": "2026-04-20T08:00:00Z",
  "from_role": "ppm",
  "to_role": "spm_vm_lifecycle",
  "message_id": "SRQ-PPM-0112",
  "status": "unread",
  "report_for_date": "2026-04-20",
  "report_due_by": "2026-04-20T17:00:00Z",
  "fields_required": [
    "overall_status",
    "active_tasks",
    "blocked_tasks",
    "coder_queue_depth",
    "issues"
  ]
}
```

---

## `DCF` — dependency_conflict

**What it does:** PPM logs a dependency conflict. Resolved internally if possible, otherwise escalated.

**Who sends it:** `ppm` (log record)

**Example:**

```json
{
  "schema_version": "1.0",
  "message_type": "dependency_conflict",
  "timestamp": "2026-04-20T14:10:00Z",
  "conflict_id": "DCF-PPM-0003",
  "task_a": "TSK-0063",
  "task_b": "TSK-0071",
  "nature": "Both require write access to libvirt connection singleton",
  "resolution": "Sequenced TSK-0063 to run first. Logged and proceeding.",
  "escalation_required": false,
  "escalation_message_id": ""
}
```

---

## v1.3 — Design and reflection

How SD interacts with Marko and releases specifications.

## `DRF` — design_reflection

**What it does:** SD restates Marko's analysis in its own words and asks him to confirm the understanding. Enforces Checkpoint 1.

**Who sends it:** `sd` → `pm` → `client` (Marko)

**Example:**

```json
{
  "schema_version": "1.0",
  "message_type": "design_reflection",
  "timestamp": "2026-04-20T10:00:00Z",
  "from_role": "sd",
  "to_role": "pm",
  "message_id": "DRF-SD-0008",
  "status": "unread",
  "subsystem": "networking",
  "original_analysis_reference": "REQ-CLIENT-0041",
  "reflection": "The networking subsystem must route all VM traffic through a virtual bridge. No direct host-to-VM connection. Isolation between VMs is mandatory by default and can only be relaxed per-pair via explicit ACL. Marko specifically excluded NAT from scope; bridge mode only.",
  "explicit_exclusions_noted": ["NAT", "direct host connection"],
  "open_questions": [
    "Should the bridge name be derivable from VM name, or configured?"
  ],
  "signoff_requested": true
}
```

---

## `SPR` — spec_release

**What it does:** SD releases a specification to PPM for task decomposition. reflection_signoff_id is mandatory.

**Who sends it:** `sd` → `ppm`, `spm`, `sec`, `doc`

**Example:**

```json
{
  "schema_version": "1.0",
  "message_type": "spec_release",
  "timestamp": "2026-04-20T13:00:00Z",
  "from_role": "sd",
  "to_role": "ppm",
  "also_notified": ["spm_networking", "sec", "doc"],
  "message_id": "SPR-SD-0013",
  "status": "unread",
  "subsystem": "networking",
  "spec_version": "1.0",
  "spec_path": "docs/specs/networking_subsystem_v1.0.md",
  "reflection_signoff_id": "RSO-PM-0008",
  "summary": "Networking subsystem. Bridge-mode only. Default isolation with per-pair ACL relaxation.",
  "interfaces_defined": ["NetworkBridge", "ACLManager", "StorageNetworkBridge"],
  "data_exchanges": [
    {"from": "vm_lifecycle", "to": "networking", "payload": "bridge_assignment_request"}
  ],
  "breaking_changes": false
}
```

---

## `DDC` — design_decision

**What it does:** SD records a significant design decision to its state and appends to the spec document. Log record, not an inbox message.

**Who sends it:** `sd` (log record)

**Example:**

```json
{
  "schema_version": "1.0",
  "message_type": "design_decision",
  "timestamp": "2026-04-20T11:30:00Z",
  "decision_id": "DDC-SD-0017",
  "subsystem": "networking",
  "subject": "Bridge name derivation",
  "decision": "Bridge name is configured explicitly, not derived from VM name.",
  "rationale": "Derivation couples network config to VM naming. VMs rename over time; bridges rarely do. Explicit config survives renames.",
  "alternatives_considered": [
    "Derive from VM name (rejected: rename coupling)",
    "Random UUID (rejected: not human-readable)"
  ],
  "spec_section": "networking_v1.0 section 3.1"
}
```

---

## `RSO` — reflection_signoff

**What it does:** PM writes this on Marko's behalf after consulting him. Unlocks SD to proceed with spec release.

**Who sends it:** `pm` (on client's behalf) → `sd`

**Example:**

```json
{
  "schema_version": "1.0",
  "message_type": "reflection_signoff",
  "timestamp": "2026-04-20T12:30:00Z",
  "from_role": "pm",
  "to_role": "sd",
  "message_id": "RSO-PM-0008",
  "status": "unread",
  "in_response_to": "DRF-SD-0008",
  "signoff_status": "approved_with_corrections",
  "client_corrections": "Bridge name should be configured, not derived. Marko confirmed this explicitly.",
  "signoff_authority": "pm on behalf of Marko Tahvanainen",
  "signoff_timestamp": "2026-04-20T12:28:00Z"
}
```

---

## v1.5 — Execution layer

How SPM coordinates coders, tools, and reviews.

## `CAG` — coder_assignment

**What it does:** SPM gives a coder a specific build task with technical detail beyond the PPM description.

**Who sends it:** `spm` → `coder`

**Example:**

```json
{
  "schema_version": "1.0",
  "message_type": "coder_assignment",
  "timestamp": "2026-04-20T10:00:00Z",
  "from_role": "spm_vm_lifecycle",
  "to_role": "coder_vm_01",
  "message_id": "CAG-SPM-VM-LIFECYCLE-0029",
  "status": "unread",
  "task_id": "TSK-0067",
  "title": "Graceful VM shutdown",
  "technical_detail": "Use virDomainShutdown with VIR_DOMAIN_SHUTDOWN_ACPI_POWER_BTN flag. Poll every 500ms for state change. On timeout (configurable, default 30s), fall back to virDomainDestroy.",
  "interface_contracts": ["vm_lifecycle_v1.3 section 2.4"],
  "tools_available": ["libvirt_rust_wrapper v0.3.1"],
  "supersedes": "",
  "design_language_reminder": "Error codes must follow VM_* prefix convention."
}
```

---

## `TRQ` — tool_request

**What it does:** SPM asks TM for a tool that does not yet exist. The task waits until the tool arrives.

**Who sends it:** `spm` → `tm`

**Example:**

```json
{
  "schema_version": "1.0",
  "message_type": "tool_request",
  "timestamp": "2026-04-20T09:45:00Z",
  "from_role": "spm_vm_lifecycle",
  "to_role": "tm_vm_lifecycle",
  "message_id": "TRQ-SPM-VM-LIFECYCLE-0011",
  "status": "unread",
  "tool_name": "libvirt_state_poller",
  "purpose": "Poll virDomainGetState at configurable interval with timeout. Needed by TSK-0067 and TSK-0068.",
  "inputs": "domain_handle, poll_interval_ms, timeout_ms",
  "outputs": "final_state, elapsed_ms",
  "cross_subsystem_candidate": true,
  "blocks_tasks": ["TSK-0067", "TSK-0068"],
  "priority": "high"
}
```

---

## `RRQ` — review_request

**What it does:** SPM asks CR to review code. self_review_notes are mandatory.

**Who sends it:** `spm` → `cr`

**Example:**

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
  "self_review_notes": "Reviewed the 30s timeout path carefully. Tests cover both ACPI success and timeout fallback. One concern: the poll interval is not unit-tested for the 0ms edge case. Ignored this as the config validator rejects 0ms.",
  "interface_contracts_verified": ["vm_lifecycle_v1.3 section 2.4"],
  "design_language_compliance_confirmed": true
}
```

---

## `RRS` — review_result

**What it does:** CR returns a review decision. Approved IDs become the cr_sign_off_id in task_completion.

**Who sends it:** `cr` → `spm`

**Example:**

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

## `CPK` — code_package

**What it does:** Coder submits finished code to SPM. Must include passing test results.

**Who sends it:** `coder` → `spm`

**Example:**

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
  "dependencies_used": ["libvirt_rust_wrapper v0.3.1", "libvirt_state_poller v0.1.0"],
  "build_command": "cargo build --release"
}
```

---

## v1.6 — Tool delivery and coverage

## `TDL` — tool_delivery

**What it does:** TM (or MTM) delivers a finished tool to SPM. Unblocks tasks that were waiting.

**Who sends it:** `tm` or `mtm` → `spm`

**Example:**

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
  "usage": "from libvirt_state_poller import poll_state; result = poll_state(domain, interval_ms=500, timeout_ms=30000)",
  "inputs": "domain_handle, poll_interval_ms, timeout_ms",
  "outputs": "final_state, elapsed_ms",
  "test_coverage": "92 percent, covers happy path and timeout",
  "doc_path": "tools/vm_lifecycle/libvirt_state_poller/README.md",
  "unblocks_tasks": ["TSK-0067", "TSK-0068"]
}
```

---

## `CVN` — coverage_notice

**What it does:** MTM notifies a TM that it is entering or exiting coverage mode for that subsystem.

**Who sends it:** `mtm` → `tm`

**Example:**

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

## `TUP` — tool_update

**What it does:** TM pushes an update to an existing tool. Separate type code so it never collides with initial delivery.

**Who sends it:** `tm` → `spm`

**Example:**

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

## `STN` — shared_tool_notice

**What it does:** MTM announces a new shared tool is available. Any local tools it supersedes must be deprecated.

**Who sends it:** `mtm` → all consumers

**Example:**

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
  "tool_path": "tools/shared/libvirt_state_poller/",
  "tool_version": "1.0.0",
  "description": "Polls libvirt domain state with configurable interval and timeout. Works across all subsystems.",
  "usage": "from shared.libvirt_state_poller import poll_state",
  "inputs": "domain_handle, poll_interval_ms, timeout_ms, optional callback",
  "outputs": "final_state, elapsed_ms",
  "test_coverage": "95 percent. All subsystem integration paths tested.",
  "doc_path": "tools/shared/libvirt_state_poller/README.md",
  "absorbed_from": "tm_vm_lifecycle",
  "supersedes_local_tools": ["tm_vm_lifecycle"],
  "intended_consumers": ["spm_vm_lifecycle", "spm_storage", "spm_networking", "cr", "test"]
}
```

---

## `DPN` — deprecation_notice

**What it does:** MTM announces a shared tool is being retired. Consumers must migrate by the deprecation date.

**Who sends it:** `mtm` → affected consumers

**Example:**

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
  "tool_version": "1.0.0",
  "deprecation_date": "2026-06-01",
  "reason": "Superseded by libvirt_state_observer v2.0.0 which adds event-stream subscription without polling overhead.",
  "replacement_tool": "libvirt_state_observer",
  "replacement_path": "tools/shared/libvirt_state_observer/",
  "migration_notes": "Call sites using poll_state(domain, 500, 30000) become observe_state(domain, timeout_ms=30000). Callback is now mandatory.",
  "breaking_changes": true,
  "affected_roles": ["spm_vm_lifecycle", "spm_storage", "cr"]
}
```

---

## Notes on the examples

Values used here are illustrative. Task IDs, message IDs, and timestamps are realistic but invented. Copy the structure, replace the values.

Three things to remember when adapting:

1. **Increment the sequence counter** in your state file after every send. Never reuse IDs.
2. **Write the notification** to the recipient's inbox after every outbox write. Without it the message is invisible.
3. **Set `status: unread`** on outgoing messages. The recipient changes it to `pending` when acted upon, `done` when resolved.