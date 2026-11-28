# Three citations can still be one source

28 November 2026 | Public-source analysis and an offline Python fixture.

A small investment team asks its research agent to check whether a fund extended its subscription window. The memo cites the manager's notice, a distributor's copy and a newsletter that paraphrased the same notice. The footnotes look busy. The analyst still has one underlying record.

Keep the links. Each has a use. Count corroboration separately, at the level of the specific claim. A distributor who confirms a different cutoff for its own channel is reporting a separate observation. A distributor forwarding the original notice is another route to the same observation.

## What the sources establish

[LLM's fragment documentation](https://llm.datasette.io/en/stable/fragments.html#viewing-fragments-in-your-logs) shows how to recover the content supplied to a prompt through its SHA-256 identifier and logs. A content hash can identify an exact captured excerpt. Two paraphrases can have different hashes while relying on the same underlying record.

[W3C PROV](https://www.w3.org/TR/prov-dm/#component2) describes derivation and more specific relationships, including quotation and primary source. An analyst's evidence table can borrow that distinction without adopting the whole model. [The Qorx source-check exercise](https://github.com/bbrainfuckk/qorx-jev-source-check) asks whether the cited excerpt contains enough evidence to answer the question.

## Try it with a diligence memo

Pick one material claim. For each citation, record the excerpt, its version or retrieval date, the underlying record if known, and whether the excerpt supports, contradicts or leaves the claim open. For a source that combines its own reporting and a forwarded notice, label the individual observations separately. Keep unknown origins visible for someone to check.

The fixture below uses a synthetic desk rule that calls for two separately collected records when marking a claim corroborated. A single authoritative source can be sufficient for many tasks, such as checking a deadline in a signed notice. Choose the rule before counting the links.

```python
from dataclasses import dataclass
from typing import Optional


@dataclass(frozen=True)
class Citation:
    label: str
    reviewed_origin: Optional[str]
    finding: str


def assess(citations: list[Citation]) -> tuple[str, int]:
    supporting_origins = {
        citation.reviewed_origin
        for citation in citations
        if citation.finding == "supports" and citation.reviewed_origin
    }
    count = len(supporting_origins)

    if any(citation.finding == "contradicts" for citation in citations):
        return "review conflicting evidence", count
    if count >= 2:
        return "corroborated under desk rule", count
    return "corroboration pending", count


notice = Citation("manager notice", "manager-2026-11-26", "supports")
mirror = Citation("distributor copy", notice.reviewed_origin, "supports")
rewrite = Citation("newsletter paraphrase", notice.reviewed_origin, "supports")
unknown = Citation("summary of unclear origin", None, "supports")
vague = Citation("administrator mention", "admin-call-01", "inconclusive")
checked = Citation("administrator's own cutoff record", "admin-ops-07", "supports")
conflict = Citation("same channel shows the old cutoff", None, "contradicts")

copies = [notice, mirror, rewrite]
assert assess(copies) == ("corroboration pending", 1)
assert assess(copies + [unknown, vague]) == ("corroboration pending", 1)
assert assess(copies + [checked]) == ("corroborated under desk rule", 2)
assert assess(copies + [checked, conflict]) == ("review conflicting evidence", 2)
print("4 cases passed")
```

## What ran

The four assertions passed under Python 3.9 with synthetic records and no network calls. The script checks what the decision rule does with reviewed inputs. It leaves source identification, independence and interpretation of the excerpts to the reviewer. In a live workflow, first check that comparable evidence refers to the same fund, share class, distribution channel and effective date. A discrepancy may be a scope difference worth recording.

A practical display for the memo is “three citations, one underlying record,” with the sources available one click away. Before the approval call, where would you go to check the claim independently?

Public sources examined 28 November 2026. Original analysis and synthetic fixture prepared with Codex assistance. Referenced works retain their authorship and licenses.
