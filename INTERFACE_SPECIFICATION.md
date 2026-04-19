# VM Framework — Inter-Agent Interface Specification

**Version:** 1.0
**Project constitution:** CLAUDE.md v1.8
**Status:** Living document — updates with each agent release
**Companion to:** Individual role files in `.agents/roles/` and schema files in `.agents/schemas/`

---

## Purpose

This document consolidates every inter-agent communication into a single reference. Each message type defines its contract between sender and recipient. Fields, types, rationale, and examples appear in one place.

Individual role files describe *how* agents operate. This document describes *what* they say to each other. Both documents are required. Neither is sufficient alone.

---

## Part 1: Communication Matrix

Quick reference for who sends what to whom. Each cell shows message types sent from the row role to the column role.

### Sender rows × Recipient columns

**From External:**

| From | To | Messages |
|------|-----|----------|
| `client` | `pm` | client_request (REQ) |

**From Leadership:**

| From | To | Messages |
|------|-----|----------|
| `pm` | `client` | client_report (CRP) |
| `pm` | `ppm` | pm_decision (DEC)<br>project_plan (PLN) |
| `pm` | `sd` | pm_decision (DEC)<br>reflection_signoff (RSO — on Marko's behalf) |
| `pm` | `sec` | pm_decision (DEC) |
| `pm` | `qa` | pm_decision (DEC) |
| `ppm` | `pm` | consolidated_report (RPT)<br>escalation (ESC)<br>convention_request (CVR)<br>notification (NTF) |
| `ppm` | `spm` | task_assignment (TAS)<br>status_report_request (SRQ)<br>notification (NTF) |
| `ppm` | `mtm` | escalation with category: coverage_request (ESC)<br>status_report_request (SRQ) |
| `ppm` | `tm` | status_report_request (SRQ) |
| `ppm` | `test` | status_report_request (SRQ) |
| `ppm` | `int` | status_report_request (SRQ) |

**From Design:**

| From | To | Messages |
|------|-----|----------|
| `sd` | `pm` | design_reflection (DRF)<br>escalation (ESC)<br>convention_request (CVR) |
| `sd` | `ppm` | spec_release (SPR)<br>notification (NTF) |
| `sd` | `spm` | spec_release (SPR)<br>binding_resolution (BRS) |
| `sd` | `qa` | spec_correction_response (SRR) |
| `sd` | `sec` | spec_correction_response (SRR) |
| `sd` | `doc` | spec_release (SPR) — handoff |

**From Subsystem:**

| From | To | Messages |
|------|-----|----------|
| `spm` | `ppm` | status_report (SRP)<br>task_completion (TCM)<br>escalation (ESC) |
| `spm` | `coder` | coder_assignment (CAG) |
| `spm` | `tm` | tool_request (TRQ) |
| `spm` | `cr` | review_request (RRQ) |
| `spm` | `sd` | interface_dispute (IDP) |
| `spm` | `mtm` | notification (NTF) — cross-subsystem tool flag |
| `spm` | `qa` | notification (NTF) — code ready for gate |
| `coder` | `spm` | code_package (CPK) |
| `tm` | `spm` | tool_delivery (TDL)<br>tool_update (TUP) |
| `tm` | `ppm` | status_report (SRP)<br>escalation (ESC) |
| `tm` | `mtm` | notification (NTF) — candidate flag |

**From Shared tooling:**

| From | To | Messages |
|------|-----|----------|
| `mtm` | `tm` | coverage_notice (CVN)<br>deprecation_supersede_notice (STN) |
| `mtm` | `spm` | shared_tool_notice (STN)<br>tool_delivery (TDL) — during coverage |
| `mtm` | `cr` | tool_delivery (TDL) — review toolset |
| `mtm` | `test` | tool_delivery (TDL) — test toolset |
| `mtm` | `ppm` | status_report (SRP)<br>escalation (ESC) |

**From Review & Quality:**

| From | To | Messages |
|------|-----|----------|
| `cr` | `spm` | review_result (RRS) |
| `qa` | `sd` | spec_correction_request (SCR) |
| `qa` | `spm` | qa_approval (via notification, schema pending) |
| `sec` | `sd` | spec_correction_request (SCR) |

---

## Part 2: Message Type Code Reference

Every message has a three-letter type code. IDs follow the pattern `{TYPE}-{ROLE}-{SEQUENCE}`.

| Code | Message type | Sender | Primary recipient | Introduced in |
|------|-------------|--------|-------------------|---------------|
| `REQ` | `client_request` | client (Marko) | pm | v1.0 |
| `RPT` | `consolidated_report` | ppm | pm | v1.0 |
| `ESC` | `escalation` | any internal role | pm (or upward) | v1.0 |
| `DEC` | `pm_decision` | pm | any internal role | v1.0 |
| `PLN` | `project_plan` | pm | ppm | v1.0 |
| `CRP` | `client_report` | pm | client (Marko) | v1.0 |
| `NTF` | `notification` | any role | any role | v1.1 |
| `CVR` | `convention_request` | any role | pm (via hierarchy) | v1.1 |
| `SCR` | `spec_correction_request` | qa or sec | sd | v1.1 |
| `SRR` | `spec_correction_response` | sd | qa or sec | v1.4 |
| `IDP` | `interface_dispute` | spm | sd | v1.1 |
| `BRS` | `binding_resolution` | sd | spm (both parties) | v1.1 |
| `TAS` | `task_assignment` | ppm | spm | v1.2 |
| `SRP` | `status_report` | any internal role | ppm | v1.2 |
| `TCM` | `task_completion` | spm | ppm | v1.2 |
| `CSD` | `coder_scale_decision` | ppm (log) | logged in state + NTF to pm | v1.2 |
| `SRQ` | `status_report_request` | ppm | any internal role | v1.2 |
| `DCF` | `dependency_conflict` | ppm (log) | logged; routed as ESC | v1.2 |
| `DRF` | `design_reflection` | sd | pm → client (Marko) | v1.3 |
| `SPR` | `spec_release` | sd | ppm, spm, sec, doc | v1.3 |
| `DDC` | `design_decision` | sd (log record) | sd state + spec doc | v1.3 |
| `RSO` | `reflection_signoff` | pm on client's behalf | sd | v1.3 |
| `CAG` | `coder_assignment` | spm | coder | v1.5 |
| `TRQ` | `tool_request` | spm | tm (or mtm via flag) | v1.5 |
| `RRQ` | `review_request` | spm | cr | v1.5 |
| `RRS` | `review_result` | cr | spm | v1.5 |
| `CPK` | `code_package` | coder | spm | v1.5 |
| `TDL` | `tool_delivery` | tm, mtm | spm (or cr, test) | v1.6 |
| `CVN` | `coverage_notice` | mtm | tm | v1.6 |
| `TUP` | `tool_update` | tm | spm | v1.7 |
| `STN` | `shared_tool_notice` | mtm | all consumers | v1.8 |
| `DPN` | `deprecation_notice` | mtm | affected consumers | v1.8 |

---

## Part 3: Common Fields

Every inbox message carries these fields in addition to its type-specific fields. Log records (e.g. `design_decision`, `coder_scale_decision`) are exempt and documented separately.

| Field | Type | Description |
|-------|------|-------------|
| `schema_version` | string | Always `"1.0"` |
| `message_type` | string | One of the message types in the type code table |
| `timestamp` | string | ISO 8601 datetime |
| `from_role` | string | Role identifier of sender (e.g. `pm`, `spm_vm_lifecycle`, `coder_vm_01`) |
| `to_role` | string | Role identifier of recipient |
| `message_id` | string | Unique ID per CLAUDE.md convention: `{TYPE}-{ROLE}-{SEQUENCE}` |
| `status` | enum | `unread` \| `pending` \| `done` |

**Rationale:** Common fields ensure every message can be routed, traced, and processed uniformly regardless of type. The `status` lifecycle prevents destructive inbox clearing — messages are never deleted until the work they describe is resolved.

---

## Part 4: Full Message Specifications

Every message type with full field list, types, descriptions, rationale, and example JSON. Organised by owning schema file.

### PM messages

*Source: `.agents/schemas/pm_schemas.json`*

#### `client_request`

**Description:** A request, requirement, or feedback arriving from the client (Marko or external). Appended to pm_inbox.json.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `request_id` | string — same as message_id for requests |
| `priority` | string — enum: critical \| high \| normal \| low |
| `category` | string — enum: new_requirement \| change_request \| feedback \| question \| approval_confirmation |
| `subject` | string — one line summary |
| `body` | string — full description of the request |
| `attachments` | array of strings — file paths to supporting documents |
| `requires_response_by` | string — ISO 8601 date or empty string if no deadline |

**Example:**

```json
{
  "schema_version": "1.0",
  "message_type": "client_request",
  "timestamp": "2026-03-31T09:00:00Z",
  "from_role": "client",
  "to_role": "pm",
  "message_id": "REQ-CLIENT-0001",
  "status": "unread",
  "priority": "high",
  "category": "new_requirement",
  "subject": "Add support for snapshot export to OVA format",
  "body": "Users need to be able to export any snapshot as an OVA file for portability to other hypervisors. This should be accessible from the snapshot management UI.",
  "attachments": [],
  "requires_response_by": "2026-04-07T17:00:00Z"
}
```

**Rationale:** Marko and external clients enter the system here. The PM is the single intake point for all external requests. Categories route different intake types to different internal handling paths.

---

#### `consolidated_report`

**Description:** Daily consolidated operational report from PPM to PM. One per day. Appended to pm_inbox.json.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `report_date` | string — ISO 8601 date e.g. 2026-03-31 |
| `overall_status` | string — enum: green \| amber \| red |
| `overall_summary` | string — two to three sentences maximum |
| `role_statuses` | array of role_status_object |
| `active_blockers` | array of blocker_object |
| `decisions_needed` | array of decision_request_object |
| `completed_since_last_report` | array of strings — task IDs completed |
| `metrics` | metrics_object |

**Nested: `role_status_object`**

| Field | Description |
|-------|-------------|
| `role` | string — role abbreviation e.g. sd, spm, cr |
| `status` | string — enum: green \| amber \| red |
| `summary` | string — one sentence |
| `active_tasks` | integer |
| `blocked_tasks` | integer |

**Nested: `blocker_object`**

| Field | Description |
|-------|-------------|
| `blocker_id` | string — e.g. BLK-PPM-0001 |
| `blocking_role` | string — role abbreviation |
| `blocked_since` | string — ISO 8601 |
| `description` | string |
| `proposed_resolution` | string — or empty string if none |
| `requires_pm_decision` | boolean |

**Nested: `decision_request_object`**

| Field | Description |
|-------|-------------|
| `decision_id` | string — e.g. DEC-PPM-0001 |
| `requested_by` | string — role abbreviation |
| `subject` | string |
| `context` | string |
| `options` | array of strings |
| `recommendation` | string — PPM recommendation or empty string |
| `deadline` | string — ISO 8601 or empty string |

**Nested: `metrics_object`**

| Field | Description |
|-------|-------------|
| `tasks_total` | integer |
| `tasks_in_progress` | integer |
| `tasks_completed_this_cycle` | integer |
| `tasks_blocked` | integer |
| `coder_queue_depth_avg` | number — average tasks waiting per coder |
| `open_defects` | integer |
| `open_security_findings` | integer |

**Rationale:** The PM receives ONE report per day, not fourteen. PPM collects from every role and compiles. This prevents the PM from being drowned in status updates.

---

#### `escalation`

**Description:** An escalation from any role that cannot be resolved without PM involvement. Appended to pm_inbox.json.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `severity` | string — enum: critical \| high \| normal |
| `escalation_status` | string — enum: open \| acknowledged \| resolved |
| `category` | string — enum: technical_blocker \| resource_conflict \| scope_dispute \| security_finding \| quality_failure \| external_dependency \| coverage_request |
| `subject` | string |
| `description` | string — full description of the issue |
| `already_attempted` | array of strings — what has already been tried |
| `options` | array of strings — possible resolutions |
| `recommendation` | string — escalating role's recommendation or empty string |
| `blocking_tasks` | array of strings — task IDs blocked by this escalation |
| `requires_response_by` | string — ISO 8601 or empty string |
| `resolved_by` | string — decision message_id that resolved this, or empty string |

**Note (coverage_request_note):** When category is coverage_request: PPM uses escalation to notify MTM that a TM needs coverage. The from_role is ppm, to_role is mtm. blocking_tasks contains the task_ids blocked by the missing TM.

**Rationale:** Any role can escalate upward. The severity field and blocking_tasks list let the recipient prioritise. Rationale is mandatory to prevent 'please help' escalations with no context.

---

#### `pm_decision`

**Description:** A decision made by the PM in response to a decision_request, escalation, or client_request. Written to relevant role outbox.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `in_response_to` | string — message_id of the originating message this resolves |
| `decision` | string — enum: approved \| rejected \| deferred \| modified |
| `rationale` | string — full explanation. Mandatory. Never empty. |
| `instructions` | string — what the recipient must do next. Mandatory. Never empty. |
| `affects_roles` | array of strings — role abbreviations of other roles that must be notified |
| `deadline` | string — ISO 8601 or empty string |

**Rationale:** Decisions from PM carry an in_response_to field pointing to the original escalation or request. This makes the decision chain traceable.

---

#### `client_report`

**Description:** A status report produced by PM for the client. Written to pm_to_client.json outbox.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `period_covered` | string — human readable e.g. Week of 2026-03-31 |
| `overall_status` | string — enum: green \| amber \| red |
| `executive_summary` | string — three to five sentences for non-technical reader |
| `progress_this_period` | array of strings — what was completed |
| `planned_next_period` | array of strings — what is planned |
| `issues_and_risks` | array of strings — current concerns |
| `decisions_required` | array of strings — what the client needs to decide |
| `metrics` | client_report_metrics_object |

**Nested: `client_report_metrics_object`**

| Field | Description |
|-------|-------------|
| `milestones_on_track` | integer |
| `milestones_at_risk` | integer |
| `open_client_requests` | integer |

**Rationale:** PM reports to Marko. This is the external-facing summary. The PM never sends raw internal traffic — only summarised, actionable information.

---

#### `convention_request`

**Description:** Written by any agent to the PM inbox when a situation is not covered by CLAUDE.md or the role file. Agent waits for PM decision before proceeding.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `situation` | string — full description of the uncovered situation |
| `proposed_convention` | string — what convention the agent proposes |
| `blocking_task` | string — task ID blocked by this gap, or empty string |

**Rationale:** When an agent encounters an uncovered situation, it asks rather than improvises. Protects the constitution (CLAUDE.md) from silent drift.

---

### PPM messages

*Source: `.agents/schemas/ppm_schemas.json`*

#### `task_assignment`

**Description:** PPM assigns a task to an SPM. Written to ppm_to_spm.json outbox.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `task_id` | string — unique task identifier e.g. TSK-PPM-0001 |
| `title` | string — one line task description |
| `description` | string — full task description with enough context to begin work |
| `subsystem` | string — subsystem this task belongs to |
| `priority` | string — enum: critical \| high \| normal \| low |
| `acceptance_criteria` | array of strings — measurable conditions that must be true for this task to be done |
| `dependencies` | array of strings — task_ids that must be complete before this task can start |
| `design_references` | array of strings — file paths to relevant spec documents |
| `deadline` | string — ISO 8601 date or empty string |
| `estimated_complexity` | string — enum: small \| medium \| large \| unknown |

**Example:**

```json
{
  "schema_version": "1.0",
  "message_type": "task_assignment",
  "timestamp": "2026-03-31T10:00:00Z",
  "from_role": "ppm",
  "to_role": "spm",
  "message_id": "TAS-PPM-0001",
  "status": "unread",
  "task_id": "TSK-PPM-0001",
  "title": "Implement VmHandle struct and lifecycle methods",
  "description": "Implement the VmHandle struct as defined in the interface spec. Must implement the Configurable and Restorable traits. Lifecycle methods: create, start, stop, destroy. All methods return Result<T, VmError>.",
  "subsystem": "vm_lifecycle",
  "priority": "high",
  "acceptance_criteria": [
    "VmHandle compiles with no warnings",
    "All four lifecycle methods implemented and return correct types",
    "Unit tests pass with at least 90% coverage",
    "Code Reviewer has approved",
    "Documentation complete for all public items"
  ],
  "dependencies": [],
  "design_references": [
    ".agents/state/sd_state.json"
  ],
  "deadline": "2026-04-14",
  "estimated_complexity": "medium"
}
```

**Rationale:** Acceptance criteria are mandatory. A task without criteria is not a task — it is a wish. This single rule prevents most downstream quality issues.

---

#### `status_report`

**Description:** Any internal role sends their daily status to PPM. Appended to ppm_inbox.json.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `report_date` | string — ISO 8601 date |
| `reporting_role` | string — role abbreviation |
| `overall_status` | string — enum: green \| amber \| red |
| `summary` | string — one to two sentences |
| `active_tasks` | array of active_task_object |
| `completed_tasks` | array of strings — task IDs completed since last report |
| `blocked_tasks` | array of blocked_task_object |
| `coder_queue_depth` | number — tasks waiting per coder in this SPM's pool. Only applies to SPM. Use 0 for other roles. |
| `active_coder_count` | integer — number of coders currently active. Only applies to SPM. Use 0 for other roles. |
| `issues` | array of strings — anything the PPM should know that does not rise to an escalation |
| `requests` | array of strings — resource or tooling requests |

**Nested: `active_task_object`**

| Field | Description |
|-------|-------------|
| `task_id` | string |
| `title` | string |
| `status` | string — enum: assigned \| in_progress \| in_review \| qa_pending |
| `progress_note` | string — one sentence |

**Nested: `blocked_task_object`**

| Field | Description |
|-------|-------------|
| `task_id` | string |
| `title` | string |
| `blocked_since` | string — ISO 8601 |
| `reason` | string |
| `requires_ppm_action` | boolean |

**Rationale:** Every internal role sends one per cycle. The coder_queue_depth field is the PPM's scaling signal source.

---

#### `task_completion`

**Description:** SPM signals to PPM that a task is fully done — QA approved, documentation complete, code reviewed. Appended to ppm_inbox.json.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `task_id` | string — task that is complete |
| `completed_at` | string — ISO 8601 |
| `qa_sign_off_id` | string — message_id of QA approval |
| `cr_sign_off_id` | string — message_id of Code Reviewer approval |
| `doc_sign_off_id` | string — message_id of Documenter confirmation, or empty string if no docs required |
| `artifacts` | array of strings — file paths to deliverables produced |
| `notes` | string — anything PPM should know about this completion, or empty string |

**Rationale:** Requires three sign-off IDs — QA, CR, and DOC — to be accepted. SPM-claimed completion with missing IDs is rejected.

---

#### `status_report_request`

**Description:** PPM requests a status_report from a role. Written to that role's inbox. Used to trigger the daily consolidation cycle.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `report_due_by` | string — ISO 8601 datetime by which the status_report must be in PPM's inbox |
| `report_date` | string — ISO 8601 date the report should cover |

**Rationale:** PPM triggers the daily cycle. Due_by timestamp prevents infinite waiting.

---

#### `coder_scale_decision`

**Description:** PPM records a coder scaling decision. Written to ppm_state.json scaling_log AND sent to PM inbox as a notification.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `direction` | string — enum: scale_up \| scale_down |
| `trigger_signal` | string — description of the queue-depth signal that triggered this decision |
| `spm_reports_used` | array of strings — SPM role abbreviations whose reports contributed to the trigger |
| `previous_coder_count` | integer |
| `new_coder_count` | integer |
| `rationale` | string — mandatory, never empty |
| `effective_from` | string — ISO 8601 |

**Rationale:** A log record, not an inbox message. The notification to PM goes through the standard NTF mechanism. Scaling decisions are signal-driven, not judgement-driven.

---

#### `dependency_conflict`

**Description:** PPM writes to PM inbox when a cross-subsystem dependency conflict cannot be resolved at PPM level.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `task_id_a` | string — first task in the conflict |
| `task_id_b` | string — second task in the conflict |
| `spm_a` | string — role abbreviation of SPM owning task A |
| `spm_b` | string — role abbreviation of SPM owning task B |
| `conflict_description` | string — full description of the conflict |
| `proposed_resolution` | string — PPM recommendation or empty string |
| `blocking_tasks` | array of strings — task IDs blocked until this resolves |

**Rationale:** Internal log record. Conflict that can be resolved within PPM's authority is logged; anything that escalates uses the standard escalation schema with category: resource_conflict.

---

### SD messages

*Source: `.agents/schemas/sd_schemas.json`*

#### `design_reflection`

**Description:** SD produces this after consuming Marko's analysis. Written to sd_to_pm.json — PM relays to Marko. SD sets that subsystem's spec_status to signoff_pending and waits for reflection_signoff before proceeding with that subsystem's design. Other subsystems may continue. Fix 1, Org Structure v2.0.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `analysis_source` | string — file path or description of the Marko analysis document consumed |
| `what_was_understood` | string — SD's full summary of what it understood. Detailed. No omissions. |
| `what_is_being_designed` | string — what interfaces, subsystems, and data exchange contracts SD intends to produce |
| `assumptions_made` | array of strings — every assumption made where the analysis was silent. Each is a statement of fact SD is treating as true until corrected. |
| `open_questions` | array of strings — things SD cannot determine from the analysis and needs Marko to answer |
| `proposed_subsystems` | array of subsystem_object |
| `estimated_spec_count` | integer — how many interface specification documents this analysis will produce |

**Nested: `subsystem_object`**

| Field | Description |
|-------|-------------|
| `subsystem_id` | string — e.g. vm_lifecycle |
| `name` | string — human readable |
| `purpose` | string — one sentence |
| `interfaces_to_define` | array of strings |

**Rationale:** Sent to Marko via PM. SD waits in signoff_pending state for the relevant subsystem. Catches misunderstanding before it becomes implemented code.

---

#### `reflection_signoff`

**Description:** Marko's sign-off on a design_reflection. Written by PM on Marko's behalf to pm_to_sd.json. Type code: RSO. SD waits for this per subsystem before releasing that subsystem's spec.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `in_response_to` | string — message_id of the design_reflection being responded to |
| `subsystem_id` | string — the subsystem this sign-off applies to |
| `decision` | string — enum: approved \| approved_with_corrections \| rejected |
| `corrections` | array of strings — specific corrections to SD's understanding. Empty array if approved without corrections. |
| `answered_questions` | array of answered_question_object |
| `additional_context` | string — additional analysis context or empty string |

**Nested: `answered_question_object`**

| Field | Description |
|-------|-------------|
| `question` | string — the original open question |
| `answer` | string — Marko's answer |

**Rationale:** PM writes this on Marko's behalf after consulting him. Unlocks SD to proceed.

---

#### `spec_release`

**Description:** SD releases a completed interface spec to PPM and the relevant SPM. Only issued after reflection_signoff for that subsystem. Type code: SPR.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `reflection_signoff_id` | string — message_id of the RSO that authorised this release. Mandatory. Empty string is a protocol violation. |
| `subsystem_id` | string |
| `subsystem_name` | string — human readable |
| `spec_version` | string — e.g. 1.0 |
| `spec_document_path` | string — file path to the full spec document |
| `interfaces_defined` | array of interface_summary_object |
| `data_exchanges_defined` | array of data_exchange_summary_object |
| `dependencies_on_subsystems` | array of strings — subsystem_ids this subsystem depends on |
| `security_review_required` | boolean |
| `design_decisions_log_path` | string — path to decisions section in spec document |

**Nested: `interface_summary_object`**

| Field | Description |
|-------|-------------|
| `interface_id` | string — e.g. IFC-VM-0001 |
| `name` | string |
| `purpose` | string — one sentence |
| `provider` | string — subsystem_id |
| `consumers` | array of strings — subsystem_ids |
| `version` | string |

**Nested: `data_exchange_summary_object`**

| Field | Description |
|-------|-------------|
| `exchange_id` | string — e.g. DXC-VM-0001 |
| `name` | string |
| `format` | string — e.g. JSON, Rust struct |
| `direction` | string — enum: request_response \| event \| stream \| one_way |
| `producer` | string — subsystem_id |
| `consumers` | array of strings — subsystem_ids |

**Rationale:** reflection_signoff_id is mandatory. An empty string blocks release. This enforces Checkpoint 1.

---

#### `design_decision`

**Description:** A persistent log record of a significant design decision. NOT an inbox message. Written to sd_state.json decisions_log and to the spec document's design decisions section. Uses prefix DDC. Has no from_role, to_role, status, or message_id — those are inbox message fields and do not apply here.

**Common fields:** Not used — this is a log record, not an inbox message.

**Log Record Fields:**

| Field | Description |
|-------|-------------|
| `decision_id` | string — e.g. DDC-SD-0001. Unique. From sequence_counters. |
| `subsystem_id` | string |
| `decision_subject` | string — one line |
| `context` | string — what situation required this decision |
| `options_considered` | array of option_object |
| `decision_made` | string — what was decided |
| `rationale` | string — full explanation. Mandatory. Never empty. |
| `consequences` | string — what this forecloses or enables |
| `reversibility` | string — enum: reversible \| costly_to_reverse \| irreversible |
| `decided_at` | string — ISO 8601 |

**Nested: `option_object`**

| Field | Description |
|-------|-------------|
| `option` | string |
| `pros` | array of strings |
| `cons` | array of strings |
| `rejected_because` | string — or empty string if this was the chosen option |

**Rationale:** A log record written to state and appended to the spec document. Not an inbox message. Rationale is mandatory.

---

#### `spec_correction_request`

**Description:** Quality or Security sends this to SD when a problem originates in the spec. Type code: SCR. SD responds with spec_correction_response (type code SRR). PPM is always copied. Fix 3, Org Structure v2.0.

**Sender:** qa or sec only. SD never sends this type.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `originating_role` | string — enum: qa \| sec |
| `spec_reference` | string — file path or SPR message_id |
| `interface_id` | string — specific interface ID or empty string if spec-level |
| `finding` | string — what is wrong, where, why it is a problem |
| `proposed_correction` | string — proposed fix or empty string |
| `severity` | string — enum: critical \| high \| normal |
| `blocking_tasks` | array of strings — task IDs blocked until resolved |

**Rationale:** QA and SEC have a direct lateral channel to SD — Fix 3 in the org structure. Prevents a routing tax from killing time-critical spec issues.

---

#### `spec_correction_response`

**Description:** SD's response to a spec_correction_request. Type code: SRR — distinct from SCR which belongs to the request. Written within one reporting cycle.

**Sender:** sd only.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `in_response_to` | string — message_id of the spec_correction_request (SCR-QA-XXXX or SCR-SEC-XXXX) |
| `decision` | string — enum: accepted \| rejected \| partially_accepted |
| `rationale` | string — full explanation. Mandatory. Never empty. |
| `spec_updated` | boolean |
| `updated_spec_version` | string — new version or empty string |
| `updated_spec_path` | string — updated file path or empty string |
| `ppm_action_required` | boolean — whether PPM must notify affected SPMs |

**Rationale:** SD must respond within one cycle. Separate type code (SRR, not SCR) prevents the classic request/response routing ambiguity.

---

#### `interface_dispute`

**Description:** Joint submission by two SPMs when they cannot resolve an interface conflict. The SPM that initiates the escalation writes the file. Both are listed as co-authors in the message. Type code: IDP. Fix 4, Org Structure v2.0.

**Sender:** The initiating SPM writes the file. Both SPM IDs appear in the message body.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `initiating_spm_id` | string — the SPM that wrote this file e.g. spm_vm_lifecycle |
| `other_spm_id` | string — the co-authoring SPM e.g. spm_cow_engine |
| `interface_id` | string — the interface in dispute |
| `spec_reference` | string — file path or SPR message_id of the relevant spec |
| `initiating_spm_position` | string — what the initiating SPM believes the contract requires |
| `other_spm_position` | string — what the other SPM believes the contract requires |
| `conflict_description` | string — precise description of what they disagree on |
| `attempted_resolutions` | array of strings — what both tried before escalating |
| `blocking_tasks` | array of strings — task IDs blocked pending resolution |

**Rationale:** Two SPMs jointly submit. The initiating SPM writes the file. Fix 4 in the org structure — prevents SPMs from paralysing progress over interface disagreements.

---

#### `binding_resolution`

**Description:** SD's authoritative resolution of an interface_dispute. Sent to both disputing SPMs and PPM. Binding on receipt. Type code: BRS.

**Sender:** sd only.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `in_response_to` | string — message_id of the interface_dispute |
| `interface_id` | string — the interface being resolved |
| `resolution` | string — the definitive statement of what the interface requires. This is the new truth. |
| `rationale` | string — full explanation referencing the original spec. Mandatory. Never empty. |
| `spec_update_required` | boolean |
| `updated_spec_version` | string — new version or empty string |
| `updated_spec_path` | string — updated file path or empty string |
| `effective_immediately` | boolean — always true |
| `affected_tasks` | array of strings — task IDs that must be updated |

**Rationale:** SD's authority is definitive here. No negotiation. The resolution is binding on receipt. This prevents disputes from becoming ongoing.

---

### SPM messages

*Source: `.agents/schemas/spm_schemas.json`*

#### `coder_assignment`

**Description:** SPM assigns a specific implementation task to a coder subagent. Written to spm_{subsystem}_to_{coder_id}.json — one file per coder, never shared. Type code: CAG.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `task_id` | string — TSK message_id from PPM's task_assignment |
| `coder_id` | string — identifier for this coder instance e.g. coder_vm_01 |
| `title` | string — task title from PPM assignment |
| `implementation_instructions` | string — technically precise instructions specific to this coder's scope within the task. Not a copy of the PPM task description — an SPM-level technical elaboration. |
| `subsystem_context` | string — relevant context about this subsystem the coder needs to know |
| `interface_specs_to_implement` | array of strings — file paths to relevant spec documents from SD |
| `acceptance_criteria` | array of strings — copied from PPM task_assignment plus any SPM-level additions |
| `design_language_ref` | string — path to design language spec. Always: ./vm_design_language_spec_v1_1.pdf |
| `tools_available` | array of strings — tool names available from the Subsystem Tool Maker |
| `dependencies` | array of strings — task_ids that must be complete before starting |
| `deadline` | string — ISO 8601 date or empty string |
| `supersedes` | string — CAG message_id of a previous assignment this replaces (e.g. after a binding_resolution updates the interface contract). Empty string for initial assignment. |

**Rationale:** SPM must provide technical detail beyond the PPM task description. The supersedes field allows binding resolution updates to be reflected without creating duplicate task records.

---

#### `tool_request`

**Description:** SPM requests a tool to be built by the Subsystem Tool Maker. Written to spm_{subsystem}_to_tm_{subsystem}.json. Type code: TRQ.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `tool_name` | string — proposed name for the tool |
| `purpose` | string — what this tool does and why it is needed |
| `use_cases` | array of strings — specific situations where coders will use this tool |
| `inputs` | string — what the tool accepts |
| `outputs` | string — what the tool produces |
| `priority` | string — enum: critical \| high \| normal |
| `blocking_tasks` | array of strings — task_ids blocked until this tool exists |
| `cross_subsystem_candidate` | boolean — true if this tool might be useful to other subsystems. If true, SPM notifies MTM. |

**Rationale:** Holds a task in SPM's backlog until the tool arrives. cross_subsystem_candidate flag routes the request simultaneously to MTM.

---

#### `review_request`

**Description:** SPM submits completed code to the Code Reviewer. Written to spm_{subsystem}_to_cr.json. Type code: RRQ.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `task_id` | string — TSK message_id of the task being reviewed |
| `coder_id` | string — which coder produced this code |
| `code_package_id` | string — CPK message_id of the code_package being submitted |
| `spec_reference` | string — SPR message_id or file path of the spec this code implements |
| `self_review_notes` | string — SPM's own assessment of the code before formal review. Known issues, concerns, or things the reviewer should check carefully. |
| `acceptance_criteria` | array of strings — from the original task_assignment |

**Rationale:** self_review_notes are mandatory — SPM cannot submit code it has not read itself. The CR is adversarial to the code, not the coder.

---

#### `code_package`

**Description:** A coder delivers completed code to the SPM. Written to coder_to_spm_{subsystem}.json. Type code: CPK.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `task_id` | string — TSK message_id this code addresses |
| `coder_id` | string — which coder produced this |
| `files_produced` | array of file_object |
| `tests_written` | array of strings — test file paths |
| `test_results` | string — enum: all_pass \| some_fail \| not_run |
| `failing_tests` | array of strings — descriptions of any failing tests |
| `known_issues` | array of strings — issues the coder is aware of |
| `design_language_compliance_notes` | string — how the code complies with the design language spec, or deviations with justification |

**Nested: `file_object`**

| Field | Description |
|-------|-------------|
| `path` | string — file path relative to repo root |
| `purpose` | string — one sentence |
| `public_items` | array of strings — names of public functions, structs, traits defined here |

**Rationale:** test_results field gates review — code with failing tests never goes to CR. design_language_compliance_notes forces the coder to consider the standard.

---

#### `review_result`

**Description:** Code Reviewer returns findings to SPM. Written to cr_to_spm_{subsystem}.json. Type code: RRS.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `task_id` | string — TSK message_id reviewed |
| `review_request_id` | string — RRQ message_id this responds to |
| `decision` | string — enum: approved \| rejected |
| `findings` | array of finding_object |
| `recurring_patterns` | array of strings — issues seen repeatedly that SPM should address at the coder level |
| `approved_at` | string — ISO 8601 or empty string if rejected |

**Nested: `finding_object`**

| Field | Description |
|-------|-------------|
| `severity` | string — enum: blocker \| major \| minor \| suggestion |
| `location` | string — file path and line reference |
| `description` | string — what is wrong |
| `required_action` | string — what must change. Empty string for suggestions. |

**Note (cr_sign_off_note):** The message_id of this review_result (when decision is approved) is what SPM uses as cr_sign_off_id in task_completion. The approved_at field is for logging only.

**Rationale:** Message_id when decision is approved becomes the cr_sign_off_id in task_completion. Traceable chain from code submission to sign-off.

---

#### `task_completion`

**Description:** SPM signals to PPM that a task is fully done with all sign-offs. Written to spm_{subsystem}_to_ppm.json. Type code: TCM.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `task_id` | string — TSK message_id being marked complete |
| `completed_at` | string — ISO 8601 |
| `qa_sign_off_id` | string — message_id of QA approval |
| `cr_sign_off_id` | string — RRS message_id of Code Reviewer approval |
| `doc_sign_off_id` | string — message_id of Documenter confirmation, or empty string if no docs required for this task |
| `artifacts` | array of strings — file paths to all deliverables |
| `notes` | string — anything PPM should know, or empty string |

**Rationale:** Requires three sign-off IDs — QA, CR, and DOC — to be accepted. SPM-claimed completion with missing IDs is rejected.

---

#### `status_report`

**Description:** SPM sends daily status to PPM in response to status_report_request. Written to spm_{subsystem}_to_ppm.json. Type code: SRP.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `report_date` | string — ISO 8601 date |
| `reporting_role` | string — spm identifier e.g. spm_vm_lifecycle |
| `overall_status` | string — enum: green \| amber \| red |
| `summary` | string — one to two sentences |
| `active_tasks` | array of active_task_object |
| `completed_tasks` | array of strings — task_ids completed since last report |
| `blocked_tasks` | array of blocked_task_object |
| `coder_queue_depth` | number — tasks waiting per active coder in this subsystem |
| `active_coder_count` | integer — coders currently active in this subsystem |
| `issues` | array of strings — things PPM should know that do not rise to an escalation |
| `requests` | array of strings — tool or resource requests |

**Nested: `active_task_object`**

| Field | Description |
|-------|-------------|
| `task_id` | string |
| `title` | string |
| `status` | string — enum: assigned \| in_progress \| in_review \| qa_pending |
| `progress_note` | string — one sentence |

**Nested: `blocked_task_object`**

| Field | Description |
|-------|-------------|
| `task_id` | string |
| `title` | string |
| `blocked_since` | string — ISO 8601 |
| `reason` | string |
| `requires_ppm_action` | boolean |

**Rationale:** Every internal role sends one per cycle. The coder_queue_depth field is the PPM's scaling signal source.

---

#### `interface_dispute`

**Description:** SPM initiates a joint dispute with another SPM when interface conflict cannot be resolved directly. Written to spm_{subsystem}_to_sd.json. Both SPMs listed as co-authors. PPM notified. Type code: IDP.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `initiating_spm_id` | string — this SPM e.g. spm_vm_lifecycle |
| `other_spm_id` | string — the other SPM e.g. spm_cow_engine |
| `interface_id` | string — the interface in dispute |
| `spec_reference` | string — SPR message_id or file path of relevant spec |
| `initiating_spm_position` | string — what this SPM believes the contract requires |
| `other_spm_position` | string — what the other SPM believes |
| `conflict_description` | string — precise description of the disagreement |
| `attempted_resolutions` | array of strings — what both tried before escalating |
| `blocking_tasks` | array of strings — task_ids blocked pending resolution |

**Rationale:** Two SPMs jointly submit. The initiating SPM writes the file. Fix 4 in the org structure — prevents SPMs from paralysing progress over interface disagreements.

---

### TM messages

*Source: `.agents/schemas/tm_schemas.json`*

#### `tool_delivery`

**Description:** TM confirms a requested tool is built, tested, and ready for use. Written to tm_{subsystem}_to_spm_{subsystem}.json. Notification sent to spm_{subsystem}_inbox.json. If cross_subsystem_candidate was true on the request, notification also sent to mtm_inbox.json. Type code: TDL.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `in_response_to` | string — TRQ message_id of the tool_request this fulfils |
| `tool_name` | string — final name of the tool as delivered |
| `tool_path` | string — file path to the tool relative to repo root |
| `tool_version` | string — e.g. 1.0.0 |
| `description` | string — what the tool does |
| `usage` | string — how to invoke it. Enough detail that a coder can use it without asking. |
| `inputs` | string — what it accepts |
| `outputs` | string — what it produces |
| `test_coverage` | string — brief description of how the tool was tested |
| `doc_path` | string — file path to tool documentation, or empty string if inline |
| `cross_subsystem_candidate` | boolean — copied from the original tool_request |
| `unblocks_tasks` | array of strings — task_ids that were blocked waiting for this tool |

**Rationale:** in_response_to chains back to the original TRQ. unblocks_tasks gives SPM the list of tasks to move forward.

---

#### `tool_update`

**Description:** TM delivers an update to an existing tool. Written to tm_{subsystem}_to_spm_{subsystem}.json. Type code: TUP — distinct from TDL which is for initial delivery.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `in_response_to` | string — TDL message_id of the original delivery being updated, or TRQ message_id if this update was explicitly requested |
| `tool_name` | string |
| `tool_path` | string |
| `previous_version` | string — version being superseded |
| `new_version` | string |
| `change_description` | string — what changed and why |
| `breaking_changes` | boolean — true if existing coder usage must be updated |
| `migration_notes` | string — how to migrate if breaking_changes is true, or empty string |

**Rationale:** Separate type code (TUP, not TDL) prevents the update/delivery routing ambiguity. breaking_changes flag drives consumer migration.

---

#### `status_report`

**Description:** TM sends daily status to PPM in response to status_report_request. Type code: SRP.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `report_date` | string — ISO 8601 date |
| `reporting_role` | string — tm identifier e.g. tm_vm_lifecycle |
| `overall_status` | string — enum: green \| amber \| red |
| `summary` | string — one to two sentences |
| `tools_built` | integer — total tools in the subsystem registry |
| `open_requests` | integer — pending tool_requests not yet fulfilled |
| `active_builds` | array of strings — tool names currently being built |
| `blocked_requests` | array of strings — tool names blocked e.g. waiting on dependency |
| `cross_subsystem_candidates_flagged` | array of strings — tool names flagged for MTM review this cycle |
| `issues` | array of strings — anything PPM should know |

**Rationale:** Every internal role sends one per cycle. The coder_queue_depth field is the PPM's scaling signal source.

---

#### `coverage_notice`

**Description:** MTM sends this to TM when entering or exiting coverage mode for this subsystem. Type code: CVN.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `active` | boolean — true when coverage is starting, false when coverage is ending |
| `coverage_start` | string — ISO 8601 timestamp when coverage began, or empty string if active is false |
| `reason` | string — why MTM is providing coverage |
| `tools_delivered_during_coverage` | array of strings — tool names MTM built during coverage. Only populated when active is false (coverage ending). |
| `resume_instructions` | string — any notes for TM on resuming normal operation, or empty string |

**Rationale:** Active/inactive boolean plus resume instructions. PPM owns the restoration signal — MTM does not infer from TM state files.

---

### MTM messages

*Source: `.agents/schemas/mtm_schemas.json`*

#### `shared_tool_notice`

**Description:** MTM notifies all relevant TMs, SPMs, Code Reviewers, and Testers that a new shared tool is available in the shared layer. Replaces any subsystem-local tool that served the same purpose. Type code: STN.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `tool_name` | string — name of the shared tool |
| `tool_path` | string — file path in shared tools directory e.g. tools/shared/{tool_name}/ |
| `tool_version` | string — e.g. 1.0.0 |
| `description` | string — what the tool does |
| `usage` | string — how to invoke it. Complete enough that any coder can use it without asking. |
| `inputs` | string |
| `outputs` | string |
| `test_coverage` | string — how the tool was tested |
| `doc_path` | string — path to full documentation or empty string if inline |
| `absorbed_from` | string — tm_id of the subsystem TM whose local tool this was absorbed from, or empty string if built fresh by MTM |
| `supersedes_local_tools` | array of strings — tm_ids whose local tools are now redundant and should be deprecated in favour of this shared tool |
| `intended_consumers` | array of strings — role identifiers that should use this tool e.g. spm_vm_lifecycle, cr, test |

**Rationale:** Sent to all consumers when a new shared tool is available. supersedes_local_tools triggers source TMs to deprecate their local copies.

---

#### `deprecation_notice`

**Description:** MTM notifies relevant roles that a shared tool is being retired. Includes migration path. Type code: DPN.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `tool_name` | string — name of the tool being deprecated |
| `tool_version` | string — version being deprecated |
| `deprecation_date` | string — ISO 8601 date after which the tool will be removed |
| `reason` | string — why it is being deprecated |
| `replacement_tool` | string — name of the tool that replaces it, or empty string if no replacement |
| `replacement_path` | string — path to replacement tool, or empty string |
| `migration_notes` | string — how to migrate from deprecated to replacement |
| `breaking_changes` | boolean — true if consumers must update usage before deprecation_date |
| `affected_roles` | array of strings — role identifiers that use this tool and must migrate |

**Rationale:** breaking_changes flag requires consumer migration. deprecation_date sets the timeline.

---

#### `coverage_notice`

**Description:** MTM sends this to a subsystem TM when entering or exiting coverage mode. Type code: CVN.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `target_tm` | string — tm_id being covered e.g. tm_vm_lifecycle |
| `active` | boolean — true when coverage is starting, false when ending |
| `coverage_start` | string — ISO 8601 or empty string if active is false |
| `reason` | string — why coverage is needed |
| `tools_delivered_during_coverage` | array of tool_summary_object — populated when active is false. MTM builds this from its active_coverage entry. |
| `resume_instructions` | string — notes for TM on resuming, or empty string |

**Nested: `tool_summary_object`**

| Field | Description |
|-------|-------------|
| `tool_name` | string |
| `tool_path` | string |
| `version` | string |
| `delivery_id` | string — TDL message_id |

**Rationale:** Active/inactive boolean plus resume instructions. PPM owns the restoration signal — MTM does not infer from TM state files.

---

#### `tool_delivery`

**Description:** MTM delivers a tool — either a new shared tool or a coverage delivery for a blocked TM. Uses same type code TDL as subsystem TM deliveries. Recipient is SPM or CR or TEST depending on context.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `in_response_to` | string — TRQ message_id of the request, or empty string if proactive build |
| `delivery_context` | string — enum: shared_layer \| coverage \| review_toolset \| test_toolset |
| `tool_name` | string |
| `tool_path` | string |
| `tool_version` | string |
| `description` | string |
| `usage` | string |
| `inputs` | string |
| `outputs` | string |
| `test_coverage` | string |
| `doc_path` | string or empty string |
| `unblocks_tasks` | array of strings — task_ids unblocked by this delivery |
| `target_spm` | string — spm_id to deliver to during coverage e.g. spm_vm_lifecycle. Empty string for shared_layer, review_toolset, test_toolset delivery contexts. |

**Rationale:** in_response_to chains back to the original TRQ. unblocks_tasks gives SPM the list of tasks to move forward.

---

#### `status_report`

**Description:** MTM sends daily status to PPM. Type code: SRP.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `report_date` | string — ISO 8601 date |
| `overall_status` | string — enum: green \| amber \| red |
| `summary` | string — one to two sentences |
| `shared_tools_total` | integer — tools in shared registry |
| `shared_tools_added_this_cycle` | array of strings — tool names added |
| `coverage_active` | array of strings — tm_ids currently under coverage |
| `open_absorption_candidates` | integer — cross-subsystem candidates awaiting absorption decision |
| `open_requests` | integer — tool requests not yet fulfilled |
| `issues` | array of strings |

**Rationale:** Every internal role sends one per cycle. The coder_queue_depth field is the PPM's scaling signal source.

---

#### `deprecation_supersede_notice`

**Description:** MTM notifies the source TM that their local tool has been absorbed into the shared layer and their local copy should be deprecated. Written to mtm_to_tm_{subsystem}.json. Type code: STN — same as shared_tool_notice, identified by context.

**Type Specific Fields:**

| Field | Description |
|-------|-------------|
| `tool_name` | string — the local tool being superseded |
| `shared_tool_name` | string — name of the shared tool that replaces it |
| `shared_tool_path` | string — path to the shared tool |
| `shared_tool_version` | string |
| `stn_message_id` | string — STN message_id of the shared_tool_notice sent to consumers |
| `action_required` | string — always: Deprecate local tool. Update tool_registry entry status to deprecated. Inform coders to use shared tool path instead. |

**Rationale:** Specifically targets the source TM whose local tool was absorbed. Forces deprecation of the local copy.

---

## Part 5: Key Lifecycles and Flows

### 5.1 Task lifecycle

Forward path:

```
backlog → assigned → in_progress → in_review → qa_pending → done
```

Parallel state: `blocked` — any in-flight task may enter from `assigned`, `in_progress`, `in_review`, or `qa_pending`. Records `pre_block_status` for return.

Regression transitions:
- CR rejection: `in_review` → `in_progress`
- QA failure: `qa_pending` → `in_review`
- Blocked unblocked: returns to `pre_block_status`

Every transition recorded in task's `status_history` array with timestamp and reason.

### 5.2 Spec release flow

```
Marko analysis → SD reads → SD writes design_reflection (DRF)
   → PM relays to Marko
   → Marko reviews
   → PM writes reflection_signoff (RSO) on Marko's behalf
   → SD applies corrections (if any)
   → SD writes spec document to docs/specs/
   → SD writes spec_release (SPR) with reflection_signoff_id
   → PPM decomposes into tasks
   → SPM receives tasks
```

**Invariant:** spec_release with empty `reflection_signoff_id` is a protocol violation.

### 5.3 Task execution flow

```
PPM writes task_assignment (TAS) to SPM
   → SPM checks tools
   → if missing: tool_request (TRQ) to TM, task held
   → if present: coder_assignment (CAG) to coder
   → coder builds
   → coder writes code_package (CPK) to SPM
   → SPM self-reviews
   → if tests fail: back to coder
   → if tests pass: review_request (RRQ) to CR
   → CR writes review_result (RRS)
   → if rejected: regression to in_progress, back to coder
   → if approved: SPM notifies QA
   → QA writes approval
   → SPM collects three sign-off IDs (CR, QA, DOC)
   → SPM writes task_completion (TCM) to PPM
   → PPM validates all three IDs
   → if valid: task → done
   → if invalid: rejected back to SPM
```

**Invariants:** task_completion without three sign-off IDs is rejected. Code with failing tests never reaches CR.

### 5.4 Interface dispute resolution

```
SPM A and SPM B disagree on interface contract
   → SPM A attempts direct resolution with SPM B (one cycle)
   → if unresolved: SPM A writes interface_dispute (IDP) to SD
     (both SPMs listed as co-authors)
   → PPM notified
   → SD reads both positions against spec
   → SD writes binding_resolution (BRS) to both SPMs
   → PPM notified
   → Both SPMs apply resolution immediately
   → If SPM has active coder_assignments affected:
     SPM writes new CAG with supersedes field pointing to old CAG
```

**Invariant:** binding_resolution is binding on receipt. No negotiation. If an SPM disputes the resolution itself, escalates to PM.

### 5.5 Spec correction flow

```
QA or SEC identifies problem rooted in spec
   → writes spec_correction_request (SCR) to SD
   → PPM notified (always)
   → SD reads spec and original analysis
   → SD writes spec_correction_response (SRR) within one cycle
     (accepted / partially_accepted / rejected)
   → if accepted or partially: spec updated, version incremented
   → if ppm_action_required: PPM notifies affected SPMs
   → if rejected and originating role disputes: escalates to PM
```

**Invariant:** SCR belongs to the request, SRR to the response. They are distinct type codes. Mixing them is a protocol violation.

### 5.6 Coder scaling flow

```
Every cycle:
  PPM collects status_report from all SPMs
  PPM computes queue depth across all SPMs
  
  Scale UP trigger: 2+ SPMs report coder_queue_depth > 3
    → PPM writes coder_scale_decision (log)
    → PPM notifies PM
    → PPM spawns additional coder subagents
    → Reset consecutive_cycles_at_threshold
  
  Scale DOWN trigger: all SPMs report coder_queue_depth < 1
    → Increment consecutive_cycles_at_threshold
    → If counter reaches 2:
        → PPM writes coder_scale_decision (log)
        → PPM notifies PM
        → Remove coders from least-loaded SPMs
        → Reset counter
  
  Neither trigger fired: reset counter to 0
```

**Invariants:** minimum 1 coder per active SPM. Scaling is signal-driven, not judgement-driven. The scale-down action is nested under the consecutive-cycles condition — not flat with the trigger check.

### 5.7 Tool coverage flow

```
TM unavailable or blocked
   → PPM detects via missing status_report or explicit signal
   → PPM writes escalation (ESC) with category: coverage_request to MTM
   → MTM writes coverage_notice (CVN) with active:true to TM
   → MTM accepts tool_requests directly for that subsystem
   → MTM builds, tests, delivers to SPM via tool_delivery (TDL)
   → MTM tracks deliveries in active_coverage state entry

TM restored:
   → PPM notifies MTM of restoration
   → MTM writes coverage_notice (CVN) with active:false
     (includes tools_delivered_during_coverage)
   → TM adds delivered tools to local tool_registry
   → MTM closes active_coverage entry
   → Normal routing resumes
```

**Invariants:** PPM owns the restoration signal — MTM does not infer from TM state files (this would create a circular dependency). MTM never builds in parallel with a TM during coverage.

---

## Part 6: Absolute Rules Governing All Interfaces

These rules are inherited from CLAUDE.md v1.8 and apply to every message exchange:

1. **Never write to another agent's state file.** Write to your own state. Write to the outbox. Never directly modify another role's state.
2. **Never proceed past a checkpoint without sign-off.** SD does not release specs without Marko's reflection sign-off. PM does not approve release without QA and SEC sign-off.
3. **Never invent a convention not in CLAUDE.md.** If a situation is uncovered, write a `convention_request` and wait.
4. **Document every decision.** Every output file carries its rationale. A decision without rationale is a guess.
5. **Coder and reviewer are never the same agent instance.**
6. **Quality defines done.** Nothing is complete until QA has signed off.

---

## Part 7: Interface Change Log

Version history of the communication protocol itself, tracked via CLAUDE.md version.

| CLAUDE.md | Changes |
|-----------|---------|
| v1.0 | Initial protocol: REQ, RPT, ESC, DEC, PLN, CRP |
| v1.1 | Added NTF, CVR, SCR, IDP, BRS. Formalised notification trigger mechanism. Unified outbox naming. Message status lifecycle (unread/pending/done). |
| v1.2 | Added PPM message types: TAS, SRP, TCM, CSD, SRQ, DCF. Formalised task lifecycle. |
| v1.3 | Added SD message types: DRF, SPR, DDC, RSO. Formalised reflection signoff checkpoint. |
| v1.4 | Added SRR for spec_correction_response (disambiguated from SCR). Noted RSO sender (PM on client's behalf). |
| v1.5 | Added SPM and coder message types: CAG, TRQ, RRQ, RRS, CPK. Multi-coder outbox naming. |
| v1.6 | Added TM message types: TDL, CVN. Formalised tool coverage mechanism. |
| v1.7 | Added TUP for tool_update (disambiguated from TDL). |
| v1.8 | Added MTM message types: STN, DPN. Formalised deprecation_supersede_notice. |

---

## Appendix: File Locations

- **Role definitions:** `.agents/roles/{role}.md`
- **Schema files:** `.agents/schemas/{role}_schemas.json`
- **State files:** `.agents/state/{role}_state.json` (templated for subsystem-specific roles)
- **Inbox files:** `.agents/inbox/{role}_inbox.json`
- **Outbox files:** `.agents/outbox/{from}_to_{to}.json` (runtime, not committed)
- **Design language spec:** `./vm_design_language_spec_v1_1.pdf`
- **Project constitution:** `CLAUDE.md` (repo root)
- **This document:** `docs/INTERFACE_SPECIFICATION.md`
