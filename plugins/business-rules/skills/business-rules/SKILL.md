---
name: business-rules
description: Creates and enforces BUSINESS_RULES.md, the project's business rules contract (separate from CLAUDE.md), which the AI must obey and only the human can change. Use it when asked to define, document or establish business rules, scope or limits of a product; when asked to modify an existing rule; and before writing or altering product code in a project that already has BUSINESS_RULES.md, to verify the change doesn't break any rule. Don't use it for questions, doubts or explanations that don't alter code.
---

# Business Rules

Business rules — the limits and invariants that define what a product can and cannot do — get lost in conversation memory or end up as loose constants in the code, with no author and no provenance. Once that happens they stop being a contract and become a negotiable implementation detail, and anyone (including you) ends up proposing the shortcut around them in good faith. This skill keeps them in `BUSINESS_RULES.md` at the project root, separate from `CLAUDE.md` because they answer different questions: `CLAUDE.md` says how to work in this repo, `BUSINESS_RULES.md` says what is non-negotiable.

The rule that governs everything else: **the human changes the rules, not you.** Once the file exists, your job is to comply with it and flag conflicts, never to rewrite it so new code fits.

## Step 0 — Detect the mode

Does `BUSINESS_RULES.md` exist at the project root?

- **It doesn't** → creation mode.
- **It does** → load it and ask the user what they need: to use it as reference for the coding task ahead, to change/add/remove something, or to add new context. If their intent is already obvious from what they asked, don't ask again.

## Creation mode

First check whether there is functional code. Empty scaffolding from a project generator doesn't count; anything with real logic does (models, endpoints, validations, components with behavior).

- **No code**: ask the context questions in `references/intake.md`.
- **With code**: read it first and treat it as *raw context* — it's your starting point, but incomplete and ambiguous. Use it to ask deeper questions anchored to the actual code (what decisions it leaves implicit, what edge cases it doesn't cover, what rules you suspect but aren't confirmed) instead of the generic ones from the list.

Either way: write the file using the template below, in the language the user is speaking, with rules stated in plain, verifiable terms. "A user cannot see another user's orders" works; "the system must be secure" doesn't. Anything the user didn't tell you goes in marked as pending, never invented.

## Reference mode — before writing code

Check what's about to be built against every rule in the file. Look for two kinds of conflict:

- **Direct**: it contradicts the text of a rule (the rule says "maximum 50% discount" and they ask for an 80% coupon).
- **Indirect**: it doesn't contradict the text but breaks the rule in practice (the rule requires the amount paid to match the total, and they ask to add tips to the charge without touching the total). These are the dangerous ones because nobody sees them coming. Hunt for them by asking yourself: *what rule could stop holding as a consequence of this?*

**If there's a conflict, stop before writing the code.** If you generate it anyway and warn afterwards, the user already has an implementation that contradicts their own contract, and undoing it costs far more than a thirty-second conversation.

The warning has four blocks, because each answers something different the user needs in order to decide:

1. **Which rule breaks** — quoted verbatim and by number, so they can open the file and read the same line.
2. **How it breaks** — the concrete mechanism: what the code would do that the rule forbids, or what condition would stop holding. If it's indirect, show the chain of effects.
3. **What it means for the proposed design** — the real cost: which parts of the design no longer hold up, what gets redone, what's affected downstream, what risk they take on if they proceed anyway.
4. **What their options are** — at least two, with the trade-off of each. Typically: adjust the implementation, change the rule (their call, via update mode), or a third path that satisfies both goals. Be concrete: "use a `tip` field separate from `total`" works; "consider refactoring" doesn't.

When the recommendation touches verifiable ground — current regulation, legal or tax limits, industry standards, documented behavior of an API or payment gateway — look it up and cite the source instead of deducing it from memory: your knowledge may be out of date and the decision has real consequences. If the user asks for real examples, find them with their source; an invented example presented as real is worse than none. For trivial internal-logic conflicts, with no regulatory or third-party implications, don't search anything: the four blocks are enough and searching only adds noise.

There's a full example of a well-built warning in `references/warnings.md`. Read it if this is the first warning you're issuing in the session, or if the conflict is indirect or tangled.

**After the warning**, wait for the decision. If the user decides to change the rule, switch to update mode — don't edit `BUSINESS_RULES.md` as part of implementing the feature, because that would turn every implementation into implicit authorization to rewrite the contract. If they decide to break it without changing it, you can (it's their product), but say so explicitly so there's a record that it was a conscious call.

## Update mode

Apply the change the user asked for, leaving the rest of the file untouched, and show the before → after of what changed. This is the only case where you edit the file: you do it on explicit human instruction, not on your own judgment while writing code.

## BUSINESS_RULES.md template

```markdown
# Business Rules — [Project name]
Last updated: [date]

## Purpose
[One or two sentences: what this product or module is about, and why it exists.]

## Scope
### What it does
- ...

### What it does NOT do
- ...

## Users
[Who uses it: roles, account types, relevant permissions.]

## Rules
1. [Clear, verifiable rule, e.g. "An order cannot be marked as paid if the recorded amount doesn't match the calculated total."]
2. ...

## Special cases and exceptions
- ...

## What happens when a rule is violated
- ...

---
These rules cannot be altered by the AI. Only the human responsible for the project may change them, and only through an explicit change request.
```
