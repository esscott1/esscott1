# Eric Scott

**Enterprise Architect · AI Enablement Specialist**
Principal Consultant, [OTS Consulting](https://otsconsulting.ai)

I help organizations take AI from a promising demo to a governed, production platform. That means choosing the right architecture pattern, building it with infrastructure as code, putting guardrails and evaluations in front of production, and leaving teams with something they can run on their own. My background is 15+ years of enterprise, cloud, and security architecture across AWS, Azure, and GCP.

[LinkedIn](https://www.linkedin.com/in/ericscott411itleadership/) · [OTS Consulting](https://otsconsulting.ai)

---

## Featured solutions

The projects below are working systems, not tutorials. Each one demonstrates implementation techniques I bring to client work. The business domain is just the vehicle.

### [A&E RV Solutions](https://github.com/esscott1/AE-RV-Solutions) · live at [aervsolutions.com](https://aervsolutions.com)

A production AI chatbot, the knowledge system behind it, and the platform that runs it.

- **AI that interviews people to teach another AI.** An employee-facing assistant interviews staff one question at a time and turns what they know into structured knowledge entries (capabilities, FAQs, notes) for the public chatbot. A person approves every entry before the chatbot can use it.
- **Governed retrieval-augmented generation (RAG).** Only approved content can be indexed, and that is enforced by the storage structure itself, not by convention. Content is embedded with Titan into S3 Vectors and served through a Bedrock Knowledge Base with a relevance threshold. Every answer reports its source, and the workflow won't credit the knowledge base unless passages were actually retrieved.
- **Deterministic orchestration around the model.** An AWS Step Functions workflow controls every request. A classifier (Claude Haiku 4.5 with a forced tool call) routes hazardous, emergency, and extraction attempts to fixed replies. Anything unexpected also gets a fixed reply rather than a model answer, so the model writes an answer only on the safe path.
- **Infrastructure as code with guardrails before merge.** Terraform is organized as one module per feature. Every infrastructure PR gets a `terraform plan` as a required check, `main` is protected, and `prevent_destroy` covers critical resources. CI authenticates to AWS through GitHub OIDC, so no cloud keys are stored, and pull-request plans run under a read-only role.
- **Operating AI in production.** A safety evaluation suite fails if any hazardous prompt is routed to an answer. The chatbot has a kill switch that works in seconds from three independent paths. Quotas, throttling, and budget alarms limit usage and cost. Transcripts collect minimal personal data, and admin dashboards show usage, cost, and step-level latency.

### [Monte Carlo Retirement Simulator](https://github.com/esscott1/MonteCarlo) · live at [montecarlo.otsconsulting.ai](https://montecarlo.otsconsulting.ai)

A working simulator that also serves as a reference implementation of an AI-driven software delivery loop.

- **From request to pull request with AI agents.** A user's change request becomes a Jira story. A headless Claude Code agent running in GitHub Actions then implements it on a branch and opens a pull request. Today a person approves at triage and again at PR review; the same pipeline could run fully automated wherever the risk allows. It shows how short the path can be from wanting a change to having it delivered.
- **Tool use with server-side enforcement.** The application calls Claude through a single forced tool call to write the story. The server checks the output against the required shape byte for byte and replaces anything that doesn't match, so injected text can't reshape the request. The model never touches Jira; a deterministic client does the writing.
- **MCP through a Claude Code Skill.** A custom skill uses the Atlassian MCP server to link each commit and its Jira issue in both directions and move the issue through its workflow.
- **Specialized review subagents.** Code-review and structure-review subagents run with read-only tools. One of their checks is model right-sizing: whether each AI call uses the cheapest model tier that can do the job reliably.
- **Tested engine, automated deployment.** A .NET simulation engine with unit tests is deployed to Azure App Service through a path-filtered GitHub Actions pipeline.

---

## What working with me looks like

I help teams make the architecture decisions that decide whether AI holds up in production, and I show the trade-offs rather than just recommending a tool. [ClaudeCowork](https://github.com/esscott1/ClaudeCowork) builds the same workflow two ways, as managed Claude skills and as an owned LangGraph pipeline, to make those trade-offs concrete.

- **The right amount of autonomy.** Deterministic orchestration first, with model control only where it pays for itself, and the choice written down in an [architecture decision record](https://github.com/esscott1/ClaudeCowork/blob/main/docs/adr/0001-orchestration-approach.md).
- **Managed vs. owned.** When a managed platform is enough, and when owning the integrations is worth the operating cost.
- **No framework lock-in.** Business rules, prompts and integrations sit outside the orchestration framework, and a CI test enforces that boundary.
- **Reliability where agents actually fail.** Retries, schema-validated model output, resumable checkpoints, and failures handled as states rather than crashes.
- **Guardrails in code, with people in the loop.** Rules the model can't override, and approval points that pause and resume cheaply.
- **Least privilege by default.** Read-only scopes, no personal data in repos, and no credentials handed to AI tools just for convenience.
- **Migration without downtime.** Run the new system in shadow beside the old one and cut over a component at a time.

The full list, with each point linked to the code that shows it: [AI enablement playbook](ai-enablement-playbook.md).

---

Open to Enterprise Architect and AI enablement roles. The best way to reach me is [LinkedIn](https://www.linkedin.com/in/ericscott411itleadership/).
