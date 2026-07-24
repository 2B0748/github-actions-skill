# I Built an AI Skill That Writes GitHub Actions Workflows — With Zero Config Errors

> Stop hand-crafting YAML. Let any AI agent generate production-ready workflows with 12 quality gates, 5 decision templates, and 3 mandatory security configs.

---

## The Problem

Hand-writing GitHub Actions YAML is error-prone. Did you remember `permissions`? Did you pin action versions? Did you configure caching? Is concurrency set?

Every omitted config is either slower builds, security holes, or wasted billable minutes.

## The Solution

I built an **AI Skill** — an executable instruction set, not a vague prompt. It forces any AI agent to follow a strict pipeline:

```
Decision Tree → Mandatory Config Injection → Cache Matching → 12 Quality Gates → Production YAML
```

### One sentence, instant result:

> "Add CI for my Node.js project — run eslint and jest on PRs"

**What the AI generates under the hood:**

1. ✅ Detects `package.json` → Node.js
2. ✅ Template A (CI) selected
3. ✅ `permissions: contents: read` injected
4. ✅ `concurrency` with auto-cancel
5. ✅ `cache: 'npm'` → ~70% faster installs
6. ✅ All actions SHA-1 pinned (`@11bd71...` not `@v4`)
7. ✅ 12-point checklist validated

**5 seconds. Zero omissions.**

## What Sets It Apart

| Approach | Universal | Security Check | Caching | Quality Gates |
|----------|-----------|---------------|---------|---------------|
| GitHub templates | ✅ | ❌ | ❌ | ❌ |
| Manual YAML | ✅ | ❌ | ❌ | ❌ |
| Raw ChatGPT prompt | ✅ | ❌ | ❌ | ❌ |
| **This Skill** | ✅ All agents | ✅ 5 red lines | ✅ 6 langs | ✅ 12 gates |

Raw LLM chats miss configs every time. This Skill makes them **mandatory**, not optional.

## 5 Decision Paths Cover 95% of Use Cases

- "lint / test / PR" → CI Workflow
- "Tag / Release / publish" → Release Workflow  
- "cron / scheduled / Dependabot" → Scheduled Workflow
- "security / CodeQL / scan" → Security Workflow
- "Issue / triage / auto-label" → Automation Workflow

Every template enforces **least-privilege permissions, concurrency control, and SHA-1 pinned actions** — no exceptions.

## 5 Security Red Lines

1. ❌ Hardcoded tokens → blocked
2. ❌ `pull_request_target` misuse → blocked
3. ❌ Overwriting workflows without confirmation → blocked
4. ❌ `@main` / `@master` action refs → blocked
5. ❌ `permissions: write-all` → blocked

## Universal — Works With ANY AI Agent

| Agent | How |
|-------|-----|
| **Claude Code** | `SKILL.md` → `.claude/skills/` |
| **Cursor** | `PROMPT.md` → `.cursor/rules/` |
| **GitHub Copilot** | `PROMPT.md` → `.github/copilot-instructions.md` |
| **Windsurf / Aider / Cline** | Paste into System Prompt |
| **International users** | `PROMPT.en.md` (English) |

## Before/After

```yaml
# ❌ Without Skill — typical raw LLM output
name: CI
on: push
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4    # unpinned
      - run: npm install              # no cache
      - run: npm test                 # no permissions, no concurrency

# ✅ With Skill — production-grade
name: CI
on:
  pull_request:
    branches: [main]
  push:
    branches: [main]
concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: true
permissions:
  contents: read
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      - uses: actions/setup-node@cd063736c72066a58f36dd95fb0460484b220fb7 # v4.3.0
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npx eslint .
  test:
    needs: lint
    # ... full pipeline
```

## Try It

```bash
git clone https://github.com/2B0748/github-actions-skill.git
```

👉 Star the repo if you find it useful. PRs welcome for new language templates and edge cases.
