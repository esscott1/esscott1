# AI enablement: what I advise on, and where you can see it

These are the questions I help organizations work through when they move AI from pilots into production. Each one points to working code where you can check how I handle it, mostly in [ClaudeCowork](https://github.com/esscott1/ClaudeCowork). That repo holds the same job-search workflow built two ways: as managed Claude skills and as an owned LangGraph pipeline.

**Status:** ✅ in the repo today · 🟡 partly in place · 🔜 planned and documented

[← Back to profile](README.md)

---

## 1. Choosing the right amount of autonomy
- **What I advise:** use the most deterministic orchestration that does the job, and give a model control only where the work is genuinely open-ended. Model-driven orchestration costs more and is harder to test.
- **Where it shows:** ✅ [ADR 0001](https://github.com/esscott1/ClaudeCowork/blob/main/docs/adr/0001-orchestration-approach.md) compares six patterns: deterministic pipeline, state graph, orchestrator–worker, durable workflow engine, hierarchical, and swarm. It says why each was chosen or not. The [evaluate graph](https://github.com/esscott1/ClaudeCowork/blob/main/orchestration/job-search/approaches/langgraph/src/job_search_langgraph/evaluate_graph.py) has bounded, inspectable paths.

## 2. Managed vs. owned (build vs. buy)
- **What I advise:** managed platforms such as Claude skills and connectors are fast to adopt. Owning the integrations buys reliability and control, but also brings hosting, credential and monitoring work. That trade should be made on purpose, not by default.
- **Where it shows:** ✅ The same workflow is built both ways in one repo: [`skills/`](https://github.com/esscott1/ClaudeCowork/tree/main/skills/personal-productivity) is managed, and [`orchestration/`](https://github.com/esscott1/ClaudeCowork/tree/main/orchestration/job-search) is owned. The ADR lists the operational cost as a consequence rather than hiding it.

## 3. Avoiding framework lock-in
- **What I advise:** keep business rules, prompts and integrations independent of the orchestration framework, so a change of framework isn't a rewrite.
- **Where it shows:** ✅ The `core` package contains no framework code, and a [boundary test](https://github.com/esscott1/ClaudeCowork/blob/main/orchestration/job-search/core/tests/test_import_boundary.py) fails CI if one creeps in. Routing rules live in [`policy.py`](https://github.com/esscott1/ClaudeCowork/blob/main/orchestration/job-search/core/src/job_search_core/policy.py). A second approach is a new sibling folder. The model sits behind an [interface](https://github.com/esscott1/ClaudeCowork/blob/main/orchestration/job-search/core/src/job_search_core/ports.py), so moving to another provider, or to an open-weight model, means writing one adapter.

## 4. Resisting over-engineering
- **What I advise:** don't build abstractions before a second real use case exists. One finished system is worth more than four half-built ones.
- **Where it shows:** ✅ I shipped one orchestration approach with the seams for another, and deliberately left out a multi-framework comparison harness. [ADR 0001](https://github.com/esscott1/ClaudeCowork/blob/main/docs/adr/0001-orchestration-approach.md) records that decision.

## 5. Reliability engineering for agents
- **What I advise:** most agent failures happen at the edges (access, malformed output, bad handoffs), not in the model's reasoning. Design failures as known states, not crashes.
- **Where it shows:** ✅
  - Fetches retry with backoff ([JD fetcher](https://github.com/esscott1/ClaudeCowork/blob/main/orchestration/job-search/core/src/job_search_core/adapters/http_jd_fetcher.py)).
  - Model output is validated against a schema before anything downstream uses it ([LLM adapter](https://github.com/esscott1/ClaudeCowork/blob/main/orchestration/job-search/core/src/job_search_core/adapters/anthropic_llm.py)).
  - Steps are safe to re-run, and each job's progress is checkpointed so it resumes where it stopped.
  - "Couldn't fetch the job description" becomes a paused job, not a failed run.
  - The docs say plainly that no framework fixes a site that blocks automated access. Orchestration can contain that failure, but it can't remove it.

## 6. Guardrails in code, not just prompts
- **What I advise:** rules that must never be broken belong in code the model can't argue past.
- **Where it shows:** ✅ The status lifecycle is [enforced in code](https://github.com/esscott1/ClaudeCowork/blob/main/orchestration/job-search/core/src/job_search_core/domain.py). A low-scoring job can't be marked submitted, and a submitted one can't be re-scored. Every change records who made it: the pipeline, the user, or an email. [Tests](https://github.com/esscott1/ClaudeCowork/blob/main/orchestration/job-search/core/tests/test_domain_policy.py) cover the illegal transitions.

## 7. Human-in-the-loop design
- **What I advise:** choose deliberately where people approve, and make pausing and resuming cheap so oversight doesn't become a bottleneck.
- **Where it shows:** ✅ The graph pauses for a resume review and for a manually supplied job description, and resumes even in a new process. A [test](https://github.com/esscott1/ClaudeCowork/blob/main/orchestration/job-search/approaches/langgraph/tests/test_evaluate_graph.py) checks that resuming doesn't repeat the steps that already ran.

## 8. A system of record vs. agent state
- **What I advise:** long-lived business state belongs in a database you can query, not in an agent framework's internal checkpoints.
- **Where it shows:** ✅ [ADR 0002](https://github.com/esscott1/ClaudeCowork/blob/main/docs/adr/0002-tracker-is-system-of-record.md): the tracker is the system of record, and graph runs are short.

## 9. Evals as the quality gate
- **What I advise:** a prompt change is a code change. Version it, test it against labeled cases, and gate it in CI.
- **Where it shows:** 🟡 Prompts are [versioned](https://github.com/esscott1/ClaudeCowork/blob/main/orchestration/job-search/core/src/job_search_core/prompts/fit_scoring.md). There are [eight labeled cases](https://github.com/esscott1/ClaudeCowork/blob/main/orchestration/job-search/evals/job-fit/cases.json) with expected score bands, scored against a synthetic candidate. CI runs the eval harness and triggers real-model evals when a prompt changes. The first real-model baseline hasn't been committed yet.

## 10. Data privacy and least privilege
- **What I advise:** grant agents the minimum access they need, keep sensitive data out of shared locations, and don't hand out credentials just to save a step.
- **Where it shows:** ✅
  - Google access uses [read-only scopes](https://github.com/esscott1/ClaudeCowork/blob/main/orchestration/job-search/core/src/job_search_core/adapters/google_auth.py).
  - Personal data is [gitignored at any depth](https://github.com/esscott1/ClaudeCowork/blob/main/orchestration/job-search/.gitignore).
  - Evals use a [synthetic persona](https://github.com/esscott1/ClaudeCowork/blob/main/orchestration/job-search/evals/job-fit/persona_resume.md), not a real resume.
  - Repository credentials stay out of cloud AI workspaces, even where putting them there would be convenient.

## 11. Operational knowledge that prevents outages
- **What I advise:** know the platform traps that cause slow, puzzling failures before they reach production.
- **Where it shows:** ✅ The [setup guide](https://github.com/esscott1/ClaudeCowork/blob/main/docs/setup/job-search-langgraph.md) flags Google's 7-day refresh-token expiry for OAuth apps left in "Testing" status. Missed, that causes an authentication failure about once a week.

## 12. Migrating without downtime
- **What I advise:** run the new system alongside the old one in shadow mode, compare results, and cut over one component at a time.
- **Where it shows:** 🔜 The plan is in [ADR 0001](https://github.com/esscott1/ClaudeCowork/blob/main/docs/adr/0001-orchestration-approach.md) and the [roadmap](https://github.com/esscott1/ClaudeCowork/blob/main/orchestration/job-search/README.md#roadmap). The managed skills keep running as production until the owned pipeline is proven.

## 13. Lifecycle governance for agents
- **What I advise:** agent definitions need the same discipline as code: review, versioning, a registry, and checks that what's running matches what was approved.
- **Where it shows:** 🟡
  - Every agent definition change goes through a branch and a reviewed PR.
  - A `CLAUDE.md` file writes the team's conventions down so AI coding assistants follow them too.
  - A lifecycle agent manages proposal, versioning, registry and drift, and treats *released* and *installed* as separate states. It isn't in the repo yet.
  - A drift check has already caught installed agents that were out of sync with the repo, which is exactly the gap this process exists to close.

## 14. Knowing where AI tools can act
- **What I advise:** cloud and local AI agents have different permission boundaries, so place each kind of work where its access fits instead of widening permissions to make it work.
- **Where it shows:** ✅ The repo's `CLAUDE.md` splits work between local development and the hosted Claude app, and explains why.
