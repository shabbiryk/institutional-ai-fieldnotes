# A citation check is only as fresh as its last complete page

2026-11-24 | Public-source analysis and a local synthetic test.

A research desk keeps a short watchlist of sources used in its decision memos. On Tuesday the update feed returns two pages. Page one contains an ordinary addition. Page two contains a withdrawal of the item behind last week's recommendation. If the job saves the new checkpoint after page one and then loses its connection, Wednesday can start with a clean dashboard and an unreviewed withdrawal.

## What the sources establish

The [LRRK Agent Desk worked example](https://lrrk.ai/resources/agent-tools) demonstrates a source-specific change feed. Its documented client keeps the original source version throughout pagination and advances only after the last exact page when the checkpoint is ready. It treats an unavailable source and an unknown checkpoint as separate states. Its examples distinguish corrections from withdrawals. The [FastMCP discussion](https://github.com/PrefectHQ/fastmcp/discussions/6606) includes a contributor's reported check of an unrecognized checkpoint.

This is a documented example from one provider. For a familiar comparison, [HTTP Semantics, section 13.1.2](https://www.rfc-editor.org/rfc/rfc9110.html#section-13.1.2) lets a conditional GET validate a stored representation against the selected resource. A change feed also needs to account for the records between two versions. A successful response from either kind of endpoint says only what its own contract establishes.

## A small acceptance test

Use a synthetic watchlist with one cited record. Give your importer a two-page update batch, fail it between pages, and ask what the analyst would see. The old checkpoint should survive the interruption. After a complete retry, the analyst should see the withdrawal as a review item linked to the decision that originally cited it. Keep the old decision record so a reviewer can reconstruct what the desk knew at the time.

This Python fixture uses an in-memory SQLite database. It models an importer that receives a complete batch from a validated adapter and records its checkpoint and review items in one transaction.

```python
import sqlite3

db = sqlite3.connect(":memory:")
db.executescript("""
CREATE TABLE state (version TEXT NOT NULL);
INSERT INTO state VALUES ('v1');
CREATE TABLE review (record TEXT NOT NULL, kind TEXT NOT NULL);
""")

def apply_update(pages, target):
    with db:
        for records, final in pages:
            db.executemany("INSERT INTO review VALUES (?, ?)", records)
            if final:
                db.execute("UPDATE state SET version = ?", (target,))
                return
        raise ValueError("update batch ended before the final page")

def snapshot():
    version = db.execute("SELECT version FROM state").fetchone()[0]
    records = db.execute("SELECT record, kind FROM review ORDER BY rowid")
    return version, records.fetchall()

page_one = ([("note-22", "added")], False)
page_two = ([("note-04", "withdrawn")], True)

try:
    apply_update([page_one], "v2")
except ValueError:
    assert snapshot() == ("v1", [])
else:
    raise AssertionError("an incomplete batch was accepted")

apply_update([page_one, page_two], "v2")
assert snapshot() == (
    "v2", [("note-22", "added"), ("note-04", "withdrawn")]
)
print("Incomplete batch rolled back. Complete retry retained the withdrawal.")
```

## Result and limits

Both assertions passed under Python 3 with the standard-library SQLite module. The run used synthetic records and made no network calls. It establishes rollback and final commit in this fixture. It leaves page-token validation, concurrent workers, duplicate delivery, and the analyst's review interface for an integration test. Fetch and validate pages before opening a production database transaction so a slow source does not hold writes open.

For the operational screen, give each source its own last complete check, any gap in comparison history, and unresolved corrections or withdrawals. The question worth asking at the Monday meeting is simple: which decisions have new evidence someone still needs to read?

Public sources checked 24 November 2026. Original analysis and fixture developed with Codex assistance. Source material retains its authorship and applicable licenses. The example uses no third-party code or private material.
