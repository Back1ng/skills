---
name: "speckit-plan-review"
description: "Walk the owner through every decision the model made on its own in finished planning artifacts — a spec-kit feature (spec, research, data model, contracts, plan) or an OpenSpec change (proposal, design, spec deltas) — re-investigate each one against the codebase, offer fresh options, and write the owner's answers back into the artifacts. For spec-kit, run after the plan command and before the tasks command."
argument-hint: "Optional: `speckit` or `openspec`, a feature directory or OpenSpec change name, and a focus (artifact, topic, decision ids)"
compatibility: "Requires a spec-kit project (.specify/ directory) or an OpenSpec project (openspec/ directory)"
metadata:
  author: "project"
user-invocable: true
disable-model-invocation: false
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty). It may name the format (`speckit` or `openspec`), point at a feature directory or an OpenSpec change, and narrow the review to one artifact, one topic, or specific decision ids from an earlier session.

## Goal

Planning commands (the spec-kit plan command, an OpenSpec proposal) resolve every open technical question by themselves and never ask the owner. This command is the missing checkpoint: it surfaces every decision the model took on its own, lets the owner confirm or change each one, and leaves the owner with a complete picture of what is about to be built.

The owner will not read the artifacts in full. After this session they must know every decision that shapes the result, and every one of those decisions must be either confirmed by them or changed.

Rules that hold for the whole session:

- Talk to the owner in the language the spec artifacts are written in.
- There is no limit on the number of questions. Do not drop a decision to keep the session short. The owner may stop at any time.
- You are reviewing the plan, not defending it. The plan's own rationale is a claim to check, not a fact.
- Do not write code and do not change anything outside the reviewed directory. Do not create `tasks.md`; step 5 says when an existing one may be edited.

## Execution Steps

### 1. Pick the format and locate the artifacts

Decide which format is under review, in this order:

1. The user input names it: the word `speckit` or `openspec`, a path to a directory, or an OpenSpec change name.
2. Otherwise detect it: `.specify/` exists → spec-kit; `openspec/changes/` exists → OpenSpec.
3. If both exist and the input does not say, ask the owner which one. Do not guess.

The rest of this document uses three names. Resolve them here:

| Name | spec-kit | OpenSpec |
|---|---|---|
| **Review directory** | the feature directory | `openspec/changes/<change>/` |
| **Decision log** | `research.md` | `design.md` |
| **Behaviour spec** | `spec.md` | `proposal.md` and the deltas under `specs/` |

**spec-kit.** Run `.specify/scripts/bash/check-prerequisites.sh --json --paths-only` from the repo root once and parse `FEATURE_DIR`, `FEATURE_SPEC`, `IMPL_PLAN`, `TASKS`. If the input gives a feature directory, use that directory instead.

- `spec.md`, `plan.md` and `research.md` must exist. If any is missing, stop and tell the owner which command to run first.
- Read every file in the directory except `tasks.md`: `spec.md`, `plan.md`, `research.md`, `data-model.md`, `quickstart.md`, everything under `contracts/`.
- Read `.specify/memory/constitution.md` if it exists.

**OpenSpec.** The change is the one named in the input. If none is named, list the directories in `openspec/changes/` except `archive/`: with exactly one, use it; with several, ask the owner which.

- `proposal.md` must exist. If `design.md` is missing, say so and create it when the first answer is recorded.
- Read `proposal.md`, `design.md`, and every delta under `specs/`. Read `tasks.md` only to learn what is planned; it is not a source of decisions.
- Read the project context the change is written against: `openspec/config.yaml`, `openspec/project.md` and other documents at the top of `openspec/` if present, and the current `openspec/specs/<capability>/spec.md` for every capability the change touches. These play the role of the constitution.

**Both formats.**

- If `tasks.md` already exists, say so once: decisions changed here affect it; step 5 says how. Then continue.
- If the decision log already has an `## Owner Decisions` section, this is a resumed review. Treat decisions recorded there as settled and skip them, unless the user input asks to revisit them.

### 2. Build the decision inventory

List every decision in the artifacts that the owner did not make. Go through all of these sources:

| Source | What counts as a decision |
|---|---|
| `spec.md` Clarifications | Any answer the owner did not give. An answer is the owner's only if the owner typed it in a conversation; a question the model asked and answered itself is a model decision. |
| `spec.md` Assumptions, Out of Scope | Every assumption, every default, every item cut from or added to the scope. |
| `spec.md` Requirements | Behaviour that goes beyond the source request (roadmap, issue, user description): extra states, extra actions, extra screens, numeric limits the source does not give. |
| `research.md` | Every `Decision` entry, and every alternative it rejected. |
| `data-model.md` | Tables and columns, what a record is tied to, what happens on delete of each related record, statuses and transitions, what the database guarantees versus what only code guarantees, snapshots versus live values. |
| `contracts/` | Routes and their shape, error behaviour, limits, who is allowed to do what, what is returned on a repeat or conflicting request. |
| `plan.md` | Complexity Tracking, new files and layers, new dependencies, kinds of tests, performance and storage choices, known limitations the plan accepts. |
| `quickstart.md` | Manual steps that replace an automated check. |

For an OpenSpec change the same kinds of decision live in different files:

| Source | What counts as a decision |
|---|---|
| `proposal.md` | What is in and out of the change, anything beyond the original request, the stated impact, every assumption and default. |
| `design.md` | Every decision and the alternatives it rejected, data and schema choices, delete behaviour, concurrency, storage, risks and trade-offs the design accepts, open questions the model closed by itself. |
| `specs/` deltas | Every added, modified or removed requirement and scenario that the request did not ask for; numeric limits; error and repeat-request behaviour; who is allowed to do what. |
| `tasks.md` | Only as evidence: kinds of tests, new dependencies or layers that appear in tasks but are not justified in `design.md`. |

Provenance check. A decision counts as the owner's only if you can point at the owner's own words: a real answer in a conversation, committed text in the source document, or the constitution (for OpenSpec: the project context files). Text that the same run wrote into the roadmap or another source file is not the owner's word; use `git diff` and `git log` on such files when a decision cites them.

For each decision record: a short id (`D-01`, `D-02`, …), one plain sentence stating what was decided, where it is written (file and section), and its group:

1. Product behaviour and scope — what the user of the product will see or be able to do.
2. Data — schema, relations, delete behaviour, statuses.
3. Privacy, security and access.
4. Concurrency and failure handling.
5. Files, performance and storage.
6. Interface structure.
7. Tests and acceptance.

Merge duplicates: the same decision restated in four files is one entry with four locations.

### 3. Show the full map first

Before the first question, show the owner the whole inventory as a compact list grouped as above: id, one sentence, and a mark.

- **Discuss** — will be asked one at a time. This is the default.
- **Routine** — no real alternative exists and the choice simply follows an existing project convention you verified in the code (for example a file location or a naming pattern). List these so the owner sees them, but do not ask about each.

A decision is never Routine if any of these is true: the chosen option is more complex than the simplest one that works; it is hard to undo later (schema, foreign keys, where files live, public URLs); it changes behaviour the product's user sees; it touches personal data, access, or money; the plan cites a reason you have not verified.

Tell the owner how many decisions there are in each mark, and that they can pull any Routine item into discussion by its id or stop at any time. Then start the questions without waiting for approval, unless the owner replies first.

### 4. Ask, one decision at a time

Order: groups 1 to 7 as listed, and inside a group the hardest-to-undo first. If an answer makes a later decision moot or changes its options, drop or rebuild that later question and say so.

#### 4a. Re-investigate before asking

Do this for each decision immediately before asking about it. Do not reuse the reasoning already written in the decision log.

- Open the code the decision touches. Find how the project already solves the same kind of problem, and whether an existing helper, component, pattern or test setup already covers it.
- Check every factual claim in the plan's rationale: that a file, class, method, column or package exists and behaves as described; that a stated limitation is real; that a rejected alternative is really impossible. Record anything false.
- Work out the options again from scratch. The list must contain:
  - the **simplest option that works**, even if the plan never considered it;
  - the option the plan chose;
  - any other realistic option you found in the code or know from practice, including ones absent from `Alternatives considered`.
- For each option, work out what it costs: new files and concepts, what breaks or degrades and in which concrete scenario, how hard it is to change later.
- Check the option against the constitution or the OpenSpec project context.

If the investigation shows the plan's choice rests on a false premise, or that it conflicts with another artifact, say that at the top of the question.

#### 4b. Question format

Present exactly one decision per message, in this shape:

```text
**D-07 · <group>** — <n> of <total>

**Decided:** <what the plan decided, one or two plain sentences> (<file> → <section>)
**Why it matters:** <the concrete consequence for the product or for maintenance>
**What I found:** <facts from the code and from checking the plan's reasoning; note false claims>

| | Option | What you get | What it costs |
|---|---|---|---|
| A | <simplest that works> | … | … |
| B | <plan's choice> | … | … |
| C | <other> | … | … |

**In the plan now:** B   **Simplest:** A   **I recommend:** <letter> — <one sentence why>
```

- Mark the same option twice or three times when they coincide.
- Recommend what you would choose after the fresh investigation. It may differ from the plan. Prefer the simplest option unless you can name the scenario in which it fails.
- Use plain wording. Define any term the owner needs in the same sentence.
- Keep cells short. Put a longer explanation in a sentence under the table.
- If the agent has a structured question tool, you may use it for the choice, but the facts and the table must be shown in the message first.

Accepted replies: an option letter; a word of agreement to keep what is in the plan; their own answer in free text; a request to explain further or to look deeper (investigate more, then ask the same decision again); a request to defer; a request to stop.

Never reveal the options of later questions in advance, and never batch several Discuss decisions into one message.

### 5. Write each answer back immediately

After every answer, before the next question:

1. In the decision log, make sure a section `## Owner Decisions` exists at the end, with a subsection `### Session YYYY-MM-DD` for today. Append one line:
   `- D-07: <decision in one sentence> → <final choice>. <Confirmed | Changed from: previous choice | Deferred>.`
2. **Confirmed** — change nothing else.
3. **Changed** — update every place the decision lives so that no artifact still describes the old choice.
   - spec-kit: the `Decision` / `Rationale` / `Alternatives considered` entry in `research.md`, then `data-model.md`, `contracts/`, `plan.md` (structure, constraints, Complexity Tracking, Constitution Check), `quickstart.md`. If the decision changes what the product's user sees or can do, also update `spec.md`: the affected requirement, scenario, assumption or scope item, plus a `- Q: … → A: …` line under `## Clarifications` → `### Session YYYY-MM-DD`.
   - OpenSpec: the decision entry in `design.md`; `proposal.md` if scope or impact changes; the requirement and its scenarios in the affected `specs/` delta, keeping the delta format (`## ADDED|MODIFIED|REMOVED Requirements`, `### Requirement:`, `#### Scenario:`).
4. Replace the old text; do not leave the old and new statements side by side. Then search the review directory for other mentions of the old choice and fix them.
5. **Deferred** — record it and leave the artifacts as they are.
6. Save after each decision, so an interrupted session loses nothing.

`tasks.md`, when it exists:

- spec-kit: do not edit it. It is regenerated by the tasks command; say so in the report.
- OpenSpec: tasks are written together with the proposal and there is no separate step that regenerates them. After a changed decision, update only the tasks that decision affects — reword, remove, or add — and keep the numbering and checkbox format. Do not touch tasks already checked off without telling the owner.

Do not reorder unrelated sections or reformat text you are not changing.

### 6. Close the session

When every Discuss decision is answered, or the owner stops:

- Ask once whether any Routine item should be discussed. If not, record them in `## Owner Decisions` as a single line: `- Routine, accepted without discussion: D-03, D-11, …`.
- Re-read the changed artifacts once and confirm they do not contradict each other: names, routes, limits, statuses, counts.
- spec-kit: if `spec.md` changed and `checklists/requirements.md` exists in the feature directory, re-evaluate its checkboxes against the updated spec and toggle only the ones whose state changed.
- OpenSpec: if the `openspec` CLI is available, run `openspec validate <change> --strict` and fix any format errors your edits introduced.

## Completion Report

Report, in the owner's language:

- How many decisions were found, confirmed, changed, deferred, and left unasked because the owner stopped.
- A table of every decision: id, one sentence, final choice, status.
- For each changed decision, the files that were edited.
- What the changes did to the size of the solution: mechanisms, files or kinds of tests removed or added.
- Facts in the plan that turned out to be false, even where the decision itself stood.
- Deferred and unasked decisions, with what will be built if nothing changes.
- The next step. spec-kit: the tasks command; if `tasks.md` existed before the session and anything changed, say it must be regenerated. OpenSpec: which tasks were edited, and that the change is ready to apply.

## Done When

- [ ] Every model-made decision in the artifacts is in the inventory and the owner has seen the full map
- [ ] Each discussed decision was re-investigated against the code before it was asked
- [ ] Each answer is recorded under `## Owner Decisions` in the decision log
- [ ] Every changed decision is updated in all artifacts, with no leftover text describing the old choice
- [ ] Completion report delivered
