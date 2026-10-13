---
marp: true
theme: paper-ink
paginate: true
---

<!-- _class: title -->

# Agent Harnesses

What a harness is, how to build one, and what the papers measured

Prepared for Chaos, 13 Oct 2026 (BR-17, BR-18). Sources: 20 papers and articles in the vault KB,
24 web sources, and our own scheme-agent bench. Every outside number comes from a source on the
last two slides, and every number of ours from the results files named on slide 18.

---

## Agent = Model + Harness

The model picks the next step. The harness is everything else:

- the loop that calls the model and runs what it asks for
- the tools and their descriptions
- the context the model sees on each turn
- memory and state that outlive one context window
- the checks that decide whether the output counts
- the sandbox and permissions around it

The framing comes from Osmani, Saboo and Kartakis, *The New SDLC with Vibe Coding* (May 2026).
They claim most agent failures are harness failures. The controlled papers below narrow that
claim: some harness changes help a lot, and some help nothing.

---

## The smallest loop that works

1. A system prompt that states the task and the rules.
2. A few tools, each with a tight description and an error message that says what to do next.
3. The loop: call the model, run the tool, append the result, repeat until done or a step limit.
4. A verifier that checks the output before the run counts as finished.
5. Context management that trims or summarises before the window fills.

mini-swe-agent does this in about 100 lines with bash as its only tool and scores over 74% on
SWE-bench Verified. Anthropic's *Building Effective Agents* gives the same advice: start simple
and add a part only when you can measure the gain.

---

## The harness moves scores as much as the model

| Source | Change | Result |
|---|---|---|
| LangChain, Feb 2026 | harness only, gpt-5.2-codex fixed | Terminal-Bench 2.0: 52.8 → 66.5 |
| *Stop Comparing LLM Agents…* (2605.23950) | harness only | Terminal-Bench 2: 69.7 → 77.0 |
| PRISM, Airbnb (2609.05736) | optimised harness vs Deep Agents | tau²-Retail +14.9, BFCL +14.2 |
| StarHarness (2608.24804) | evolved harness, weights fixed | +20 to +35 points |
| AutoHarness (2603.03329) | synthesised move checker | Gemini Flash beats Pro, 56.3% vs 38.2% |

Harness choice can also reverse model rankings (2605.23950). So a model comparison is only valid
when the harness is held fixed.

Caveats: StarHarness is abstract-only in the KB. The 2605.23950 figures come from a search
snippet.

---

## Paper: *An Empirical Study of Harness Design for Coding Agents*

arXiv:2609.20804, Fan et al., Sep 2026. One plain ReAct loop, one component changed at a time,
176 settings, 4 models, SWE-Bench Verified and Terminal-Bench 2.1.

- **Context management always paid under a tight window.** At 32k, Mistral went from 12.6% to
  66.2% and Nemotron 550B from 6.4% to 58.4%. The big model did worst unmanaged because it fills
  the window fastest.
- **The gain fades as the window grows.** Unmanaged Mistral reaches 67.4% at 128k.
- **Staged compaction wins on cost, not accuracy.** It removes old tool output at a soft limit and
  summarises at a hard one. Its accuracy tied the other policies, at the lowest cost per task.
- **The model mostly ignored the tool that restores removed output.** It went uncalled in 36 of
  64 settings.

---

## Same paper: components flip with model size

| Model | SWE-Bench, tools vs bash | Terminal-Bench, tools vs bash |
|---|---|---|
| Nemotron 30B | **25.2** vs 10.2 | **13.5** vs 3.4 |
| Nemotron 550B | 65.8 vs **69.4** | 44.9 vs **50.6** |
| Mistral 128B | **68.6** vs 45.4 | 37.1 vs **43.8** |

Planning behaved the same way. It helped the 30B model and cost the 550B model money for nothing.

**For your harness:** a component is not good in general. Re-measure tools and planning every
time you change the model. Count calls on every optional tool, because an unused tool is cost
with no effect.

Caveats: one run per setting, three of four models from one family, and the KB copy was rebuilt
from a slide deck.

---

## Paper: SWE-agent and the agent-computer interface

Yang et al., NeurIPS 2024 (arXiv:2405.15793). GPT-4 Turbo on SWE-bench Lite.

| Interface choice | Score |
|---|---|
| Full custom interface | 18.0% |
| Shell only | 11.0% |
| No editor | 10.3% |
| Editor without the lint guard | 15.0% |
| File viewer at 30 lines / whole file (vs 100 lines) | 14.3% / 12.7% |
| Full history (vs last 5 observations) | 15.0% |

Tools are a user interface for the model. Bound what each one returns, and reject bad edits at
the tool.

---

## Minimal harnesses hold up well

- **mini-swe-agent:** bash only, one fresh subprocess per action, a linear history. Over 74% on
  SWE-bench Verified.
- **pi** (Mario Zechner): four tools, a prompt plus tool spec under 1k tokens, no MCP, no
  sub-agents. Its reason for skipping MCP: one browser MCP server costs 13.7k to 18k tokens.
- **Brilliantov, Hernández-Cano and Abbé** (arXiv:2609.40303, Sep 2026):
  - giving the agent a shell and filesystem instead of chat was the largest effect
  - tree search, greedy and best-of-N were statistically indistinguishable
  - one 24-hour session medalled in 62.5% of competitions on GLM 5.2, against 47.1% for the best
    elaborate MLE system
  - agents that messaged each other scored 33%, against 56% without messaging

Caveat on the last paper: the KB holds a summary rather than the text, and the confidence
intervals are wide.

---

## Context is an attention budget

- **Context Rot** (Chroma, Jul 2025): all 18 models tested got worse as input grew, even on
  simple tasks. A focused ~300-token input beat the full ~113k-token one on LongMemEval.
- **Anthropic, *Effective context engineering*:** fetch content just in time through paths and
  IDs. Keep a long task alive with compaction, note files, and sub-agents that return
  1,000–2,000-token summaries.
- **Manus:** the KV-cache hit rate is the key cost metric. Input to output runs about 100:1, and
  cached tokens cost a tenth as much. Keep the prompt prefix stable and the context append-only.
  Mask tools rather than removing them, and keep errors in context.
- **Infinite-Parameter LLMs** (arXiv:2609.18842): ten passages read as one concatenation fell
  from 48.8 to 27.8 F1. Processing each passage separately and selecting at query time held flat
  at about 48.

---

## Two ways to manage context

**Harness-driven:** the harness decides when to remove and summarise. This is the staged policy
from 2609.20804. AIDE² (arXiv:2609.26457) found the same fix by itself: summarising history
shrank prompts 7x to 50x and stopped runs dying from prompt growth.

**Model-driven:** *Context language models* (arXiv:2609.37725, UW / Meta). The harness mirrors
the live context into a file and the model edits it with Bash.

- BrowseComp-Plus: 59.4%, an 11.4% relative gain over summarisation at 21.5% fewer FLOPs.
- After RL training, accuracy matched a summary harness trained the same way, 42.5% vs 42.1%.
  The real gain was compute: 1.34 vs 2.19 PFLOPs per question.
- Editing mid-context breaks the prompt cache. A context the model can edit also lets a prompt
  injection persist.

**Choice for a new harness:** start harness-driven. It is cheaper, cache-friendly and
measurable.

---

## Memory across runs

- **Recuris** (arXiv:2608.24876): Claude Opus 5 went from 72.4 to 87.9 on tau²-Retail.
  - The framework with an **empty** memory package changed nothing measurable.
  - Structured task state (pending, done, blocked) did most of the work: 82.0 alone vs 83.6 full.
  - Failures were on the write path. The base agent made no writes in 42% of episodes that needed
    one.
- **DolphinBench** (arXiv:2609.24971, Mem0): a test counts only if an agent with the right memory
  passes it twice and an agent without it fails twice. Discount its ranking, because Mem0 wrote
  it.
- **MAC** (arXiv:2608.21265): random retrieval captured +7.2 of a +13.2 gain. Summaries helped
  (+6.67), and replayed traces hurt (−8.33 to −11.67).

**For your harness:** run every memory feature against an empty-content control and a
random-retrieval control.

---

## Long-running agents

Anthropic, *Effective harnesses for long-running agents* (Nov 2025):

- An initializer run writes a setup script, a JSON feature list with every feature failing, a
  progress file and a first commit.
- Each later session takes one feature, tests it end-to-end, commits, and updates the progress
  file.
- The prompt states that editing or deleting tests is unacceptable.
- The target failures are declaring victory early, leaving broken state undocumented, and
  wasting a session on setup.

Anthropic, *Managed Agents* (Apr 2026), splits the system three ways:

- the model plus a stateless harness
- a sandbox per session
- an append-only event log outside the context window

Time to first token fell about 60% at p50. The post also warns that harness workarounds go
stale. Context resets added for Sonnet became unnecessary on Opus.

---

## Tools and guards

- **Anthropic, *Writing effective tools*:**
  - name tools by service and resource
  - return readable fields, not UUIDs (one response fell from 206 to 72 tokens)
  - cap output, since Claude Code caps a tool response at 25k tokens
  - write errors that point at the right next action
- **PRISM** (arXiv:2609.05736): deterministic guards at the tool boundary fix a failure silently,
  block an error, or block a call whose prerequisite is missing. Routing each failure to the
  layer that can see it mattered. Without routing, Retail lift fell from +14.9 to +5.4.
- **AutoHarness** (arXiv:2603.03329): 78% of Gemini Flash's chess losses were illegal moves. A
  synthesised legality checker reached 100% legal moves after about 14.5 refinement rounds.

---

## Verification before "done"

- LangChain's 13.7-point gain came mostly from checks:
  - a self-verification prompt
  - a checklist step that runs before the agent may finish
  - loop detection on repeated edits to one file
  - time-budget warnings
- The Claude Agent SDK loop is gather context, act, verify, repeat. Verification can be a rule, a
  visual check, or an LLM judge.
- Cognition (Apr 2026): a reviewer with a clean context that sees only the diff is one
  multi-agent pattern that works.
- AIDE²: about a quarter of the rewrites it rejected had beaten the incumbent on the signal the
  agent could see. Score on a signal the agent cannot query.

---

## Sub-agents: read in parallel, write once

- **Anthropic's research system:** an Opus lead with Sonnet sub-agents beat single-agent Opus by
  90.2%. It also used about 15x the tokens of a chat, and token use explained 80% of the variance.
  Anthropic says the pattern fits coding poorly.
- **Cognition, *Don't Build Multi-Agents*:** parallel writers make conflicting decisions. Share
  full traces, and default to one linear agent.
- **Cognition, Apr 2026:** multi-agent works when writes stay single-threaded and the other agents
  only contribute analysis.
- **2609.40303:** agents that messaged each other scored 33%, against 56% without messaging.

**For your harness:** use sub-agents as a context firewall for reading and research. Keep one
writer.

---

## Harnesses that improve themselves

| Paper | What it evolves | Result | Main caveat |
|---|---|---|---|
| AIDE² (2609.26457) | its own agent code | 0.703 → 0.778 over 100 steps; held on other models | 3-seed check inconclusive |
| PRISM (2609.05736) | prompts plus tool middleware, under edit limits | +10 to +15 | removing the edit limits cut lift to +0.7..+6.5 |
| StarHarness (2608.24804) | prompts, tools, skills, sub-agents, loop config | +20 to +35 | abstract only, no control |

The KB records PRISM and StarHarness as an unresolved conflict over how wide the evolvable surface
should be.

*On the Fragility of Self-Improving Agents* (arXiv:2608.18066, Salesforce): a loop on a noisy
eval widens variance, and gains vanish when the task order is shuffled.

---

## Evaluating your harness

- **BenchJack** (arXiv:2605.12673, Berkeley): a red-team agent reached near-perfect scores on 10
  agent benchmarks without solving a task. It found 219 flaws in 8 classes, including a shared
  environment, exposed answers, and trusting signals the agent writes.
- **SWE-bench issue #465:** models read future fixes with `git log --all` and `reflog`.
- **The defence:** a null agent, a random agent and a state-tampering agent must all score zero.
- **Variance:** use several runs. Report pass^k (all k trials succeed) for production use. The
  ablation study and ROFT both rest on one run per setting.
- **Controls:** run every new component with empty content and compare against the unmodified
  harness.

---

## Our experiments: the scheme-agent bench

scheme-agent is our own harness. The model writes Chez Scheme, and the harness checks and runs it.
Its DFA bench asks the agent to learn a hidden finite automaton by playing the L\* learner against
an oracle:

- A membership query asks whether one word is in the language.
- An equivalence query submits a whole hypothesis DFA and gets back a pass or a counterexample.
- Grading is exact, by automaton equivalence, so no judge model is involved.
- Par is computed per target from L\* and TTT, so a score is queries against a known learner.

The panel is gemma4 31B, minimax-m3, nemotron-3-ultra and qwen3.5, all at temperature 0 through
Ollama. The results live in `memory-solution/bench/data/automata/results/` and
`bench/REPORT-HELDOUT-v1.md`.

---

## Our harness bugs looked like model failures

| Harness defect | What the scores said | After the fix |
|---|---|---|
| A run ended on its first turn without a tool call | 12 zero scores on the ladder | 9 of the 12 were artifacts |
| `--memory` got a directory, not a file | all 4 models scored 0.0 on memory tasks | 3 of 4 pass both binary tasks |
| A 60-turn cap, below dfa03's L\* par of 69 | dfa03 and dfa04 looked like walls | minimax solved dfa03 |
| A 4096-token output cap | 27 of 60 runs cut off mid-turn | cloud cut-offs 15 → 0, solves 0 → 1 |

The last row shows a fix can be real and still move the score very little.

**For your harness:** record why every run ended: solved, budget out, quit, truncated or
transport failure. A zero with no recorded reason is not a measurement.

---

## Our ladder: hand-written targets

| Model | dfa01, 3 states | dfa02, 4 | dfa03, 6 | dfa04, 8 |
|---|---|---|---|---|
| gemma4 | Y | N | N | N |
| minimax | Y | N | **Y**, 65 queries vs L\* par 69 | N |
| nemotron | Y | N | N | 1 solve in 5 runs |
| qwen3.5 | Y | N | N | N |

minimax failed dfa02 while holding the right answer. Asked after its budget ran out, it gave a DFA
equivalent to the target, one submission too late.

Caveats: one run per cell outside dfa04, so the minimax row is not proof that dfa02 is harder than
dfa03. Do not read a rate into nemotron's 1 of 5.

---

## Our ladder: what moved the scores

- **Generated targets are much harder than hand-written ones.** On the same harness and cap,
  dfa01 scored 8/12 and a random 3-state target scored 0/12. All 48 runs on four generated targets
  failed.
- **Prompt wording is a harness component.** A prompt describing the guess-and-refine loop raised
  solves from 0/48 to 2/48 and equivalence queries from 31 to 82. gemma4 quit all 12 of its runs
  under it.
- **Held-out tasks lowered scores.** On Scheme tasks written after the harness was frozen, qwen fell
  from 5/5 to 2/3 and gemma4 from 4/5 to 2/3.

**For your harness:** test on generated tasks and a held-out set as well as the tasks you tuned on.
Our hand-written tasks flattered every model.

---

## The harness-checker, and where it goes

`memory-solution/harness-checker/` checks Scheme the model wrote, statically, before it runs:

- **Whitelist:** every call must name an approved procedure, and macros are forbidden.
- **Types:** arity and types are checked against stdlib signatures and the tools' JSON Schemas.
- **Termination:** every loop and recursion must be proved to end, or the program is blocked.

Put it at the tool boundary, inside the tool that runs code:

1. The model proposes code.
2. The checker runs. A violation goes back as the tool's error, naming the failing call, so the
   next turn can repair it.
3. Only clean code reaches the evaluator, which keeps its own timeout and restricted bindings.

scheme-agent's `execute_scheme_code` tool already works this way. It is the guard pattern from
slide 13, like AutoHarness's legality checker. Count its rejections, because a guard that never
fires is cost with no effect.

---

## Our memory-solution idea

memory-solution keeps whole past sessions rather than extracted facts, and the agent must search
them before it does anything else. It writes that search three ways: a technical description of
what it needs, a plain-language summary, and a guess at the text it expects to find, such as a log
line. Each phrasing is embedded, cut to a 768-bit binary vector and matched by Hamming distance
inside SQLite, so recall needs no vector database. When the work ends, a hidden note-taker agent
writes a note for the next session and scores the recalled sessions from -2 to 2. That score is
added to each recalled session's rank, and search orders its matches by rank before distance. The
memories that helped come back first next time.

---

## Sandbox and security

- Anthropic, *Beyond permission prompts*: sandboxing cut permission prompts by 84%. You need
  filesystem **and** network isolation, because either alone still allows escape or exfiltration.
  Claude Code uses bubblewrap on Linux and seatbelt on macOS.
- Managed Agents and Claude Code on the web keep credentials out of the sandbox. Git goes through
  a proxy that holds the token.
- Codex runs every tool inside an OS sandbox (Seatbelt or Landlock).
- pi takes the opposite view and runs with no permission gates by design. Zechner's reason: "as
  soon as your agent can write code and run code, it's pretty much game over."
- 2609.37725: memory the model writes for itself is a place where a prompt injection can persist.

---

## Design principles for a new harness

1. Start minimal: one loop, a few tools, one writer.
2. Treat tools as an interface. Bound every output, guard edits, and write errors that steer.
3. Make context management the first component you add, and stage it: remove first, summarise
   second.
4. Keep durable state outside the window, in a progress file, git and an event log.
5. Gate "done" on a verifier the agent cannot game.
6. Use sub-agents for reading, not writing.
7. Design for the cache with a stable prefix and append-only context.
8. Sandbox at the OS level and keep credentials out.
9. Measure each change against the unmodified harness, with an empty-content control, on more
   than one run.
10. Re-tune when you change the model, because components flip sign with scale.

---

## Questions to settle first

- **Which model, or models?** Planning and tool sets flip with model size, so the answer decides
  which components to build.
- **What is the task class?** Coding favours one writer. Research tolerates parallel readers.
- **What is the verifier?** Without one, there is nothing to stop on and nothing to measure.
- **How long do runs go?** Long runs need staged compaction and a progress file from day one.
- **What is your eval?** You need 20 or more tasks, several runs each, plus the null, random and
  tamper agents.

---

## Reading list: papers in the vault KB

| Paper | arXiv | Why read it |
|---|---|---|
| Empirical Study of Harness Design | 2609.20804 | the controlled ablation |
| AIDE² | 2609.26457 | self-improving harness, bounded context |
| PRISM / Beyond Prompts | 2609.05736 | guard middleware, budgeted lift |
| Recuris | 2608.24876 | memory, empty-package control |
| Minimal coding agent vs MLE harnesses | 2609.40303 | minimal beats elaborate |
| Context language models | 2609.37725 | model-edited context |
| StarHarness | 2608.24804 | wide evolved harness |
| AutoHarness | 2603.03329 | synthesised verifiers |
| BenchJack | 2605.12673 | eval hacking |
| Fragility of Self-Improving Agents | 2608.18066 | variance, task order |
| DolphinBench | 2609.24971 | memory graded by action |
| MAC | 2608.21265 | memory format, random baseline |

---

## Reading list: from the web

| Source | Why read it |
|---|---|
| Anthropic, *Building Effective Agents* | patterns, start simple |
| Anthropic, *Effective context engineering* | compaction, notes, sub-agents |
| Anthropic, *Writing effective tools* | tool design against evals |
| Anthropic, *Effective harnesses for long-running agents* | progress file, one feature per session |
| Anthropic, *Managed Agents* | model, sandbox, event log split |
| Manus, *Context Engineering* | KV-cache, append-only context |
| Cognition, *Don't Build Multi-Agents* | one writer |
| LangChain, *Improving Deep Agents with harness engineering* | +13.7 from the harness alone |
| OpenAI, *Harness engineering* | AGENTS.md as a map (read via a summary) |
| SWE-agent (2405.15793), mini-swe-agent | the interface, the minimal baseline |
| Chroma, *Context Rot* | length hurts every model |
| pi (Zechner) | four tools, no MCP |
