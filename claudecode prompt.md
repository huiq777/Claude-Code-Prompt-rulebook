Prompt Word Choice and Length — The Full Rulebook

This is the part most tutorials skip entirely. The *words* you choose and *how long* you write matter as much as the content itself.

---

### Signal Word Hierarchy: The Escalation Ladder

Claude Code uses a deliberate hierarchy of signal words. Each sits at a different "urgency level" — and the model is sensitive to the difference. Use the wrong level and either the rule gets ignored or everything sounds like an emergency (so nothing does).

Here is the actual ladder, derived from real usage across the codebase:

```
STRICTLY PROHIBITED  ←  absolute hard stop, no exceptions
=== CRITICAL ===     ←  violation causes real damage or data loss
CRITICAL:            ←  must not be broken, failure mode explained
IMPORTANT:           ←  easily missed but consequential
NEVER                ←  categorical prohibition
ALWAYS               ←  categorical requirement
DO NOT               ←  direct prohibition, narrower than NEVER
Note: / NOTE:        ←  contextual awareness, no urgency
```

**Real examples from Claude Code:**

| Signal word | Example | Source |
|---|---|---|
| `STRICTLY PROHIBITED` | "You are STRICTLY PROHIBITED from: Creating new files..." | [verificationAgent.ts:15](src/tools/AgentTool/built-in/verificationAgent.ts#L15), [exploreAgent.ts:27](src/tools/AgentTool/built-in/exploreAgent.ts#L27), [planAgent.ts:24](src/tools/AgentTool/built-in/planAgent.ts#L24) |
| `=== CRITICAL ===` (banner) | "=== CRITICAL: DO NOT MODIFY THE PROJECT ===" | [verificationAgent.ts:14](src/tools/AgentTool/built-in/verificationAgent.ts#L14) |
| `CRITICAL:` (inline) | "CRITICAL: Always create NEW commits rather than amending..." | [BashTool/prompt.ts:92](src/tools/BashTool/prompt.ts#L92) |
| `BLOCKING REQUIREMENT` | "this is a BLOCKING REQUIREMENT: invoke the relevant Skill tool BEFORE generating any other response" | [SkillTool/prompt.ts:190](src/tools/SkillTool/prompt.ts#L190) |
| `CRITICAL REQUIREMENT` | "CRITICAL REQUIREMENT - You MUST follow this: After answering... include a 'Sources:' section" | [WebSearchTool/prompt.ts:14](src/tools/WebSearchTool/prompt.ts#L14) |
| `IMPORTANT:` | "IMPORTANT: Go straight to the point. Try the simplest approach first." | [prompts.ts:418](src/constants/prompts.ts#L418) |
| `NEVER` | "NEVER skip hooks (--no-verify, --no-gpg-sign...)" | [BashTool/prompt.ts:90](src/tools/BashTool/prompt.ts#L90) |
| `DO NOT` | "DO NOT push to the remote repository unless the user explicitly asks" | [BashTool/prompt.ts:115](src/tools/BashTool/prompt.ts#L115) |

**The rules for choosing which level:**

- **STRICTLY PROHIBITED / === CRITICAL === (banner form):** Reserved for actions that are *impossible to undo* or *cross a security/data-loss boundary*. Used sparingly — once or twice per agent at most. If everything is a banner, nothing is.
- **CRITICAL: (inline form):** For rules where *violating them destroys something specific* — and you must explain *exactly what gets destroyed*. Always followed by a reason.
- **IMPORTANT:** For rules that are *easily forgotten* but where violation causes confusion or inefficiency, not catastrophe.
- **NEVER / ALWAYS:** Categorical. Use only when there are *genuinely zero exceptions*. If you write NEVER but there's one valid exception, the model will find it and distrust the rest of your NEVERs.
- **DO NOT:** Narrower than NEVER. Specific action, specific context. "DO NOT use -uall flag" — not a category, just one exact thing to avoid.
- **Note:** / (plain prose): For context and awareness. No urgency. Use for things the model should *know* but doesn't need to act differently on.

**Take: Every signal word is a resource. Spend them only where the stakes match. If you use CRITICAL for something minor, the model discounts all your CRITICALs.**

---

### When to Use ALL CAPS vs Sentence case

Claude Code mixes both deliberately:

- **ALL CAPS for the signal word itself:** `CRITICAL`, `NEVER`, `IMPORTANT`, `STRICTLY PROHIBITED` — the word itself is always capped
- **ALL CAPS inside the rule for the key noun/verb:** "Always create NEW commits" — NEW is capped because it's the *contrast* (new vs. amended)
- **Never ALL CAPS full sentences** — that's shouting, and it dilutes the emphasis

Examples of good contrast emphasis:
```
CRITICAL: Always create NEW commits rather than amending    ← NEW is the key contrast
NEVER commit changes unless the user explicitly asks        ← NEVER caps the signal word only
DO NOT push to the remote repository                        ← DO NOT is the signal, rest is sentence case
```

**Take: ALL CAPS entire sentences = noise. ALL CAPS signal words + key contrasting terms = signal.**

---

### Prompt Length: The Three-Zone Model

Claude Code's prompts are not all the same length. They follow a clear pattern based on what the section is for:

**Zone 1 — Identity & Role: 1–2 sentences. No more.**

From [src/constants/system.ts](src/constants/system.ts):
```
"You are Claude Code, Anthropic's official CLI for Claude."
```

From [src/tools/AgentTool/built-in/verificationAgent.ts:10](src/tools/AgentTool/built-in/verificationAgent.ts#L10):
```
"You are a verification specialist. Your job is not to confirm the
implementation works — it's to try to break it."
```

From [src/tools/AgentTool/built-in/exploreAgent.ts:24](src/tools/AgentTool/built-in/exploreAgent.ts#L24):
```
"You are a file search specialist for Claude Code... You excel at
thoroughly navigating and exploring codebases."
```

One sentence = role. One sentence = stance or goal. Done. Identity sentences should be *precise*, not long. The model doesn't need a paragraph to understand who it is — it needs the right few words.

---

**Zone 2 — Behavioral Rules: 1 bullet per rule. Reason fits in the same bullet.**

From [src/tools/BashTool/prompt.ts:88-94](src/tools/BashTool/prompt.ts#L88-L94):
```
- NEVER update the git config
- NEVER run destructive git commands (...) unless the user explicitly requests these actions.
  Taking unauthorized destructive actions is unhelpful and can result in lost work, so it's
  best to ONLY run these commands when given direct instructions
- NEVER skip hooks (--no-verify, --no-gpg-sign, etc) unless the user explicitly requests it
- CRITICAL: Always create NEW commits rather than amending...
```

Each bullet = one rule + one reason. The reason lives on the same line or the next line — never separated into a different paragraph. If the reason is far from the rule, the model may learn the rule but forget the why.

**Length guide for behavioral rules:**
- Simple prohibition: 1 line. `"NEVER update the git config"`
- Prohibition + reason: 2–3 lines max. `"NEVER run destructive git commands... Taking unauthorized destructive actions is unhelpful and can result in lost work"`
- Rule with failure-mode explanation: 3–5 lines. The CRITICAL commit-amend rule is 3 lines and that's about right.

---

**Zone 3 — Complex Agent System Prompts: Long is fine — but every line must earn its place.**

The Verification Agent system prompt ([src/tools/AgentTool/built-in/verificationAgent.ts:10-129](src/tools/AgentTool/built-in/verificationAgent.ts#L10-L129)) is ~120 lines. This is intentional. A verification agent operating autonomously faces many edge cases: what counts as a PASS, when to stop, what rationalizations to resist, how to handle missing tools. Short prompts leave those gaps — and the model fills gaps with its own defaults, which may not match what you want.

The rule: **prompt length should match task complexity, not your comfort level.** A simple fetch-and-summarize agent needs 5 lines. A verification agent making pass/fail judgments with real consequences needs 120.

What separates long-but-good from long-but-bloated:

| Long-but-good (Claude Code style) | Long-but-bloated (avoid) |
|---|---|
| Each section has a header (`=== REQUIRED STEPS ===`) | Wall of text with no navigation |
| Anti-examples are shown, not just described | Only describes the right behavior |
| Failure modes are named (`verification avoidance`) | Failure modes are implied |
| Output format is shown verbatim | Output format is described loosely |
| Rationalizations are listed and rebutted | Rationalizations aren't anticipated |

---

### Format: When to Use What

Claude Code uses four format types in prompts, each for a specific job:

**1. Prose paragraphs** — for behavioral principles that require nuance and can't be reduced to a list. From [src/constants/prompts.ts:258](src/constants/prompts.ts#L258) (`getActionsSection`):
> "Carefully consider the reversibility and blast radius of actions. Generally you can freely take local, reversible actions like editing files or running tests. But for actions that are hard to reverse..."

Principles with trade-offs live in prose. A bullet list would strip the reasoning.

**2. Bullet lists** — for categorical rules, tool lists, and step-by-step processes where order and discreteness matter. From [src/tools/BashTool/prompt.ts:88-119](src/tools/BashTool/prompt.ts#L88-L119) (git safety protocol).

Bullets work when each item is genuinely independent. Don't force connected reasoning into bullets — the connection gets lost.

**3. Section headers (`#` or `===...===`)** — to create scannable structure in long prompts. Claude Code uses two levels:
- Markdown `#` for main system prompt sections: `# System`, `# Doing tasks`, `# Executing actions with care`
- `===` banners for critical blocks inside agent prompts: `=== CRITICAL: READ-ONLY MODE ===`, `=== REQUIRED STEPS ===`, `=== BEFORE ISSUING PASS ===`

The `===` banner style is used when you *need the model to treat this block differently* — not just read it, but stop and notice it.

**4. Code blocks / verbatim output examples** — for any format the model must reproduce exactly. The Verification Agent shows an exact block of what a `PASS` check looks like, formatted in triple backticks. This is more reliable than describing the format in prose, because the model learns format by example, not by description.

---

### Output Length Instructions: Tell the Model Explicitly

Claude Code includes an entire section dedicated to this — [src/constants/prompts.ts:416-427](src/constants/prompts.ts#L416-L427):

> "Keep your text output brief and direct. Lead with the answer or action, not the reasoning. Skip filler words, preamble, and unnecessary transitions. Do not restate what the user said — just do it."
> "If you can say it in one sentence, don't use three."

And the internal (Ant-user) version is *longer* and more nuanced ([src/constants/prompts.ts:405-414](src/constants/prompts.ts#L405-L414)):

> "Match responses to the task: a simple question gets a direct answer in prose, not headers and numbered sections."
> "Use inverted pyramid when appropriate (leading with the action)"

This tells you something important: **output length instructions are themselves a part of the prompt**. You can tell a model to be brief. You can tell it to lead with conclusions. You can tell it to match response length to question complexity. These instructions work — but only if you actually write them.

**The Explore Agent also has a meta-instruction about its own speed** ([src/tools/AgentTool/built-in/exploreAgent.ts:52-55](src/tools/AgentTool/built-in/exploreAgent.ts#L52-L55)):
> "You are meant to be a fast agent that returns output as quickly as possible... Make efficient use of the tools... Wherever possible you should try to spawn multiple parallel tool calls."

This is telling the model *how to think about its own process* — not just what output to produce. You can shape the model's internal strategy with prompt instructions, not just its final output.

---

### The One-Sentence-Per-Rule Test

Before you ship any prompt rule, apply this test:

1. Can you say the rule in one sentence?
2. Can you add the failure mode in one more sentence?
3. Is there a signal word at the front that matches the stakes?

If yes to all three: the rule is ready.

If not: either you don't understand the rule well enough yet, or the rule is actually multiple rules masquerading as one.

---

When Claude Code needs to do complex research, it spawns **sub-agents** — separate Claude instances with their own focused prompts. The Explore agent, Plan agent, Verification agent — each has a completely different system prompt.

The priority hierarchy ([src/utils/systemPrompt.ts:41-123](src/utils/systemPrompt.ts#L41-L123)):

```
Override prompt (automation)
    ↓
Coordinator prompt (orchestration mode)
    ↓
Agent definition (specific role)
    ↓
Custom (--system-prompt flag)
    ↓
Default Claude Code prompt
    +
Append prompt (always added at end)
```

Each level can fully replace the one below it. An enterprise admin can deploy a managed `/etc/claude-code/CLAUDE.md` that overrides all user-level rules. A coordinator agent can give sub-agents entirely custom identities.

**This is governance architecture expressed as prompt logic.**

---

## Part 5: The 10 Actionable Suggestions

### For Using Claude Code

**1. Write a CLAUDE.md and put it in your project root.**
This file gets injected into every conversation. Put your project's conventions, architecture decisions, and "things Claude should never do in this codebase" there. It's the single highest-leverage thing you can do. Don't leave it empty.

**2. Use `.claude/rules/*.md` for modular rule sets.**
Break your CLAUDE.md into focused files. `rules/git.md`, `rules/testing.md`, `rules/style.md`. They all get loaded. Modular rules are easier to maintain and update.

**3. Be specific in your task description — file paths + line numbers.**
"Fix the auth bug" is noise. "Fix the JWT expiry handling in `src/auth/middleware.ts:147` — it's not checking the `exp` claim" is a prompt. The more precise your input, the more context Claude has, the better the output.

**4. Use `/clear` when you switch tasks.**
The context window accumulates all previous conversation. When you finish one task and start another, that history is now noise. `/clear` resets it. Fresh context = better focus.

**5. Use CLAUDE.local.md for things you don't want in version control.**
Project CLAUDE.md gets committed and shared with the team. CLAUDE.local.md is gitignored by default. Put personal preferences and private notes there.

---

### For Building Your Own Agent

**6. Give every agent a single, precise identity sentence.**
Not "you are a helpful assistant." Instead: "You are a verification specialist. Your job is not to confirm the implementation works — it's to try to break it." One sentence that establishes a role, a goal, and a stance. That sentence primes everything that follows.

**7. Separate stable rules from volatile context.**
Put behavioral rules in the system prompt. Put session-specific context (current file, git status, user preferences) in the messages layer. Never mix them. Stable content can be cached; volatile content cannot.

**8. Use XML tags to label injected content.**
Whenever you inject external content (file contents, search results, database output), wrap it in a named tag: `<file_content>`, `<search_result>`, `<user_data>`. Tell the model in the system prompt what each tag means. This prevents the model from treating external content as instructions.

**9. Write anti-examples into your few-shot prompts.**
When you want the model to output a specific format, don't just show the right format. Show the wrong format labeled as "Bad:" and the right format labeled as "Good:". The model learns the failure shape, not just the success shape.

**10. For safety-critical constraints: remove the tool, then explain why in the prompt.**
If an agent must never write files — don't just say "don't write files" in the prompt. Remove the write tool from the API call entirely. Then explain in the prompt why the constraint exists. Prompts are soft constraints. Tool removal is hard enforcement. Use both, in layers.

---

## The One-Line Summary

> Prompt engineering is not about magic words.
> It's about giving a very smart function the precise context it needs to produce the output you want — reliably, safely, and economically.

Claude Code's source code is one of the best prompt engineering textbooks available. Read it.

---

## Quick Reference Card

| Principle | What to do |
|---|---|
| Explain the why | Don't just state rules — include the failure mode |
| Name failure modes | Anticipate and name how the model will go wrong |
| Anti-examples | Show bad output labeled as bad, not just good output |
| Static first | Stable content before volatile content in the prompt |
| Named channels | Wrap injected content in XML tags with defined meaning |
| Hard + soft constraints | Remove tools AND explain rules for safety-critical behavior |
| Single identity sentence | Every agent needs one precise sentence defining its role |
| Budget discipline | Every token costs; only inject what earns its place |
| **Signal word hierarchy** | STRICTLY PROHIBITED > CRITICAL > IMPORTANT > NEVER > DO NOT — match word to stakes |
| **ALL CAPS placement** | Only the signal word + key contrast term. Never full sentences in caps. |
| **Zone 1 length (identity)** | 1–2 sentences maximum, always |
| **Zone 2 length (rules)** | 1 bullet per rule. Reason fits in the same bullet. |
| **Zone 3 length (agents)** | As long as the task needs. Every line earns its place. |
| **Format selection** | Prose for nuance, bullets for discrete rules, headers for navigation, code blocks for output format |
| **Output length instructions** | Tell the model explicitly how long to respond and in what structure |

---

## Where to Look in the Claude Code Source

| What you want to understand | File to read |
|---|---|
| How the full system prompt is assembled | [src/constants/prompts.ts](src/constants/prompts.ts) — `getSystemPrompt()` at line 444 |
| How prompt priority/hierarchy works | [src/utils/systemPrompt.ts:41](src/utils/systemPrompt.ts#L41) — `buildEffectiveSystemPrompt()` |
| A masterclass in adversarial prompting | [src/tools/AgentTool/built-in/verificationAgent.ts:10](src/tools/AgentTool/built-in/verificationAgent.ts#L10) |
| How CLAUDE.md files are loaded | [src/utils/claudemd.ts:790](src/utils/claudemd.ts#L790) — `getMemoryFiles()` |
| How context gets assembled before API call | [src/query.ts:58](src/query.ts#L58) and [src/query.ts:660](src/query.ts#L660) |
| How XML tags are injected into messages | [src/utils/api.ts:449](src/utils/api.ts#L449) — `prependUserContext()` |
| How tool descriptions are structured | [src/Tool.ts](src/Tool.ts) + [src/tools/BashTool/prompt.ts](src/tools/BashTool/prompt.ts) |
| How sub-agents are prompted | [src/tools/AgentTool/built-in/](src/tools/AgentTool/built-in/) |
| The static/dynamic cache split | [src/constants/systemPromptSections.ts:20](src/constants/systemPromptSections.ts#L20) |
| What the `<system-reminder>` tag means to the model | [src/constants/prompts.ts:190](src/constants/prompts.ts#L190) |

---

*Based on analysis of the Claude Code source code. All code references are to real files in the codebase.*
