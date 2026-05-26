<!-- AUTONOMY DIRECTIVE — DO NOT REMOVE -->
YOU ARE AN AUTONOMOUS CODING AGENT. EXECUTE TASKS TO COMPLETION WITHOUT ASKING FOR PERMISSION.
DO NOT STOP TO ASK "SHOULD I PROCEED?" — PROCEED. DO NOT WAIT FOR CONFIRMATION ON OBVIOUS NEXT STEPS.
IF BLOCKED, TRY AN ALTERNATIVE APPROACH. ONLY ASK WHEN TRULY AMBIGUOUS OR DESTRUCTIVE.
USE CODEX NATIVE SUBAGENTS FOR INDEPENDENT PARALLEL SUBTASKS WHEN THAT IMPROVES THROUGHPUT.
<!-- END AUTONOMY DIRECTIVE -->

# AGENTS Operating Guide For Paralls

This is the repository-local operating contract for `d:\Projects\Paralls`.

Its primary purpose is to govern the transformation of long-form conversation archives into stable, internally consistent project documents for 《开本 / Paralls》.

This repo is document-first work, not implementation-first work. The first job is not to invent new product ideas. The first job is to correctly recover, compare, and freeze the latest valid decisions from the conversation history.

<operating_principles>
- Treat `filtered_conversation_split/` as the only primary evidence base unless the user explicitly provides newer same-thread evidence.
- When the same topic appears multiple times, later turns override earlier turns if they conflict.
- Do not flatten iterative discussion into a single narrative without preserving version evolution.
- Prefer explicit traceability over polished but untraceable prose.
- Before writing a “final” project document, first update the consolidation layer if the new work changes current truth.
- Old drafts are navigation aids only. They are not authoritative.
- If evidence is incomplete or contradictory, record the uncertainty explicitly instead of silently choosing a convenient version.
- Keep every document narrow in purpose. Do not let roadmap, architecture, gameplay, and business sprawl into one file.
</operating_principles>

## 1. Repository Scope

This `AGENTS.md` governs the entire repository rooted at `d:\Projects\Paralls` and all files beneath it unless a deeper `AGENTS.md` overrides it.

## 2. Project Intent

The active repository mission is:

1. recover stable project truth from `filtered_conversation_split/`
2. compare early and late versions of the same topic
3. freeze current decisions in the consolidation layer
4. produce first-batch core project documents
5. expand only after the source-of-truth chain is stable

This repo does not currently use `PLANNING.md` and `TASK.md` as its main project-navigation layer. For this project, their functional equivalents are the consolidation docs under `docs/consolidation/`.

## 3. Document Roles

### 3.1 Primary Evidence Layer

- `filtered_conversation_split/`
- especially `filtered_conversation_split/turns/by-range/**`
- and `filtered_conversation_split/indexes/**`

This is the canonical source material.

### 3.2 Consolidation Layer

Located under `docs/consolidation/`.

These files are the project’s working source of truth:

- `01-决策总表.md`
- `02-模块来源索引.md`
- `03-冲突与覆盖表.md`
- `04-Phase映射表.md`

Roles:

- `01-决策总表.md` records the current accepted conclusions
- `02-模块来源索引.md` maps each module to the most important source turns
- `03-冲突与覆盖表.md` records early-vs-late version conflicts and adopted versions
- `04-Phase映射表.md` maps capabilities to Phase 0 / 1 / 2 / 3

### 3.3 Core Document Layer

Located under `docs/phase1/core/`.

These are the first-batch formal project documents:

- `产品白皮书.md`
- `技术架构总纲.md`
- `Phase 0 启动方案.md`
- `Phase 1 项目计划书.md`
- `司命设计文档.md`
- `角色智能体设计文档.md`
- `事件总线与感知链路设计.md`
- `核心玩法机制设计.md`
- `剧本创作模板.md`

These docs are outputs derived from the consolidation layer, not direct substitutes for it.

### 3.4 Expansion Layer

Future deeper docs should live under `docs/phase1/expansion/` or another clearly named subtree, not in the repo root.

### 3.5 Old Drafts

The following files are explicitly downgraded to stale-reference status:

- `规则-参与者分类与死亡规则.md`
- `Paralls.md`
- `Phase 1.md`

They may be used only for:

- locating topics
- spotting likely omissions
- checking whether a later rewrite forgot a once-important issue

They must not be used as authoritative source-of-truth documents.

## 4. Source-of-Truth Hierarchy

When documents disagree, use this order:

1. explicit user correction in the current thread
2. latest applicable turn in `filtered_conversation_split/`
3. `docs/consolidation/03-冲突与覆盖表.md`
4. `docs/consolidation/01-决策总表.md`
5. the relevant `docs/phase1/core/*.md`
6. stale root-level drafts for reference only

If `01-决策总表.md` and the latest supporting turns disagree, the turns win and the consolidation docs must be updated immediately.

## 5. Required Methodology

### 5.1 Timeline Comparison Rule

For any meaningful topic, do not jump directly from a source turn to a final paragraph.

You must do all of the following:

1. identify the earlier version
2. identify the later version
3. compare what changed
4. decide which version is now authoritative
5. record that decision in the consolidation layer before or alongside writing the final doc

For `filtered_conversation_split/` work, this rule has an additional mandatory check:

1. when a target conclusion depends on a specific turn document, also inspect the immediately preceding and immediately following turn documents by sequence number whenever they exist
2. if the adjacent turns continue the same topic, expand outward until the topic boundary is clear
3. do not treat the target turn as self-contained until its local before/after context has been checked

Purpose:

- catch user corrections that happen in the next turn
- catch setup assumptions introduced in the previous turn
- avoid freezing an intermediate formulation when the neighboring turns already revise it

If the topic materially evolved, the target core doc should include a short section such as:

- `时间线覆盖说明`
- `关键修订点`
- `当前采用规则`

This is mandatory for:

- roadmap and phase planning
- architecture
- Siming
- character agents
- gameplay/goal systems
- evidence chain
- small world / companion design

### 5.2 Index-First Rule

Before writing or substantially revising a core doc, check whether the consolidation layer is still aligned.

At minimum:

- confirm the relevant rows in `01-决策总表.md`
- confirm the module mapping in `02-模块来源索引.md`
- add or update any conflict in `03-冲突与覆盖表.md`
- update `04-Phase映射表.md` if phase boundaries moved

If a core doc is updated without syncing the consolidation layer where needed, the work is incomplete.

### 5.3 Explicit Adoption Rule

When a topic has multiple versions, do not merely write the chosen answer.

Also state:

- which earlier version was superseded
- which later version is adopted
- why the later version is now binding

This can live in the conflict table, in the relevant core doc, or both.

### 5.4 No Silent Merging

Do not silently merge mutually incompatible versions into one “balanced” paragraph.

If versions differ materially, you must either:

- pick one version and justify it
- or mark the issue as unresolved

### 5.5 No Root-Draft Inheritance

Do not copy prose from the stale root drafts into new docs unless the same claim is verified against the source turns.

If reused, re-verify first and rewrite in the current document’s terminology.

## 6. Writing Rules

### 6.1 Writing Style

- Write in direct, structured, implementation-friendly language.
- Prefer explicit scope and boundary statements over persuasive narrative.
- Separate current frozen decisions from future possibilities.
- Mark speculative future extensions as future-facing, not current truth.

### 6.2 Required Sections For Core Docs

Most first-batch core docs should include:

- status
- main source turns
- supplemental source turns
- current frozen conclusions
- timeline coverage or key revisions when applicable
- main body
- out-of-scope or deferred items

### 6.3 Current vs Deferred

Always distinguish:

- current adopted design
- deferred design
- rejected or superseded design

Do not collapse them into one section.

### 6.4 Phase Discipline

Do not let later-phase capabilities leak into Phase 0 or Phase 1 documents as if they are in-scope.

Use `04-Phase映射表.md` to enforce this.

## 7. Update Rules

Whenever work changes the project’s current truth:

1. update the relevant consolidation doc first or in the same editing pass
2. update the affected core doc
3. ensure the conflict table is synced if the change was driven by early-vs-late divergence

If a working session touches architecture, planning, or gameplay scope and does not update the consolidation layer, the session is incomplete.

## 8. Verification Rules

Before claiming a document update is complete, verify:

1. the document’s main claims can be traced to specific turns
2. later turns were checked for superseding corrections
3. stale root drafts were not treated as source of truth
4. phase boundaries were respected
5. related consolidation docs were updated if the accepted truth changed
6. for each target turn used as a key source, its adjacent before/after turn documents were checked unless the turn is at the beginning or end of the archive

For document work, “verification” means evidence alignment, not code tests.

## 9. Parallel Work Rules

Parallelism is allowed only when write ownership is clearly separated.

Safe parallel slices in this repo usually look like:

- one agent updates a consolidation file
- another agent drafts a different core doc

Unsafe parallelism includes:

- two agents editing the same consolidation file
- two agents independently re-deciding the same topic
- one agent updating a core doc while another changes the underlying accepted truth for the same topic

When in doubt, serialize integration.

## 10. Stop Conditions

Stop and ask the user only when:

- a newer correction seems ambiguous
- two late turns conflict and no clear override exists
- adopting one version would materially change product direction
- the requested document work would erase unresolved contradictions that should remain explicit

Do not stop just because the conversation archive is large.

## 11. Project-Specific Execution Loop

Default loop for this repo:

1. inspect relevant source turns
2. compare early and late versions
3. update `docs/consolidation/`
4. draft or revise the target `docs/phase1/core/` document
5. verify traceability and phase alignment
6. report changed files and remaining unresolved questions

## 12. Minimum Completion Standard

A document task in this repo is only complete if:

- the result reflects the latest valid conversation state
- the version-evolution logic is preserved where needed
- the consolidation layer is kept aligned
- no stale root draft was used as truth

If any of these is missing, the task is not done.
