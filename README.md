# AI Commit Summarizer

Summarize a range of commits by passing their titles, bodies, and diffs through GitHub Models along with any prompt you'd like. Use it in workflows to generate flexible summaries, changelogs, or other prose that reflects the exact tone and structure you request.

## How It Works

- Gathers commits between two refs (optionally including the start commit).
- Collects each commit's title, body, and diff (diffs can be scoped to specific paths).
- Builds a prompt that combines your custom instructions with the raw commit details.
- Sends the prompt to the GitHub Models inference API and returns a string summary.

## Quick Start

```yaml
name: Commit Summary

on:
  workflow_dispatch:
    inputs:
      from:
        description: Oldest commit (inclusive) to summarize
        required: true
      to:
        description: Most recent commit (defaults to the dispatch SHA)
        required: false

jobs:
  summarize:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      models: read
    steps:
      - uses: actions/checkout@v4
      - name: Generate commit summary
        id: summary
        uses: josiahsrc/ai-commit-summarizer@v1
        with:
          from: ${{ github.event.inputs.from }}
          to: ${{ github.event.inputs.to || github.sha }}
          prompt: |
            Summarize the commits for the engineering team. List major user-facing changes first,
            followed by any noteworthy technical details, with a friendly closing sentence.
          extra_instructions: Use bullet points with emoji headers.
      - name: Save summary
        run: |
          echo "## Commit Summary" >> "$GITHUB_STEP_SUMMARY"
          echo "${{ steps.summary.outputs.summary }}" >> "$GITHUB_STEP_SUMMARY"
```

## Inputs

| Input | Description | Default |
| --- | --- | --- |
| `from` | Starting commit reference to summarize. | — |
| `to` | Ending commit reference (inclusive). | `HEAD` |
| `include_start_commit` | Include the starting commit itself. | `true` |
| `max_diff_chars` | Maximum diff characters per commit before truncation. | `6000` |
| `model` | GitHub Models identifier to use. | `openai/gpt-4o-mini` |
| `temperature` | Temperature passed to the model (`0-2`). | `0.2` |
| `max_output_tokens` | Maximum tokens for the model response. | `800` |
| `system_prompt` | Override the default system prompt. | *(empty)* |
| `prompt` | Override the default summary instructions. Commit details are appended automatically. | *(empty)* |
| `extra_instructions` | Additional instructions you want the model to honor. | *(empty)* |
| `paths` | Optional newline-separated list of paths/globs to limit diffs. | *(empty)* |

Combine `prompt` and `extra_instructions` to steer tone, format, or structure (e.g., "Summarize in a two-paragraph executive memo" or "Return a concise list of bullet points with action items").

## Outputs

| Output | Description |
| --- | --- |
| `summary` | Markdown summary returned by the model. |

## Requirements

- Workflows must include `models: read` permission (called via `GITHUB_TOKEN`).
- No extra secrets are required because the default `GITHUB_TOKEN` is used to call the GitHub Models API.
