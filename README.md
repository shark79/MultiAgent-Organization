# An Autonomous Dev Team, on Open-Weight Models

Five AI agents with different jobs, different models, and different permissions. One of them
only writes code. One of them only tries to break it. None of them can close their own defect.

This repo is the team itself — the agent definitions and the model routing. It runs on
[OpenCode](https://opencode.ai) with every model served through
[OpenRouter](https://openrouter.ai).

## The team

| Agent | Mode | Model | Job |
|---|---|---|---|
| `orchestrator` | primary | `moonshotai/kimi-k3` | Plans architecture, writes task specs, dispatches, reviews evidence, gates phases. Never writes code. |
| `backend-dev` | subagent | `z-ai/glm-5.2` | Server, database schema, API, business logic, backend unit tests. |
| `frontend-dev` | subagent | `minimax/minimax-m3` | UI, component architecture, frontend unit tests. Screenshots its own work and checks it. |
| `qa` | subagent | `xiaomi/mimo-v2.5` | End-to-end tests, full suites, accessibility audits, owns the defect log. Never fixes code. |
| `adversary` | subagent | `deepseek/deepseek-v4-pro` | Security and business-logic attacks against the running app. Never fixes, never triages its own findings. |

Different models on purpose. The orchestrator's job is judgment, so it gets the strongest
reasoning model and is told to spend it on judgment rather than busywork. The developers get
models that are good and cheap at writing code. The adversary gets a model that is good at
finding the odd path.

## What actually makes it work

Not the prompts. The constraints.

- **Separation of duties, enforced by permissions, not by asking nicely.** Each agent's
  frontmatter has an `edit` allowlist. `qa` physically cannot edit product source. Developers
  physically cannot edit the defect log or the end-to-end tests. The adversary can write to
  exactly one file. An agent can't quietly grade its own homework, because the filesystem
  won't let it.
- **Only QA closes a defect.** A developer reporting "FIX READY" doesn't close anything. QA
  reruns the exact reproduction steps, regression tests around the fix, and only then sets
  CLOSED — or sets it back to OPEN.
- **A hard loop breaker.** After two reopens on the same defect, the orchestrator is forbidden
  from dispatching it a third time and must escalate to a human. This is a cap, not a judgment
  call, because a developer and a QA agent that disagree will otherwise argue until the
  budget runs out.
- **A human checkpoint at the end of every phase.** The orchestrator presents evidence and
  stops. It cannot self-approve into the next phase. That gate is where product taste gets
  injected — the one thing none of the five agents has.
- **Evidence, not assertions.** Closing a phase means walking every success criterion with a
  passing test run or a screenshot attached to it.
- **An append-only audit log.** Every agent appends one entry per action and may never edit a
  prior one. If something was wrong, the correction is appended; history is not rewritten.

## What it cost

One full build — plan, implement, test, attack, fix, ship:

| | |
|---|---|
| Total model spend | **$11.56** |
| Tokens | 27.5M |
| Requests | 340 |
| Cache hit rate | 88.2% |
| Blended cost | $0.42 per 1M tokens |

The interesting part isn't the number, it's what the number implies: with real gates —
tests that must pass, an adversary that must be satisfied, a human who must sign off — open-weight
models were good enough. The quality came from the process, not from buying the most expensive
model available.

## Running it

```
npm i -g opencode-ai          # or however you install OpenCode
export OPENROUTER_API_KEY=... # your own key, never committed
opencode
```

The orchestrator is the primary agent, so it takes the first message. Give it a requirements
document and it plans from there.

## What is deliberately not in this repo

Only the agents and their model routing are published here. Not included:

- The per-role training knowledge bases and policy files the agents load on startup.
- The requirements contract, defect log, adversarial review log, and audit log — these are
  outputs of a specific build, not part of the team.
- Any application code the team produced.

The agent files still reference those paths, so this is a readable blueprint rather than a
turnkey clone. Read the frontmatter and the hard rules; that is where the design lives.
