# claude

Composite GitHub Action wrapping
[`anthropics/claude-code-action`](https://github.com/anthropics/claude-code-action)
for p6m7g8 repos. Pinned to an exact upstream release per the fleet's
third-party pinning policy.

## Usage

```yaml
- uses: p6m7g8-actions/claude@main
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

### Overriding the review prompt

```yaml
- uses: p6m7g8-actions/claude@main
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    prompt: "Review only for security defects. Be concise."
```

## Inputs

| Input                | Default                    | Notes                                                |
| -------------------- | -------------------------- | ---------------------------------------------------- |
| `anthropic_api_key`  | none, required             | Anthropic API key.                                   |
| `prompt`             | generic code-review prompt | Prompt sent to Claude.                               |
| `track_progress`     | `"true"`                   | Creates the tracking comment the review posts into.  |
| `use_sticky_comment` | `"true"`                   | Updates one comment in place instead of one per run. |

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
- uses: p6m7g8-actions/claude@main
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    track_progress: "false"
    use_sticky_comment: "false"
```

`display_report` is intentionally not exposed and stays at its upstream default
of `"false"`. It writes Claude-authored content into the GitHub Step Summary and
upstream restricts it to trusted input only.
