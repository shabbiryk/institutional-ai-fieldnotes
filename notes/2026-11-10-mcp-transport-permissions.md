# same tool, different permission boundary

2026-11-10 | Reading note and proposed acceptance tests. No runtime tests executed.

A research assistant works on a laptop. The team puts it behind a shared service. The tools have the same names, so it is tempting to assume the permission model travelled with them. That is the bit worth checking before anyone connects the investment committee folder.

## what the primary sources establish

The [MCP authorization specification, revision 2026-07-28](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization), defines an authorization flow for HTTP transports. Authorization support is optional. For STDIO, it directs implementations toward credentials from the environment rather than this HTTP flow. An MCP connection alone therefore does not tell us which authorization boundary exists.

A concrete implementation detail makes this practical: [FastMCP's authorization documentation](https://github.com/PrefectHQ/fastmcp/blob/b4bb0f04fb7da9a19755158bc01ef91c93b38587/docs/servers/authorization.mdx) says its component authorization checks are skipped under STDIO. Access there is controlled by the local execution environment. With an AuthProvider configured for HTTP, valid tokens are required before component checks evaluate scopes and claims. Those statements describe FastMCP's documented behavior, not every MCP implementation.

These are different trust arrangements. Neither observation, by itself, proves a vulnerability.

## why a small investment team should care

Consider a hypothetical tool called `read_research_note`. On one analyst's machine, a local process may use credentials supplied by that environment. In a shared deployment, the team needs to identify the caller and decide which underlying documents that caller can read.

My inference: proving that a local tool works is weak evidence that a shared service preserves each analyst's permissions. A tool-level scope such as `research:read` also needs a separate answer to the business question: read whose research? Document ownership, fund boundaries and restricted deal material belong in the acceptance criteria.

Before deployment, write down three things: whose identity reaches the data source, which policy checks the specific document, and where a denial is recorded. If any answer is "the model knows", the design still has homework.

## a proposed test card

Use synthetic documents, two fictional funds and two test users. Keep real client material out of this exercise.

| Case | Expected outcome |
| --- | --- |
| HTTP request without a valid token, AuthProvider enabled | Rejected before the tool runs |
| Valid identity without the required tool scope | Tool access denied |
| Correct scope, document assigned to the other fictional fund | Application or data-source policy denies the document |
| Same tool invoked over STDIO | Local execution and data-source controls enforce the intended boundary; do not count on FastMCP's component OAuth checks |
| User loses document access after an earlier successful read | A fresh request is denied under the application's documented revocation policy |

The last two rows test the deployment design, not a promise made by the framework. Capture transport, test identity, document ID, policy decision and whether the tool body ran. Keep tokens and document contents out of the evidence log.

## what would change this assessment

A deliberately single-user local setup may have an adequate boundary at the operating system and data source. A shared service might already enforce document permissions downstream. In either case, demonstrate that boundary rather than adding OAuth-shaped decoration.

The useful demo is two people asking for the same document and getting different, correct answers. Less stage magic. More reason to let the team use it on Monday.

## source record

Sources checked on 2026-11-10. FastMCP documentation is pinned above to commit `b4bb0f04fb7da9a19755158bc01ef91c93b38587`; the MCP link identifies the specification revision. This is original analysis with linked sources, not reproduced documentation or an executed benchmark. Upstream documentation and code retain their authorship and applicable licenses.
