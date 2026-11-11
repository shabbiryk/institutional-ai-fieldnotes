# smaller tool output, same decision?

2026-11-11 | Source review, analysis and proposed experiment. No runtime or model tests executed.

A shell command returns a wall of text. The agent needs three facts. Compressing the wall sounds sensible. The awkward question is whether the fourth fact, the one we removed, would have changed the decision.

For a small investment or operations team, the useful metric is cost per correctly completed task, including review and recovery. A shorter transcript is an input to that calculation. It is not the result.

## a concrete implementation to inspect

[tokenjuice v0.10.0's integration guide](https://github.com/vincentkoc/tokenjuice/blob/v0.10.0/docs/framework-integrations.md) describes deterministic shell-output compaction. Its native adapters require a known zero exit status before replacement; failures, unknown status, background work and truncated captures pass through. Applications using FastMCP provide the decoder and replacement callback for their own tool schema. The documented example updates both text and structured output.

That last detail matters. A UI preview, a structured tool response and the next model request are different observation points. A change at one point does not establish a change at all three.

The project's [support matrix](https://github.com/vincentkoc/tokenjuice/blob/v0.10.0/docs/adapter-support-matrix.md) states the evidence boundary explicitly: its FastMCP qualification covers a server/proxy/client path at version 4.1.4. Native host terminals and subsequent client model consumption remain outside that qualification. Other adapters have different evidence. Do not promote one host's result into a universal savings claim.

The [native contract test source](https://github.com/vincentkoc/tokenjuice/blob/v0.10.0/scripts/frameworks-python-e2e.py) checks a successful file listing, failure output, an exact read and a raw bypass. Its FastMCP assertions compare parsed text with structured content and preserve status/custom fields. I read the test source; I did not run it or independently establish that it passes.

## where the business meaning can disappear

Consider a synthetic reconciliation job that exits zero because the program completed, while reporting that three records were rejected. An agent's next task is deciding whether the reconciliation is complete. Process success and business completion are separate facts.

My inference: a compaction acceptance test should include task-critical exceptions inside otherwise successful output. A clean directory listing is useful for proving replacement. It says less about whether an analyst can make the same decision afterwards.

This is a proposed application test, not a reported tokenjuice defect. A reducer may preserve the exception, leave that command unchanged, or require the workflow to inspect raw output. Any of those could satisfy a clearly defined task contract.

## proposed experiment

Use synthetic records and a fixed question: "Did every record reconcile, and which records need review?" Compare an unchanged-output baseline with the chosen adapter, holding the model, prompt, tools and attempt budget constant.

| Fixture | Required observation |
| --- | --- |
| Successful noisy job with rejected-record IDs in the middle | Correct exception list or an explicit need to retrieve the original evidence |
| Failed job with diagnostic output | Failure status and the evidence needed for recovery remain available |
| Exact file read and explicit raw request | The documented bypass works within the configured capture boundary |
| Downstream model call | Inspect what the model actually received, not only the UI or middleware return |
| Raw evidence unavailable or capture incomplete | State that limitation; do not silently claim complete reconciliation |

Record answer correctness, missed exceptions, follow-up reads, actual model usage, elapsed time and human review effort. Use the full assigned task set as the denominator, including tasks that fail or need recovery. Report billed cost separately where provider receipts are available; do not substitute character reduction for a billing measurement.

## retain the right evidence

The [specification](https://github.com/vincentkoc/tokenjuice/blob/v0.10.0/docs/spec.md) distinguishes reducer truncation from capture limits. Raw bypass does not undo an earlier capture limit. The [README](https://github.com/vincentkoc/tokenjuice/blob/v0.10.0/README.md) also distinguishes optional raw-artifact storage from its Pi extension's automatic storage of rewritten results.

Choose retention deliberately for the application. A compact response with a usable evidence reference is different from a compact response whose original was never retained. Smaller output is welcome. A disappearing exception is a more expensive optimization.

## source record

Primary documentation and test source checked on 2026-11-11, pinned above to v0.10.0. This is original AI-assisted analysis, not a release announcement, product endorsement or measured savings claim. Upstream code and documentation retain their authorship and applicable licenses. No private transcripts or customer material were used.
