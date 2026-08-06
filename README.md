# Business Rules

A Claude Code skill that keeps your project's business rules in a file the AI has to obey — and **stops before writing code that would break one**, instead of writing it and mentioning the conflict afterwards.

`CLAUDE.md` says how to work in your repo. `BUSINESS_RULES.md` says what is non-negotiable for the business. This skill owns that second file.

---

## Install

```
/plugin marketplace add Aaroneverrutp/business-rules
/plugin install business-rules@business-rules
```

If it prints `Run /reload-plugins to activate.`, run that.

**No GitHub, just for yourself?** Copy `plugins/business-rules/skills/business-rules/` into `~/.claude/skills/business-rules/` and it loads in every project automatically.

**Tie it to one project?** Copy it into `.claude/skills/business-rules/` inside the repo and commit it — everyone who clones gets it.

---

## What it actually does

### 1. Creation — you have no rules file yet

It checks whether the project already has functional code (empty scaffolding doesn't count).

- **Greenfield**: asks context questions — what it does, what it explicitly does *not* do, who it's for, what's non-negotiable, what happens when a rule is violated.
- **Existing code**: reads the code first, treats it as *raw context*, and asks deeper questions anchored to what the code leaves ambiguous. Not "who are your users?" but "I don't see a role check in `markAsPaid` — can anyone mark an order as paid?"

Whatever you didn't tell it gets written down as **pending**, never invented. A made-up rule is worse than a missing one, because it looks agreed upon.

### 2. Reference — before it writes code

It checks the change against every rule and looks for two kinds of conflict:

| Kind | What it looks like |
|---|---|
| **Direct** | Contradicts the rule's text — rule says max 50% discount, you ask for an 80% coupon |
| **Indirect** | Doesn't contradict the text but breaks the rule in practice — rule requires paid amount to match the total, you add tips to the charge |

Indirect ones are the dangerous ones. Nobody sees them coming.

When it finds one, it **stops before writing anything** and gives you four things:

1. **Which rule breaks** — quoted verbatim and by number, so you can go read the same line
2. **How it breaks** — the concrete mechanism, naming the responsible line of code; for indirect conflicts, the full chain of effects
3. **What it means for your design** — what stops holding up, what gets redone, what's affected downstream, what risk you take on
4. **Your options** — at least two, with trade-offs, including changing the rule (your call, routed through update mode so there's a record)

When the recommendation touches verifiable ground — regulation, tax limits, documented API behavior — it looks it up and cites the source rather than deducing it from memory.

### 3. Update — changing a rule

The only case where it edits `BUSINESS_RULES.md`, and only when you explicitly ask. It preserves the rest of the file and shows you the before → after.

---

## Why a file, and not just telling it

Same request tested three ways — asking for a 70% coupon when the established limit was 50%:

| Where the 50% limit lived | Score | What happened |
|---|---|---|
| `BUSINESS_RULES.md` | **8/8** | Defended it, routed the change through a process that leaves a record |
| The conversation, 8 turns earlier | **6/8** | Remembered and quoted it — but it wouldn't survive a new session or a second developer, and there's nowhere to record an authorized exception |
| Only a constant in the code | **2/8** | Questioned it — *"I don't know if this is a deliberate business decision or a number someone typed two years ago"* — and **recommended the workaround, with the code already written** |

Without documented rules the constraint doesn't vanish at once. First it becomes debatable, then negotiable, and eventually the assistant hands you the shortcut in good faith.

---

## Repo layout

```
business-rules/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace catalog — what /plugin marketplace add reads
├── plugins/
│   └── business-rules/
│       ├── .claude-plugin/
│       │   └── plugin.json       # Plugin manifest: name, version, author. Bump version to ship updates
│       └── skills/
│           └── business-rules/
│               ├── SKILL.md      # The skill itself. Modes, warning protocol, file template
│               └── references/
│                   ├── intake.md    # Context questions — loaded only in creation mode with no code
│                   └── warnings.md  # Worked warning example — loaded only when issuing a warning
├── LICENSE
└── README.md
```

### Token cost

Only the frontmatter description (~150 tokens) is always in context. Everything else loads on demand based on the active mode:

| Scenario | Tokens |
|---|---|
| Session where it never triggers | ~150 |
| Update mode / reference with no conflict | ~1,900 |
| Creation from scratch (loads `intake.md`) | ~2,250 |
| Issuing a warning (loads `warnings.md`) | ~3,065 |

Splitting it this way cut the always-on cost by 59% and the loaded body by 49%, with no drop in quality across the test cases.

---

## Publishing updates

Bump `version` in `plugins/business-rules/.claude-plugin/plugin.json`, commit, push. Users refresh with:

```
/plugin marketplace update business-rules
```

---

## License

MIT
