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
