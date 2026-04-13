---
name: rule-wizard
description: "A clarification wizard that uses option-based Q&A rounds before adding a rule. Captures missing details from context, presents alternatives as options, dynamically detects multiple rules, and finally adds the finalized rule(s) via /rule. 3 scopes: project, --global, --team."
argument-hint: "[--global|--team] <Turkish context -- general topic of the rule>"
---

# /rule-wizard Skill

A wizard that surfaces details easily overlooked when writing a rule directly with `/rule` (edge cases, exceptions, alternative formulations, scope, motivation, example variants) through **option-based Q&A rounds**. Once the discussion is complete, it passes the finalized rule(s) to the `/rule` skill to write them to the file.

> Related skill: [/rule](../rule/skill.md) -- The rule writer invoked at the end of this skill.
> System documentation: [.claude/docs/coding-rules-system.md](../../docs/coding-rules-system.md)

---

## Parameter Requirement

The skill is always invoked with a **context** argument. The context is a short Turkish text describing the general topic of the rule, the initial draft idea, or the problem encountered.

- ✅ `/rule-wizard API'de logging kullanımı`
- ✅ `/rule-wizard Worker doğrudan DB'ye bağlanmasın`
- ✅ `/rule-wizard Controller'larda try-catch yazılmasın, global handler üstlensin`
- ❌ `/rule-wizard` (invoked without context)

**If no context is provided:** Do not proceed with the flow. Ask the user concisely and clearly for context:

> "This skill requires an initial context. Please briefly describe the general topic or initial idea for the rule: `/rule-wizard <topic of the rule>`. Example: `/rule-wizard Worker Jobs'ların hata yönetimi`."

Do not read any files or ask any questions until the user responds.

---

## Phase 1 -- Understanding and Preparation

### 1.1 Read Existing Rules (Mandatory)

**Before** starting the questioning, always read the following files:

- `.claude/rules/coding-common.md`
- **All .md files** in the `.claude/docs/coding-standards/` directory (these may differ per project -- dynamically list and read them)

Purpose:
- **Duplication prevention:** Does the same or a very similar rule already exist?
- **Conflict detection:** Does the new rule contradict an existing one?
- **Extension opportunity:** Can the new rule be added as a bullet point within an existing rule?
- **Cross-reference:** Rules that can be referenced in the `Related` field at the final step.

### 1.2 Analyze the Context

Extract the following from the context provided by the user (silently, without showing in the response):

- **Probable scope:** Which application(s)? Common or a single application?
- **Probable intent:** Mandatory (must) / prohibitive (must not) / advisory (should)?
- **Affected layers:** Controller, Service, Repository, Consumer, Hub, Job?
- **Initial hypotheses:** Raw ideas for Apply when, Why, Examples.
- **Similar existing rules:** Note the IDs if any were found in 1.1.

### 1.3 Present the Analysis Summary

Summarize your understanding to the user in a **short paragraph**. Example:

> "As I understand it, you want API controllers not to contain try-catch blocks, and error handling to be delegated to a global handler at the upper layer. This falls under the scope of `coding-standards/api.md` and complements the existing `no-logic-in-bridges` rule. I'll now clarify the details with a few questions."

Then proceed to Phase 2.

---

## Phase 2 -- Option-Based Questioning

### Core Principles (All Binding)

1. **Every question is asked using the `AskUserQuestion` tool.** No plain text, open-ended questions. Questions like "Can you explain this?" are forbidden -- always generate options.
2. **Each question has 2-4 options.** Platform limit is 4. If there are more reasonable options, split the question; never limit yourself to "the best 4."
3. **An "Other" option is automatically added.** The user can enter free text; do not explicitly write this as an option -- the tool adds it itself.
4. **If there is a recommended option, place it first** and add `(Recommended)` to its label. Briefly explain why you recommend it in the option's `description` field.
5. **Ask in Turkish, present options in Turkish.** `/rule` will translate to English at the final step -- this skill always communicates in Turkish and produces Turkish output.
6. **Maximum 4 questions per round.** If more than 4 questions are needed, split into rounds -- answers from the first round can feed into the second.
7. **Do not re-ask about fields that are clearly derived from context.** Instead, ask a confirmation question: "I understood it as X -- is that correct?" (binary option: Correct / No, I want to change it).
8. **Options must be distinct and clear.** If two options are nearly the same, remove one. Options should be collectively exhaustive -- if an "all of the above" situation exists, present it with `multiSelect: true`.

### Areas to Cover

Clarify the following areas for each rule through Q&A. Convert fields that are clearly derived from context into confirmation questions; convert unclear ones into full questions.

#### A) Scope -- Where should it be written?

**If `--global` or `--team` flag is provided,** skip this question; the scope is already determined. If no flag is provided, ask:

**Example question:** "Where should this rule be written?"

**Option patterns:**
- Specific to this project (`.claude/`) -- default
- Applies to all my projects (`--global` -> `~/.claude/rules/`)
- Team knowledge base (`--team` -> agent or team rule file)

**Follow-up question if project scope is selected:** "Which application does it cover?"
- Dynamically list existing files in `.claude/docs/coding-standards/`
- All applications (common) -- `coding-common.md`
- One option for each existing `coding-standards/{app}.md` file

**Follow-up question if team scope is selected:** "Which agent's knowledge base should it be added to?"
- List agent files in the installed team
- Or add as a team-wide rule

#### B) Single-Sentence Rule Statement (Rule)

**Example question:** "Which best expresses the essence of the rule?"

**Option patterns:** Derive 3 alternative formulations from context. Each offers a different tone/restrictiveness:
- **Strict prohibition** version ("X must never be done")
- **Advisory** version ("Use Y for X")
- **Conditional** version ("X may only be done when Y")

The user can write their own sentence via "Other."

#### C) Motivation (Why)

**Example question:** "What is the primary motivation for this rule?"

**Option patterns (select based on context):**
- Lesson from a past mistake (specify which)
- Architectural consistency (single source of truth, single entry point, etc.)
- Testability
- Performance
- Security
- Readability / maintainability
- Regulation / compliance

Use `multiSelect: true` if multiple motivations are possible. Place the primary motivation first.

#### D) Apply When (Trigger Conditions)

**Example question:** "In which file/code patterns should this rule be triggered?"

**Option patterns:** Derive 2-4 specific triggers from context. Each option should contain a **concrete file path or code pattern**:

- `api/Controllers/*.cs` -- in controller actions
- `api/Services/*.cs` -- in service methods
- `api/Consumers/*.cs` -- in consumer handlers
- Specific attribute/pattern (e.g., `[HttpPost]`, `BackgroundService` derivatives)

Use `multiSelect: true` if multiple triggers can be selected together.

#### E) Don't Apply When (Exceptions) -- Optional

**Example question:** "Are there situations where this rule does not apply?"

**Option patterns:**
- No, no exceptions
- Yes: test code is an exception
- Yes: legacy/generated code is an exception
- Other: user specifies their own

If the user selects "No," the `Don't apply when` field is not written in the final text.

#### F) Examples

**Goal:** At least one ✅ correct and one ❌ wrong concrete example.

**Example question:** "Which ✅ correct example best represents the rule?"

**Option patterns:** Derive 2-3 short code snippets or scenarios. Each illustrates the rule from a different angle. The user selects one or writes their own via "Other." The same logic applies to the ❌ example, asked as a separate question.

#### G) Related Rules -- Optional

**Example question:** "Should this rule reference an existing rule?"

**Option patterns:** List IDs of similar rules found during Phase 1.1 + a "None" option. Example:

- `no-logic-in-bridges` (related -- gains meaning together)
- `repository-owns-db-access` (distantly related)
- None, independent rule

### Suggested Round Planning

- **Round 1 (fundamentals):** Scope + Rule statement + Motivation -- 3 questions
- **Round 2 (behavior):** Apply when + Exception + Example (✅) -- 3 questions
- **Round 3 (polish):** Example (❌) + Related + (if needed) edge case -- 2-3 questions

Rounds shrink when fields are clearly derived from context; additional questions are added when there is ambiguity. The user should never be forced to give the same answer twice within a round.

---

## Phase 3 -- Dynamic Multiple Rule Detection

The skill **starts with a single-rule assumption**. However, if any of the following signals are observed during questioning, **immediately** ask the user a distinction question and adjust the flow accordingly.

### Detection Signals

1. **Two different applications are selected in the Scope question and their natures differ** (e.g., both API and Worker, but the rule has different meanings in API controllers vs. Worker BackgroundServices).
2. **The triggers selected in Apply when point to two unrelated code layers** (e.g., `api/Controllers/` + `api/Repositories/`).
3. **The Rule statement combines two independent prohibitions** (in the form "X must not be done and Y must not be done either" -- two independent clauses).
4. **The scenarios derived in the Examples question cannot be explained by a single rule** -- each scenario exemplifies a different principle.
5. **Two independent justifications are selected in Motivation multiSelect** (e.g., performance + security, but each deserves to be a separate rule).

### Distinction Question

When any signal is triggered, ask the following using AskUserQuestion:

**Question:** "This context actually looks like two different rules. How should we proceed?"

**Options:**
- **(Recommended)** Add as two separate rules -- we will clarify each one separately
- Keep as a single rule -- we will expand the Rule statement to cover both clauses
- Let's focus on only one for now and handle the other later
- Misidentified -- this is actually a single rule

### Post-Decision Flow

- **Two separate rules:** Repeat Phase 2 independently for each rule. Finalize the first rule first, then move to the second. Do not mix question rounds -- each rule has its own answer set.
- **Keep as a single rule:** Re-ask the Rule statement question and present formulations that combine both clauses.
- **Focus on only one:** Do not forget the other; at the end of Phase 4, offer the user the opportunity to start a second round with "Shall we handle the other rule now?"
- **Misidentification:** Return to normal flow, ignore the signal.

---

## Phase 4 -- Consolidation and Final Approval

### 4.1 Generate the Turkish Rule Text

Once questioning is complete, compose a **Turkish natural language** rule text from the collected answers that will serve as the **input for the /rule skill**.

This text should contain all the information that the `/rule` skill will parse:
- **Scope** (it should be clear which file it will go to)
- **Rule** (clear single-sentence statement)
- **Why** (motivation)
- **Apply when** (specific conditions)
- **Don't apply when** (if applicable)
- **Examples** (✅ and ❌)
- **Related** (if applicable)

**Example final text:**

> "API projesindeki controller aksiyonlarında try-catch bloğu yazılmasın -- hata yönetimi üst katmandaki global exception handler'a bırakılsın. Bu kural mimari tutarlılığı korumak ve controller'ları gerçekten ince köprüler olarak tutmak için var; try-catch servis veya global handler'ın sorumluluğudur. Uygulanacağı yerler: `api/Controllers/` altındaki tüm `.cs` dosyalarındaki controller aksiyonları. Test kodu istisna sayılır. Doğru örnek: `[HttpPost] public async Task<IActionResult> Create(CreateProductRequest req) { var result = await _productService.CreateAsync(req); return Ok(result); }` -- hiçbir try-catch yok. Yanlış örnek: controller içinde `try { ... } catch (Exception ex) { return BadRequest(ex.Message); }` yazmak. İlgili kural: `no-logic-in-bridges`."

This text can be a single paragraph or split into two-three sentences if needed -- but it is not converted to compressed `Rule:` format; the `/rule` skill will do its own parsing.

### 4.2 Show to User and Get Approval

Show the generated Turkish text to the user and ask the following approval question via AskUserQuestion:

**Question:** "Is this text the final version of the rule? Can I add it now with `/rule`?"

**Options:**
- **(Recommended)** Yes, add with `/rule`
- I need to correct a part of the text -- let me tell you which part
- I think there is a missing area -- let's do an additional question round
- Cancel, do not add for now

### 4.3 Invoking the /rule Skill

When the user selects "Yes, add":

- **If there is a single rule:** Invoke `/rule <Turkish final text>`.
- **If there are multiple rules:** Invoke each one **sequentially**. Give the user a brief progress notification between them:
  - "First rule written (`{id}` -- `{file}`). Moving to the second rule now."
- **After each `/rule` invocation**, relay the result to the user as a summary.

**If correction is selected:** The user states which part they want to correct. Re-ask the question corresponding to that part (or a closely related question) via AskUserQuestion, get the answer, update the final text, and repeat 4.2.

**If additional round is selected:** Run a new question round for the area the user identified as missing, then return to 4.1.

**If cancel is selected:** Terminate the skill cleanly. Do not write to any file. Tell the user: "Rule was not written. You can start again with `/rule-wizard` whenever you want."

### 4.4 Final Summary

When the writing phase is complete, give the user a single summary message:

- How many rules were written
- Each rule's **ID** and which **file** it was written to
- Existing rules marked as **related**, if any
- Remind the user if there is a deferred rule from Phase 3 that was set aside for later

---

## Critical Principles (Summary)

1. **Context is mandatory.** The skill does not work without an argument -- it asks the user for context.
2. **Every question has options.** AskUserQuestion is used; plain text questions are never asked.
3. **If 4 options are not enough, split.** If there are 5+ reasonable options, split the question into two rounds. Never limit yourself to "the best 4."
4. **Read existing rules first.** A mandatory prerequisite to catch duplication and conflicts early.
5. **Never assume.** Every field not clearly derived from context requires a question. Even if it is a confirmation question, it must be asked.
6. **Dynamically detect multiple rules.** Start with a single-rule assumption but ask the user when a divergence signal appears.
7. **Final text is in Turkish.** The `/rule` skill will translate to English -- this skill always communicates and produces output in Turkish.
8. **`/rule` is not invoked without approval.** The finalized text is shown to the user and approval is obtained before proceeding to the writing phase.
9. **An incomplete field is worse than a nonexistent field.** The required fields of the rule format (Rule, Why, Apply when, Examples) must be fully represented in the final text.
10. **The skill can be run repeatedly for multiple rules.** If dynamic split mode was selected in Phase 3, each rule goes through the Phase 2-4 cycle individually.
