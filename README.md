# claude

Composite GitHub Action wrapping
[`anthropics/claude-code-action`](https://github.com/anthropics/claude-code-action)
for p6m7g8 repos. Pinned to an exact upstream release per the fleet's
third-party pinning policy.

## Usage

```yaml
- uses: p6m7g8-actions/p6-code-review@main
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

### Overriding the review prompt

```yaml
- uses: p6m7g8-actions/p6-code-review@main
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    prompt: "Review only for security defects. Be concise."
```

An override replaces the default prompt wholesale, including the verdict
contract instruction it carries. See
[Machine-readable verdict contract](#machine-readable-verdict-contract) before
overriding.

## Machine-readable verdict contract

The default prompt instructs Claude to close every review with three fixed
lines, as the last three lines of the comment:

```text
Review-Lane: code
Overall: patch is correct
Findings (total): 0
```

[`p6m7g8-actions/ai-adjudicator`](https://github.com/p6m7g8-actions/ai-adjudicator)
matches these strings literally when it reads a review comment, so the wording,
order, and position are load-bearing:

- `Review-Lane` names the lane that produced the verdict. For this action the
  value is always `code`. Sibling lanes `security` and `conventions` are
  planned, which is why the field exists now.
- `Overall` is `patch is correct`, or `patch is incorrect` when at least one
  reported finding is merge-blocking.
- `Findings (total)` is the count of findings reported, emitted even when that
  count is `0`.

All three lines appear on every review, including a clean one. They go at the
end rather than the start because `track_progress: "true"` makes
`claude-code-action` prepend its own `Claude finished ... in Xm Ys` header, so
the top of the comment is not this action's to control.

Without these lines the adjudicator has nothing to match and fails closed, which
is what a prose sign-off such as "Approving in spirit" used to produce.

## Inputs

| Input                | Default                  | Notes                                                     |
| -------------------- | ------------------------ | --------------------------------------------------------- |
| `anthropic_api_key`  | none, required           | Anthropic API key.                                        |
| `prompt`             | review prompt + contract | Prompt sent to Claude. An override replaces it wholesale. |
| `track_progress`     | `"true"`                 | Creates the tracking comment the review posts into.       |
| `use_sticky_comment` | `"true"`                 | Updates one comment in place instead of one per run.      |

### Why the comment defaults differ from upstream

Upstream defaults both `track_progress` and `use_sticky_comment` to `"false"`.
With neither set, the review runs to completion and is then discarded: there is
no tracking comment to write into, and the prompt does not itself instruct
publication. That yields a passing check with no artifact. This wrapper defaults
both to `"true"` so the review is actually published, as a single comment that
is updated in place across pushes.

Both remain configurable, so a consumer that wants upstream behavior can opt
out:

```yaml
- uses: p6m7g8-actions/p6-code-review@main
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    track_progress: "false"
    use_sticky_comment: "false"
```

`display_report` is intentionally not exposed and stays at its upstream default
of `"false"`. It writes Claude-authored content into the GitHub Step Summary and
upstream restricts it to trusted input only.
