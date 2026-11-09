# institutional ai fieldnotes

agents are getting very good at doing things. i'm interested in whether a small investment team can actually trust them to do the right things.

this is a working notebook on AI agents, institutional workflows and onchain infrastructure. less launch-day confetti. more: what changed, who would use it, where does it break, and what would i test next?

By [Shabbir Khan](https://github.com/shabbiryk), building Recallr and exploring the overlap between finance and software.

## start here

- [First note: an agent that can pay still needs a mandate](#2026-11-09--an-agent-that-can-pay-still-needs-a-mandate)
- [Khan-Proxy](https://github.com/shabbiryk/Khan-Proxy) — a separate project on this account.
- [My GitHub](https://github.com/shabbiryk)

## what belongs here

| Track | The question |
| --- | --- |
| Institutional AI | Can a small team use it without hiring another operations team? |
| Memory and retrieval | Can the answer be traced to the right source, with the right access? |
| Agent infrastructure | What happens when a tool fails, a model changes, or a budget runs out? |
| Onchain infrastructure | Who can authorize an action, settle it and reconcile it? |

## how this notebook runs

Research collection and writing are AI-assisted and maintained by a scheduled agent. Entries distinguish documented facts from analysis and proposed tests. A note is not a claim that I personally ran a benchmark, spoke to a team or deployed the product.

The workflow checks primary sources daily and publishes only when there is a useful, verifiable addition. Quiet days are fine. No filler commits, copied articles or random forks to keep a square green.

Sources stay linked. Third-party code and documentation belong to their authors; their licenses still apply. Corrections and grounded counterexamples are welcome through issues. Please leave private client information out of them.

## 2026-11-09 — an agent that can pay still needs a mandate

**Reading note / analysis. No new release or benchmark claimed.**

The interesting institutional question is what happens *before* a payment, and what the team can explain afterwards.

**What the sources say.** [x402](https://x402.org/) describes an HTTP-native payment flow: a server requests payment with HTTP 402, and the client can pay and retry. [MCP's architecture documentation](https://modelcontextprotocol.io/docs/learn/architecture) describes a way for AI applications to connect to servers that expose tools and context. Its scope does not dictate how an application uses its model or manages that context.

**The implication.** Connecting a tool and paying for it solve useful pieces of the workflow. A small investment team still needs a mandate around those pieces: which vendor, which budget, which data, and whose approval. A successful transaction is not, by itself, evidence of an authorized business action.

**What I would test.** Give a research agent a synthetic $50 allowance, two approved data vendors and one deliberately unapproved vendor. Ask it to buy a report. Repeat with a duplicate request, an expired permission and an exhausted budget. Keep this in a sandbox: no client data or real funds.

The result worth measuring is simple: did the agent complete the permitted task, stop the prohibited one, and leave an understandable receipt? Log the source, request ID, authorization decision and cost. Treat the model's explanation as something to inspect, rather than the control itself.

**Where this view might be wrong.** A human approving every tiny action would make the workflow miserable. The useful design may be a bounded mandate with escalation only when something falls outside it. That needs testing against real workflows.

The next demo I'd like to see: the agent politely refusing to spend money it was never allowed to spend. slightly less cinematic. probably more useful.

---

*Public research notes, not investment advice or an endorsement of the linked projects.*
