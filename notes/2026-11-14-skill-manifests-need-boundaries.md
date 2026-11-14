# Skill manifests are a control surface

2026-11-14 | Source review, analysis and proposed experiment. Research began November 13; sources rechecked November 14. No runtime test executed.

A skill catalog looks like a menu. In a small investment team, that menu might reveal which workflows exist before anyone opens a document.

## What the release says

[FastMCP v4.2.0a5](https://github.com/PrefectHQ/fastmcp/releases/tag/v4.2.0a5) is a prerelease. It adds SkillManifestProvider: discovery can use frontmatter plus file digests and sizes, with content loaded on read. SkillCatalogProvider builds on this and avoids rehashing every file per request. The release also lists frontmatter verification and checks on skill limits before directory or remote reads.

That describes upstream behavior, not a security assessment of a particular installation.

## The institutional question

My inference: metadata deserves an access policy of its own. Hiding a skill body does not necessarily hide a sensitive workflow name or summary. A catalog called 'Restricted acquisition diligence' can say rather more than its owner intended. The example is synthetic.

For a team building a shared research assistant, ask three separate questions:

- Who can discover the skill and its metadata?
- Who can read its supporting files?
- Who can execute the tools it describes?

Those permissions should be explicit. A digest is useful evidence about content identity; it does not establish that a caller has authority to act.

## A test card worth running

This is a proposed application test, not a claim that FastMCP lacks these controls.

Create a synthetic two-user catalog with one shared skill, one restricted skill and one remotely stored skill. For each user, record the visible names and summaries, attempted reads, authorization decisions and bytes fetched.

Then try a changed file, an oversized file and a mounted provider. Check how the application detects content changes, when limits are applied, and whether discovery through the mount preserves the intended policy. Define the expected outcome before running each case. Version pin the provider and client so the result is reproducible.

Success means permitted work still works, restricted metadata stays restricted, and denials are explainable. Record discovery latency too, but do not let a faster list operation be the only acceptance criterion.

The catalog can be small. The surprise should be smaller.

## Limits and provenance

No package was installed, exploit reproduced or benchmark measured for this note. These are original analysis and proposed tests based on the linked primary release. Upstream code and documentation retain their authorship and licenses.

Research and writing are AI-assisted by Codex under this notebook's disclosed scheduled workflow.
