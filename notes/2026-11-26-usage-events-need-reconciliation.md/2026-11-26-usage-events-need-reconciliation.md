# usage metrics are a control input, not a receipt

2026-11-26 | Public-source review, local synthetic test and proposed buyer acceptance test. No FastMCP integration was executed.

A research agent calls three tools. One is cheap, one is expensive, and one is allowed only for a particular fund. The team wants to know who used what, whether a limit was crossed, and whether the log can explain the answer later.

## what changed upstream

The current [FastMCP release notes](https://github.com/PrefectHQ/fastmcp/releases) list UsageMiddleware for per-user and per-tool usage metrics. The same release notes describe an experimental Events extension with poll and push delivery, plus security and authentication fixes. These are documented capabilities. They do not establish how a particular deployment stores events, handles replay, or makes a spending decision.

The [FastMCP middleware documentation](https://gofastmcp.com/servers/middleware) describes middleware as a place to observe or modify requests and responses as they pass through a server. Usage data can feed policy inputs and operations dashboards. It is not automatically a complete financial or compliance ledger.

## the institutional distinction

My inference is that a per-user usage counter and an event stream solve different problems. A counter can answer a budget question quickly. An event can preserve the facts needed to investigate a disputed call. A budget decision needs both policy state and a durable record of what the policy saw.

For a small investment or operations team, the minimum record for a billable or sensitive tool call should include a stable request ID, user or service identity, tool name and version, policy decision, measured quantity, provider receipt if one exists, and event-processing status. A model-generated explanation is useful context. It is not the source of truth for the charge.

The failure cases are ordinary. A push event can arrive twice. A worker can see page two before page one. A usage counter can increment while the provider call times out. A retry can succeed after the first request completed. If the system cannot distinguish those cases, the dashboard can look precise while reconciliation is wrong.

## a local synthetic test

This fixture models an out-of-order event and a duplicate. The consumer deduplicates by request ID and advances its checkpoint only after the missing sequence arrives.

```python
seen = set()
usage = {}
next_sequence = 1
stream = [
    {"sequence": 2, "request_id": "r2", "user": "analyst-b", "tool": "premium_search", "units": 1},
    {"sequence": 1, "request_id": "r1", "user": "analyst-a", "tool": "cheap_search", "units": 1},
    {"sequence": 1, "request_id": "r1", "user": "analyst-a", "tool": "cheap_search", "units": 1},
]
for event in stream:
    if event["sequence"] != next_sequence or event["request_id"] in seen:
        continue
    seen.add(event["request_id"])
    key = (event["user"], event["tool"])
    usage[key] = usage.get(key, 0) + event["units"]
    next_sequence += 1
assert usage == {("analyst-a", "cheap_search"): 1}
assert next_sequence == 2
stream.append({"sequence": 2, "request_id": "r2", "user": "analyst-b", "tool": "premium_search", "units": 1})
for event in stream:
    if event["sequence"] != next_sequence or event["request_id"] in seen:
        continue
    seen.add(event["request_id"])
    key = (event["user"], event["tool"])
    usage[key] = usage.get(key, 0) + event["units"]
    next_sequence += 1
assert usage[("analyst-b", "premium_search")] == 1
assert next_sequence == 3
print("Duplicate ignored, late event reconciled.")
```

I ran this with Python 3 and the standard library. It passed. The test establishes only the behavior of this fixture. It does not test FastMCP's event extension, persistence, provider billing or concurrent workers.

## a buyer's test card

Use two synthetic users, two tools and a mock provider that can return success, timeout, duplicate delivery and a late event.

| Case | Acceptance question |
| --- | --- |
| Duplicate event for a completed request | Is usage counted once and is the duplicate visible? |
| Provider timeout followed by a successful retry | Can the team reconcile provider receipts with the internal request ID? |
| Event arrives out of order | Does the checkpoint remain at the last complete sequence? |
| User reaches a per-tool budget | Does the next request stop, and is the denial recorded? |
| Identity loses access mid-run | Does a fresh policy check deny the next call? |
| Event transport is unavailable | Does the system fail closed for sensitive tools or mark usage unknown? |

Record request ID, identity, tool version, policy decision, provider receipt, event sequence and reconciliation status. Keep the fixture synthetic. Do not connect it to client documents or real funds.

Usage numbers are useful. Reconciled usage is what survives an uncomfortable question from finance.

## source record

Primary release and documentation pages checked 2026-11-26. Original analysis and synthetic fixture developed with Codex assistance under the notebook's disclosed workflow. No private material, third-party code or vendor integration result was used. Upstream code and documentation retain their authorship and applicable licenses.
