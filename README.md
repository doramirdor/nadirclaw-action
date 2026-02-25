# NadirClaw Action

Route your CI's LLM calls through [NadirClaw](https://github.com/doramirdor/NadirClaw) for automatic cost savings. Simple prompts go to cheap models, complex ones go to premium. Your AI-powered workflows get cheaper without getting worse.

## Quick Start

```yaml
- uses: doramirdor/nadirclaw-action@v1
  with:
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
    google-api-key: ${{ secrets.GOOGLE_API_KEY }}
```

That's it. `OPENAI_BASE_URL` is now set to `http://localhost:8856/v1`. Any tool that reads this env var (most OpenAI SDKs, aider, etc.) will route through NadirClaw automatically.

## How It Works

1. Installs NadirClaw in the runner
2. Starts the proxy on localhost
3. Sets `OPENAI_BASE_URL` for all subsequent steps
4. Every LLM call gets classified and routed to the right model

Simple prompts ("what does this error mean?") go to Gemini Flash. Complex prompts ("refactor this module") go to Claude Sonnet. You save 50-70% on typical CI workloads.

## Inputs

| Input | Default | Description |
|---|---|---|
| `version` | `latest` | NadirClaw version to install |
| `port` | `8856` | Proxy port |
| `premium-model` | `claude-sonnet-4-20250514` | Model for complex prompts |
| `cheap-model` | `gemini-2.5-flash` | Model for simple prompts |
| `reasoning-model` | `o3` | Model for chain-of-thought |
| `openai-api-key` | - | OpenAI API key |
| `anthropic-api-key` | - | Anthropic API key |
| `google-api-key` | - | Google API key |
| `startup-timeout` | `30` | Seconds to wait for proxy |

## Outputs

| Output | Description |
|---|---|
| `base-url` | The `OPENAI_BASE_URL` pointing to NadirClaw |

## Example: AI Code Review

```yaml
name: AI Code Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - uses: doramirdor/nadirclaw-action@v1
        with:
          premium-model: claude-sonnet-4-20250514
          cheap-model: gemini-2.5-flash
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
          google-api-key: ${{ secrets.GOOGLE_API_KEY }}

      - name: Run AI review
        run: |
          pip install aider-chat
          git diff origin/main...HEAD | aider --message "Review this diff for bugs and improvements." --yes
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
```

## Example: AI Test Generation

```yaml
name: Generate Tests
on:
  workflow_dispatch:
    inputs:
      target:
        description: 'File to generate tests for'
        required: true

jobs:
  generate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - uses: doramirdor/nadirclaw-action@v1
        with:
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
          google-api-key: ${{ secrets.GOOGLE_API_KEY }}

      - name: Generate tests
        run: |
          pip install aider-chat
          aider --message "Write unit tests for ${{ github.event.inputs.target }}" --yes

      - uses: peter-evans/create-pull-request@v6
        with:
          title: "test: AI-generated tests for ${{ github.event.inputs.target }}"
          branch: ai-tests/${{ github.run_id }}
```

## Why?

A typical CI workflow with AI (code review, test gen, docs) makes 20-50 LLM calls. Most of those calls are simple. Without routing, you're paying premium prices for every call. NadirClaw routes the simple ones to cheap models and reserves premium for the calls that actually need it.

No data leaves the runner except to the LLM providers you configure. NadirClaw runs entirely in-process.

## License

MIT
