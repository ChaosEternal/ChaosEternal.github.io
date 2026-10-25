# SDLC roles and playbooks: product owner, developer, tester

This page gives one entry per playbook. Most entries
give the same six fields, so any playbook can be lifted out on its own:

- **Trigger:** what starts it.
- **Inputs:** what must exist before it starts.
- **Steps:** the work, in order.
- **Output:** what it leaves behind.
- **Done when:** the check that ends it.
- **Handover:** who receives the playbook's handover record, defined below, and which of its parts
  that reader needs most.

The playbooks follow Scrum terms, because most delivery teams use them. A sprint is a fixed
timebox of one to four weeks. A story is one unit of user-visible value small enough to finish
inside a sprint. Acceptance criteria are the testable conditions a story must meet before the
product owner accepts it.

## Sources

The playbooks are not copied from any one book. Their steps were written from common delivery
practice, and most of the named practices in them have a known origin. This section records where
each one comes from, so a reader can check a term against its source.

Delivery practice:

- **Scrum terms.** Sprint, product owner, backlog refinement, sprint planning, sprint review,
  retrospective and definition of done follow [The Scrum Guide](https://scrumguides.org/scrum-guide.html)
  by Ken Schwaber and Jeff Sutherland, 2020 edition. The guide names three accountabilities:
  developers, a product owner and a Scrum Master. The tester role and the definition of ready are
  not in it.
- **Definition of ready.** Scrum practitioners added it around 2008, after the definition of done.
  The [Agile Alliance glossary](https://www.agilealliance.org/glossary/definition-of-ready/) traces
  the idea back to the 1985 ETVX framework.
- **Story form.** "As a user, I want a capability so that I get a benefit" was invented at
  Connextra in 2001, per the [Agile Alliance glossary](https://www.agilealliance.org/glossary/user-story-template/).
- **INVEST.** Bill Wake proposed the checklist in a 2003 article. Mike Cohn's *User Stories
  Applied* (2004) recommends it, per the [Agile Alliance glossary](https://www.agilealliance.org/glossary/invest/).
- **Given/When/Then.** Dan North introduced the scenario form in
  [Introducing BDD](https://dannorth.net/blog/introducing-bdd/) (2006).
- **Three amigos.** George Dinwiddie first used the term in this sense in 2009, per the
  [Agile Alliance glossary](https://www.agilealliance.org/glossary/three-amigos/).
- **MoSCoW.** Dai Clegg developed it in 1994 for rapid application development. DSDM adopted it
  from 2002, per [Wikipedia](https://en.wikipedia.org/wiki/MoSCoW_method).
- **Cost of delay.** Preston Smith and Don Reinertsen used it in *Developing Products in Half the
  Time* (1991). Reinertsen developed it further in *The Principles of Product Development Flow*
  (2009), as [Wikipedia](https://en.wikipedia.org/wiki/Cost_of_delay) describes.
- **WSJF.** Weighted shortest job first is from the same 2009 book.
- **Story points and velocity.** Both grew out of Extreme Programming around 2000. The Agile
  Alliance glossary dates [story points](https://www.agilealliance.org/glossary/points-estimates-in/)
  and [velocity](https://www.agilealliance.org/glossary/velocity/) separately.
- **Continuous integration.** Kent Beck developed it as part of Extreme Programming in the 1990s,
  per Martin Fowler's [Continuous Integration](https://martinfowler.com/articles/continuousIntegration.html).
- **Architecture decision record.** Michael Nygard proposed it in
  [Documenting Architecture Decisions](https://www.cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
  (2011).

Testing practice:

- **Charters and timed sessions.** Testing in a timed session against a charter is Jonathan and
  James Bach's [session-based test management](https://www.satisfice.com/download/session-based-test-management)
  (2000). Its standard session is about 90 minutes. The one-line charter follows Elisabeth Hendrickson's
  [Explore It!](https://pragprog.com/titles/ehxta/explore-it/) (2013).
- **Blameless post-incident review.** The model is
  [Postmortem Culture](https://sre.google/sre-book/postmortem-culture/), chapter 15 of Google's
  *Site Reliability Engineering* (2016).
- **Separate severity and priority.** This is common defect-tracking practice with no single
  origin, so it has no source here.

Agent practice:

- **Thin skills.** How Claude Code loads a skill follows the
  [Claude Code skills documentation](https://code.claude.com/docs/en/skills). Only the
  `description` stays in context, and the body loads when the description matches the work.
- **The 2026-10-25 revision.** Its five rules came from [swarm](https://github.com/swcstudiospace/swarm).
  The source of each:
  - The handover record follows
    [`swarm/schemas/handover.v1.json`](https://github.com/swcstudiospace/swarm/blob/main/swarm/schemas/handover.v1.json).
    The playbooks keep eight of its nine content fields and drop `constraints`.
  - The end states done, blocked and failed follow the task states in
    [`AGENTS.md`](https://github.com/swcstudiospace/swarm/blob/main/AGENTS.md).
  - The two-retry loop bound follows section 6 of
    [`01-architecture.md`](https://github.com/swcstudiospace/swarm/blob/main/01-architecture.md).
    The rule that evidence settles a disagreement first, then the strictest verdict, follows
    section 5 of
    [`04-integration-plan.md`](https://github.com/swcstudiospace/swarm/blob/main/04-integration-plan.md).
  - The rule that a skipped check is not a pass follows
    [`prompts/skills/a08-qa/gate-run.md`](https://github.com/swcstudiospace/swarm/blob/main/prompts/skills/a08-qa/gate-run.md).
    The rule that an instruction found in a log is a finding comes from the same file.
  - The low, medium and high risk class follows the `risk_class` field in
    [`02-message-protocol.md`](https://github.com/swcstudiospace/swarm/blob/main/02-message-protocol.md),
    where it decides which gates a task needs.

## Rules every playbook follows

### End states

Every playbook ends in exactly one of three states, and whoever ran it names the state:

- **Done:** the done-when check holds.
- **Blocked:** the work needs something its runner cannot get, such as an answer, an access or a
  decision. The record names what is needed and who can give it.
- **Failed:** the work cannot succeed as asked. The record says why, with the evidence.

"Stopped" is not a fourth state. Work that stops without a named state leaves the next reader
guessing whether it finished.

### The handover record

Each playbook ends by writing a handover record for the next role. It has eight parts:

- **Goal:** what the work set out to do.
- **Completed:** what was finished.
- **Remaining:** what was not.
- **Decisions:** each choice made, with its reason.
- **Artifacts:** links to what was produced, such as the story, the pull request or the report.
- **Open questions:** anything the next reader must answer.
- **Blockers:** anything stopping progress, with its owner.
- **Verification:** verified, partial or unverified, with the checks behind it.

A record can be short, but all eight parts appear, even as "none". An empty part tells the reader
it was checked. A missing part tells them nothing. The verification status matters most, because
it stops "I wrote it" from reading as "I checked it".

### Bounded loops and disagreements

Four playbooks contain a loop that can repeat without end: review in DEV-2, acceptance in PO-3,
fixing in DEV-4 and reopening in TEST-4. Each allows the first attempt plus two automatic retries.
If the third attempt also fails, the loop stops and goes up to a person, with the evidence from
every round and the options as the escalating role sees them. A loop that fails three times
usually has a cause outside the loop, such as an unclear criterion or a wrong design.

When two roles disagree about whether work passes, two rules settle it:

1. **Evidence beats opinion.** A failing test, a log or a reproduction outweighs a view without
   one.
2. **The strictest verdict wins.** If both sides have evidence, the work does not pass until the
   person who owns that decision rules on it.

## How the three roles divide the work

The product owner decides **what** gets built and in which order. The developer decides **how**
it gets built. The tester checks **whether** what was built does what was asked, and finds what
nobody asked about.

| | Product owner | Developer | Tester |
|---|---|---|---|
| Owns | The backlog and its order | The code and its design | The test strategy and suites |
| Answers | "Is this worth building?" | "How do we build it?" | "Does it work, and how does it break?" |
| Signs off | Story acceptance | Code review | Release recommendation |
| Main artifact | Ranked backlog | Merged, tested change | Test cases and defect reports |

## Product owner

### PO-1 Backlog refinement

- **Trigger:** a recurring session, usually once or twice a sprint. Also any new request from a
  stakeholder.
- **Inputs:** raw requests, user feedback, defect reports, the product roadmap.
- **Steps:**
  1. Turn each request into a story in the form "As a `<user>`, I want `<capability>` so that
     `<benefit>`."
  2. Write acceptance criteria for it. Given/When/Then form makes them directly testable.
  3. Split any story too large to finish in one sprint.
  4. Bring the top stories to the team for questions and a size estimate.
  5. Mark a story ready once it meets the team's definition of ready.
- **Output:** the top of the backlog holds about two sprints of ready stories.
- **Done when:** every story planned for the next sprint has criteria, an estimate and no open
  questions.
- **Handover:** to the team at sprint planning. Most needed: which stories are ready, and the open
  question on each one that is not.

A common definition of ready uses the INVEST checklist: each story is Independent, Negotiable,
Valuable, Estimable, Small and Testable.

### PO-2 Prioritisation

- **Trigger:** before each sprint planning, and whenever a large new item arrives.
- **Inputs:** the refined backlog, business goals, the cost of delay of each item, and known
  dependencies and risks.
- **Steps:**
  1. Score each candidate on value, urgency and risk reduction.
  2. Weigh that score against the item's size.
  3. Move dependencies ahead of the work that needs them.
  4. Bring high-risk items forward, so they fail early if they fail at all.
  5. Publish the ranked order and the reason for the top items.
- **Output:** one strictly ordered backlog. No two items share a rank.
- **Done when:** the team can pull work from the top without asking which comes first.
- **Handover:** to the team. Most needed: the reason for each top item, so a later re-rank can be
  argued with.

Two common scoring methods are MoSCoW and WSJF. MoSCoW sorts items into Must, Should, Could and
Won't. WSJF, weighted shortest job first, divides cost of delay by job size.

### PO-3 Sprint review and acceptance

- **Trigger:** a developer marks a story done, and at the sprint review at the end of each
  sprint.
- **Inputs:** the delivered story and its acceptance criteria. For a medium or high-risk story,
  also the tester's results.
- **Steps:**
  1. Check each acceptance criterion against the working software, not against a description of
     it. Record how each one was checked.
  2. Accept the story, or reject it with the criterion it fails. If the third build is rejected
     too, stop. Take it to the three amigos (SHARED-1) with the evidence from every round and the
     options you see. Its criteria are probably unclear.
  3. At the sprint review, demonstrate the accepted stories to stakeholders.
  4. Turn stakeholder feedback into new backlog items.
- **Output:** each story is accepted or returned. Feedback lands in the backlog.
- **Done when:** no story from the sprint sits unreviewed.
- **Handover:** to the developer for a returned story, naming the failed criterion and how it was
  checked. To stakeholders for the accepted ones.

### PO-4 Release planning and stakeholder updates

- **Trigger:** each release cycle, and a fixed reporting rhythm such as fortnightly.
- **Inputs:** the team's velocity, which is the story points it finishes per sprint. Also the
  ranked backlog and any external deadlines.
- **Steps:**
  1. Forecast how far down the backlog the team will reach by each date, using average velocity.
     Give a range, not a single point.
  2. Agree the release scope and date with stakeholders.
  3. Report progress against that forecast each period.
  4. Raise a scope or date trade-off as soon as the forecast slips.
- **Output:** a release plan and a regular progress report.
- **Done when:** stakeholders know what ships next and when.
- **Handover:** to stakeholders. Most needed: the forecast range and any trade-off waiting on
  their decision.

## Developer

### DEV-1 Story to design

- **Trigger:** a ready story enters sprint planning.
- **Inputs:** the story and its acceptance criteria, plus the current architecture.
- **Steps:**
  1. Ask the product owner any question the criteria leave open.
  2. Break the story into technical tasks of a day or less.
  3. Choose an approach. Record any decision that is costly to reverse, for example as an
     architecture decision record.
  4. Estimate the story with the team.
  5. Check the risk class agreed at SHARED-1 against the chosen design. Raise it with the product
     owner if the design changes it.
- **Output:** a task list, an estimate, and a design note where one was needed.
- **Done when:** any developer on the team could start the first task.
- **Handover:** to whoever takes the first task. Most needed: each costly-to-reverse decision and
  its reason.

### DEV-2 Code and review

- **Trigger:** a developer picks up a task.
- **Inputs:** the task, the design note, and the tester's test cases from TEST-1.
- **Steps:**
  1. Write a failing test for the behaviour, then the code that makes it pass.
  2. Keep each change small enough to review in one sitting.
  3. Open a pull request that says what changed, why, and how it was tested.
  4. Get at least one peer review, or two for a high-risk story. Address every comment, either
     with a fix or with a reason.
  5. If the review still blocks after two rounds of fixes, stop. Escalate to a person with the
     comments from every round and the options you see.
  6. Merge only after the review approves the latest commit.
- **Output:** a reviewed change with tests.
- **Done when:** the change is merged and the build is green.
- **Handover:** to the tester. Most needed: what changed, how it was tested, and its verification
  status.

### DEV-3 Build and merge through CI

- **Trigger:** every push to a shared branch.
- **Inputs:** the change and the CI pipeline. CI, continuous integration, means every change is
  built and tested automatically on merge.
- **Steps:**
  1. CI compiles the code and runs unit tests, static analysis and the security scan.
  2. CI runs the integration and regression suites.
  3. A failing pipeline blocks the merge.
  4. A broken main branch is fixed or reverted before any other work continues.
- **Output:** a deployable build of the main branch.
- **Done when:** the pipeline passes and the build is available to test environments.
- **Handover:** to the tester. Most needed: the build identifier and every pipeline check that
  did not run.

### DEV-4 Defect fix

- **Trigger:** a defect report assigned through triage (TEST-4).
- **Inputs:** the report with its steps to reproduce, severity and priority.
- **Steps:**
  1. Reproduce the defect. If it does not reproduce, return it with what was tried.
  2. Write a test that fails because of the defect.
  3. Trace the fault back to its root cause, not just to where it shows.
  4. Fix the cause and watch the new test pass.
  5. Check whether the same cause affects other code, and fix or report each case.
  6. If three fixes in a row fail, stop patching. Escalate to a person with the evidence from
     every round and the options you see. The design is probably wrong.
- **Output:** a fix plus a regression test that guards it.
- **Done when:** the tester confirms the fix and the regression suite passes.
- **Handover:** to the tester for retest. Most needed: the root cause, and any other code the same
  cause affects.

### DEV-5 Incident response and hotfix

- **Trigger:** a production incident, meaning a live failure that affects users.
- **Inputs:** alerts, logs, and the on-call runbook.
- **Steps:**
  1. Acknowledge the incident and name one incident lead.
  2. Restore service first. Rolling back or switching off a feature usually beats fixing forward.
  3. Ship a hotfix through a shortened but still reviewed pipeline if a rollback is not enough.
  4. Post status updates on a fixed rhythm until the incident closes.
  5. Hold a blameless post-incident review and put its actions in the backlog.
- **Output:** restored service, an incident record and follow-up items.
- **Done when:** service is normal and every follow-up has an owner.
- **Handover:** to the post-incident review. Most needed: the timeline, what restored service,
  and each follow-up with its owner.

## Tester

### TEST-1 Test design

- **Trigger:** a story is refined, before any of its code exists.
- **Inputs:** the story and its acceptance criteria.
- **Steps:**
  1. Derive one or more test cases from each acceptance criterion.
  2. Add the cases the criteria miss: boundaries, invalid input, empty states, permissions and
     error paths.
  3. Decide what to automate and at which level. Most checks belong in fast unit and API tests.
     Only a few belong in slow end-to-end UI tests.
  4. Share the cases with the developer before coding starts.
- **Output:** test cases linked to the story.
- **Done when:** every acceptance criterion has at least one test case.
- **Handover:** to the developer before coding starts. Most needed: which cases are automated,
  and at which level.

### TEST-2 Exploratory testing

- **Trigger:** a story is built and deployed to a test environment.
- **Inputs:** the build, the story, and a time-boxed charter. A charter is a one-line mission,
  for example "explore checkout with expired cards to find payment-state bugs".
- **Steps:**
  1. Write the charter first if none was given. Run a session of 60 to 90 minutes against it.
  2. Vary inputs, order, timing and user roles beyond what the scripted cases cover.
  3. Take notes as you go: what was tried, what was seen, and any question it raised.
  4. Report each defect found, using TEST-4.
- **Output:** session notes and defect reports.
- **Done when:** the charter is covered or its time box ends.
- **Handover:** to the developer and the product owner. Most needed: the defects raised, and the
  parts of the charter the session did not reach.

### TEST-3 Regression

- **Trigger:** every build, through CI.
- **Inputs:** the automated regression suite.
- **Steps:**
  1. Run the suite on each build.
  2. Investigate each failure. Sort it as a real defect, a test needing an update, or a flaky
     test.
  3. Investigate each test that was skipped or did not run. A skipped test is not a pass. It
     counts as failed until it runs.
  4. Fix or quarantine flaky tests quickly, because a suite nobody trusts gets ignored.
  5. Add a test for each fixed defect and each new feature.
  6. Prune tests that no longer check anything.
- **Output:** a suite that passes on a healthy build and fails on a broken one.
- **Done when:** the suite is green, or every red result has a defect or a fix in flight.
- **Handover:** to the developer whose change turned the suite red. Most needed: how each red
  result was sorted, and every test that did not run.

### TEST-4 Defect reporting and triage

- **Trigger:** any test, session or user report finds unexpected behaviour.
- **Inputs:** the observed behaviour and the build it was seen on.
- **Steps:**
  1. Write the report:
     - A title that names the symptom.
     - The environment and build.
     - Exact steps to reproduce.
     - Expected and actual results.
     - Evidence such as logs or screenshots.
  2. Set severity, meaning how badly it hurts the user.
  3. At triage, the product owner sets priority, meaning how soon it gets fixed.
  4. Triage assigns the defect to a developer, defers it, or closes it with a reason.
  5. Retest after the fix and close or reopen the report. If the third fix fails retest too,
     stop. Take the defect back to triage with the evidence from every round and the options you
     see. Its cause is probably not where the fixes looked.
- **Output:** a tracked defect with a decision attached.
- **Done when:** the defect is verified fixed, or closed with a stated reason.
- **Handover:** to triage, then to the assigned developer. Most needed: the steps to reproduce and
  the triage decision.

Severity and priority are separate on purpose. A typo on the home page is low severity but can be
high priority. A crash in a rarely used admin screen can be high severity and low priority.

### TEST-5 Release sign-off

- **Trigger:** a release candidate is ready.
- **Inputs:** the candidate build, the exit criteria, and the open defect list.
- **Steps:**
  1. Run the full regression suite and any release-only checks, such as performance, security
     and upgrade paths.
  2. Check the candidate against each exit criterion. A typical set is: no open critical or high
     defects, all release stories accepted, and regression green. A check that was skipped or
     could not run is not a pass. It counts as failed until it runs.
  3. For each high-risk story in the release, check its security review and rollback plan.
  4. List any known issues that ship, each with a workaround.
  5. Give a go or no-go recommendation with the evidence behind it.
- **Output:** a release report with a go or no-go recommendation.
- **Done when:** the recommendation and its evidence reach the release owner.
- **Handover:** to the release owner, the person who makes the release decision. That is
  usually the product owner. Most needed: the evidence for each exit criterion, and every check
  that did not run.

## Playbooks all three roles share

### SHARED-1 Three amigos

- **Trigger:** a story is about to become ready, during refinement.
- **Inputs:** the draft story and its criteria.
- **Who:** the product owner, one developer and one tester. That trio is the "three amigos".
- **Steps:**
  1. The product owner explains the need and the value behind it.
  2. The developer asks how it could be built and what it touches.
  3. The tester asks how it could fail and how anyone would know.
  4. Together they turn the answers into concrete examples and agree the final criteria.
  5. They agree the story's risk class, low, medium or high, as SHARED-2 defines it. The class
     decides which checks and sign-offs the story needs.
- **Output:** agreed acceptance criteria and example cases.
- **Done when:** all three would judge a finished story the same way.
- **Handover:** to the product owner, who marks the story ready. Most needed: the agreed criteria
  and the risk class.

### SHARED-2 Definition of done

- **Trigger:** anyone is about to call a story, task or change done.
- **Inputs:** the story, its risk class, and the evidence for each item below.
- **Steps:**
  1. Find the story's risk class. If it has none, treat it as high.
  2. Check each item that applies to that class, and record the evidence for it.
  3. Report the story done only if every applicable item holds. Otherwise name the item that
     failed.
- **Output:** a checked list with the evidence for each item.
- **Done when:** every applicable item holds, or the failing item is named.
- **Handover:** to the product owner for acceptance (PO-3). Most needed: the verification status
  and any item that failed.

The definition of done is a checklist the whole team owns. A story counts as finished only when
every item that applies to it holds. Which items apply depends on the story's risk class:

- **Low:** internal, small and easy to revert.
- **Medium:** changes behaviour that users see.
- **High:** touches security, money or personal data, or is hard to reverse, such as a data
  migration.

Every story needs these:

- Code is reviewed and merged.
- Unit and integration tests are written and passing.
- Regression is green.
- The handover record is written, with its verification status.

Medium and high stories also need these:

- Acceptance criteria are verified by the tester.
- Documentation and release notes are updated.

High stories also need these:

- A second reviewer has approved the code.
- A security review has passed.
- A rollback plan exists and has been tried.

A check that was skipped, or could not run, is not a pass. It counts as failed until it runs.

Product owner acceptance is not on the list, because it comes next, in PO-3.

The team revises the list at retrospectives. It should only grow stricter as the team matures.

## One story through all the playbooks

1. The product owner writes the story and its criteria (PO-1) and ranks it (PO-2).
2. The three amigos agree the final criteria (SHARED-1).
3. The tester derives test cases (TEST-1) while the developer designs (DEV-1).
4. The developer writes and reviews the code (DEV-2), and CI builds and tests it (DEV-3, TEST-3).
5. The tester explores the built feature (TEST-2) and reports defects (TEST-4).
6. The developer fixes them (DEV-4).
7. The story meets the definition of done (SHARED-2) and the product owner accepts it (PO-3).
8. At release, the tester recommends go or no-go (TEST-5), the release owner decides, and the
   product owner reports it shipped (PO-4).
