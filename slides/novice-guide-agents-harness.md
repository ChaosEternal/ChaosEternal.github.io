---
marp: true
theme: paper-ink
paginate: true
---

<!-- _class: title -->

# A novice guide to agents and harness

What an agent is made of, where it fails, and how you know it works

For anyone who has used a chat assistant and never built an agent

---

## A chat model on its own

Text in, text out, one reply at a time. On its own it:

- **cannot act:** it can write a command, not run one
- **remembers nothing:** the app re-sends the whole chat every time
- **knows only what is in front of it:** your files reach it only as text

A brilliant consultant on a phone call, with no hands and no notebook.

---

## An agent is a model in a loop with tools

```
messages = [instructions, user_request]
while True:
    reply = model(messages)
    messages.append(reply)
    if not reply.tool_calls:
        break
    for call in reply.tool_calls:
        messages.append(run_tool(call))
```

- **Turn:** one call to the model
- **Tool call:** the model asks for an action
- **Tool result:** what my code got back, read on the next turn

---

<!-- _class: big-number -->

# Agent = Model + Harness

The harness is the loop plus everything that decides what the model sees and may do.

Claude Code, Codex CLI and Cursor are harnesses around rented models.

---

## The model sees only its context window

- The **context window** is all the text the model reads on one turn, counted in tokens.
- It is **finite**: what does not fit is not there.
- It is **re-read in full every turn**: a big tool result on turn 3 is paid for on turns 4, 5 and 6.

Once the model is rented and the loop written, the only decision left is which tokens the model
reads on each turn.

---

<!-- _class: quote -->

> For a model, what is real is the tokens in its window.

From my post, *Context management is the only thing in agent*

---

## Every feature does one of three things

- **Admit:** what goes in. Tool definitions, instructions, results, retrieved notes.
- **Evict:** what comes out. Summaries, truncation, a script that hides raw output.
- **Recall:** how evicted text gets back. By name, by search, by an index.

Admit too much and every turn pays. Evict the wrong thing and a fact is gone. Recall by the wrong
key and the agent cannot find what it cannot name.

---

## The parts of a harness

| Part | What it decides |
|---|---|
| Tools | what the model can do; a description is a prompt |
| Instructions | standing text on every turn, e.g. `CLAUDE.md` |
| Memory | what survives from one session to the next |
| Knowledge and skills | text loaded only when the task needs it |
| Guardrails | permissions, sandbox, timeouts |
| Verification | tests and reviewers that say "this is wrong" |

---

## Memory has two shapes

- **Pushed:** loads into every run before the task is read.
  - The only place a correction works.
  - Costs tokens every turn, so keep it small.
- **Pulled:** loads only when the model searches for it.
  - Free until read.
  - Works only if the model suspects it exists and can name the key.

A pile of notes searched by similarity fails both ways.

---

## A skill is a loading rule

A short name always in the window, and a body that loads when the name matches.

- **M4Bench (2026):** the spec alone added 0.162 reward; wrapping it as skills added 0.024.
- **Skill optimizers (2026):** success +0.1 points, 16% more tokens, mostly extra turns.

The content does the work. Keep the wrapper thin.

---

## Why I keep a knowledge base

Andrej Karpathy's LLM wiki pattern:

- **Raw sources stay unchanged.** Emails, papers, daily notes.
- **The model maintains the wiki.** Each source compiles into topic pages.
- **An index names every page.** Recall by a name the model can see.

The work happens once per source, not on every turn. Skills stay short procedures. Memory stays
short-lived.

---

## One task, end to end

1. I email "Can you draft an outline". **No agent is running.**
2. A script without a model polls the inbox and finds it. **Plumbing.**
3. The scheduler starts a run with rules and memory loaded. **Instructions, pushed memory.**
4. The model fetches the mail and checks the sender. **Tools, guardrails.**
5. It reads my posts and notes by name. **Knowledge base, pulled memory.**
6. It writes, commits, checks the page serves. **Tools, verification.**
7. It replies and writes a note for the next run. **Memory.**

---

## Where agents fail

Most failures are harness failures:

- the **context** was wrong or buried
- a **tool** was missing, so it guessed
- **no check** ran on the result

Beginner mistakes:

- stuffing the window with everything
- giving the model dozens of tools
- no way to verify the work
- believing "done" without evidence

---

## The fashionable mistake: many subagents

- Context degrades at every handoff.
- Specialists overlap, and their instructions conflict.
- Runs get slower. One team ran about 2x faster after merging.

One loop that loads skills and tools on demand usually wins. Keep a subagent for parallel work
whose context can be thrown away, such as a search or a review.

---

## Evaluations: how you know it works

A fixed set of tasks, scored the same way, every run. A demo is not one.

- **Score what it did, not what it said.** DolphinBench grades memory by the action taken.
- **Three ways to score:** a program, a person with a rubric, a model as judge.
- **One run is an anecdote.** Run each task several times and report the spread.

Ten tasks with known good results is enough to start.

---

## Evaluating a change

Every harness change is a guess until measured. Run the same tasks with and without it.

- **Harness^2 (2026):** blind edits dropped a score from 73.4% to 63.2%.
- **BenchJack (2026):** near-perfect on ten public benchmarks, zero tasks solved.

The cheap check: **an agent that does nothing must score zero.**

---

## How to start

- Use an existing harness before writing your own.
- Write the instructions file first, and keep it short.
- Add one tool at a time.
- Give the agent a checker it can run on its own work.
- Write ten evaluation tasks before the second feature.
- Ask of every feature: does it admit, evict or recall, and what does it cost every turn?
</content>
</invoke>
<invoke name="Bash">
<parameter name="command">cd ~/src/memory-solution/doc-review && git rev-parse --abbrev-ref HEAD && /home/overlord/.venvs/docreview/bin/python -c "
import sys; sys.path.insert(0,'.')
import view_specs, renderer
src=open('/home/overlord/brain/drafts/presentation-novice-guide-agents-harness.md').read()
print([n for n in dir(renderer) if 'block' in n.lower() or 'parse' in n.lower()][:10])
"