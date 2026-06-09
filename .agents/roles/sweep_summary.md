# Role file sweep — Marko → the Operator (v2.10 follow-up)

Companion to `claude_md_v2_10_patch.md`. The CLAUDE.md patch abstracted the human role; this sweep aligns the role files with that abstraction.

---

## Scope summary

Across the 15 role files in `.agents/roles/`, the Marko sweep affects three files. The other twelve had zero references.

| File | Marko hits | Action |
|------|------------|--------|
| `pm.md` | 16 | Full sweep + 2 pronoun fixes |
| `sd.md` | 14 | Full sweep + 1 sentence-start capitalisation |
| `qa.md` | 1 | Single possessive sweep |
| `cmvc.md`, `cr.md`, `doc.md`, `infra.md`, `int.md`, `mtm.md`, `ppm.md`, `sec.md`, `spm.md`, `test.md`, `tm.md`, `ux.md` | 0 | No edits required |

Updated files are in `/mnt/user-data/outputs/role_files_v2_10/` ready to drop into `.agents/roles/`.

---

## Sweep rules applied

1. `Marko's` → `the Operator's` (possessive, mid-sentence)
2. `Marko` → `the Operator` (bare name, mid-sentence)
3. Sentence-start and bullet-start cases: `Marko` → `The Operator` (capital T)
4. Gendered pronouns referring to Marko: `his` → `their` on the two specific lines where the antecedent was Marko

The Operator is a role, not a gendered person. Pronouns are now neutral throughout.

---

## pm.md — 16 edits + 2 pronoun fixes

All references swept. Two lines required pronoun adjustment because the antecedent changed:

| Line | Before | After |
|------|--------|-------|
| 125 | Communicate on Marko's behalf without **his** explicit instruction | Communicate on the Operator's behalf without **their** explicit instruction |
| 227 | **Marko** only sees things that genuinely need **his** attention | **The Operator** only sees things that genuinely need **their** attention |

Other 14 edits: standard sweep, no judgement calls required. Full list available by diffing the original against the file in `/mnt/user-data/outputs/role_files_v2_10/pm.md`.

---

## sd.md — 14 edits + 1 capitalisation

One line started with "Marko" mid-paragraph; capital T preserved on rewrite:

| Line | Before | After |
|------|--------|-------|
| 19 | You do not produce analysis. **Marko** does. | You do not produce analysis. **The Operator** does. |

Other 13 edits: standard sweep. Section heading "## Consuming analysis from Marko" became "## Consuming analysis from the Operator".

---

## qa.md — 1 edit

| Line | Before | After |
|------|--------|-------|
| 120 | List any items requiring **Marko's** input (via PM escalation). | List any items requiring **the Operator's** input (via PM escalation). |

---

## Housekeeping — applied in same pass

Three items surfaced during the file audit. All resolved in this output bundle.

### 1. Duplicate file: `infra_.md` and `infra.md` — **resolved**

These two files were byte-identical (verified with `diff`). Upload artefact. The output bundle contains one canonical `infra.md`. `infra_.md` is omitted.

### 2. Stale file: `tm.md` vs `tm_.md` — **resolved**

`tm_.md` was the v2.4-aware version (MTM tool gate, TAP/TRJ direct, subsystem TM vs cross-subsystem hierarchy). `tm.md` was pre-gate and contradicted CLAUDE.md v2.4+. The output bundle contains the v2.4-aware content under the canonical name `tm.md`.

### 3. Stale role file footers — **resolved**

`cmvc.md`, `infra.md`, and `int.md` all carried `Governed by CLAUDE.md v2.5` footers. All three now read `Governed by CLAUDE.md v2.10`.

Verified that no other role file had a stale `Governed by CLAUDE.md` footer. The `doc.md`, `sec.md`, `test.md`, and `ux.md` files use a different footer style that does not declare a CLAUDE.md version. Their convention is left untouched.

---

## Verification

After applying the swept files: `grep -ri "marko" .agents/roles/` should return no matches. If it does, the sweep is incomplete.

---

## Status after this sweep is applied

- CLAUDE.md v2.10 abstraction is consistent with the role files.
- The constitution and the role definitions agree.
- Forking the project remains a one-line edit: change `Operator: Marko Tahvanainen` in CLAUDE.md and inherit clean.

Open items unchanged from v2.10 patch status:

1. Malome bootstrap protocol specification — still `pending_definition`.
2. Interface Specification v2.0 finalisation — still pending.

Housekeeping items from the original sweep are now closed.
