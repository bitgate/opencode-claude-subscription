# OpenCode with a Claude Pro or Max subscription

OpenCode stopped bundling its Claude Pro/Max login plugins in version 1.3.0, so the way back is a custom provider: drop this repo's [`opencode.json`](opencode.json) into your project, hand OpenCode the `p2a_` key from [Plan2API](https://plan2api.com/integrations/opencode) as `PLAN2API_API_KEY`, and pick `plan2api/claude/claude-sonnet-5` in `/models`. Claude Sonnet 5 and Opus 5.5 then run on your subscription with tool calls, streaming and a 1M-token context, with no Anthropic API key.

The same provider carries your ChatGPT Plus or Pro plan (`plan2api/openai/gpt-6.1-sol`), Google AI Pro (`plan2api/gemini/gemini-4-argon-high`) and Kimi Code (`plan2api/kimi/k3`), and it works on OpenCode 1.x and OpenCode 2.

**[Get a free `p2a_` key](https://plan2api.com/connect?utm_source=github&utm_medium=readme&utm_campaign=opencode-claude-subscription)** | [OpenCode guide on plan2api.com](https://plan2api.com/integrations/opencode) | [API reference](https://plan2api.com/docs)

## 1. Turn your Claude subscription into an API key

1. Open [plan2api.com/connect](https://plan2api.com/connect?utm_source=github&utm_medium=readme&utm_campaign=opencode-claude-subscription) and sign up with Google.
2. Connect Claude by signing in with your Claude account on Anthropic's own sign-in page. Pro, Max 5x and Max 20x all work.
3. Copy the `p2a_` key you get after connecting. It is shown once; create more on the Keys page (`plan2api.com/dashboard/keys`).

ChatGPT Plus or Pro connects with a one-time device code on OpenAI's own page, Google AI Pro with your Google account, and Kimi Code with its API key.

## 2. Install OpenCode

```bash
curl -fsSL https://opencode.ai/v2/install | bash
```

Or `brew install anomalyco/tap/opencode-v2`, or `npm install -g @opencode/cli`.

## 3. Add the Plan2API provider

Put [`opencode.json`](opencode.json) in your project root, or in `~/.config/opencode/opencode.json` to use it in every project:

```bash
mkdir -p ~/.config/opencode
curl -fsSL https://raw.githubusercontent.com/bitgate/opencode-claude-subscription/master/opencode.json \
  -o ~/.config/opencode/opencode.json
```

Already have a global config? Merge the `provider.plan2api` block into it instead of overwriting.

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "plan2api": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Plan2API",
      "options": {
        "baseURL": "https://api.plan2api.com/v1",
        "apiKey": "{env:PLAN2API_API_KEY}"
      },
      "models": {
        "claude/claude-sonnet-5": {
          "name": "Claude Sonnet 5",
          "limit": { "context": 1000000, "output": 128000 }
        },
        "claude/claude-opus-5-5": {
          "name": "Claude Opus 5.5",
          "limit": { "context": 1000000, "output": 128000 }
        }
      }
    }
  },
  "model": "plan2api/claude/claude-sonnet-5",
  "small_model": "plan2api/claude/claude-haiku-4-5"
}
```

The file in the repo adds Claude Haiku 4.5, GPT-6.1 Sol, Gemini 4 Argon, Gemini 3.8 Flash and Kimi K3.

## 4. Give OpenCode your key

OpenCode 2 runs model requests in a background service, so set the key on the service:

```bash
opencode service set env PLAN2API_API_KEY p2a_your_key
```

For a one-off private session instead: `PLAN2API_API_KEY=p2a_your_key opencode --standalone`. OpenCode 1.x reads it from your shell:

```bash
export PLAN2API_API_KEY="p2a_your_key"
```

## 5. Pick the model

Run `opencode`, type `/models` and choose **Claude Sonnet 5** under **Plan2API**. The config already makes it the default and sends session titles to Claude Haiku 4.5 through `small_model`, so titles don't burn your Sonnet allowance.

## Native OpenCode 2 config

OpenCode 2 reads the `provider` / `npm` / `options` shape above as-is, so the same file works on both versions. If you prefer OpenCode 2's native shape (`providers`, `package`, `settings`), use [`v2-native/opencode.jsonc`](v2-native/opencode.jsonc) instead. It uses the `@opencode/ai/providers/openai-compatible` package, reads the key from `PLAN2API_API_KEY` through `env`, and maps short model refs such as `plan2api/claude-sonnet-5` to Plan2API ids with `modelID`. Keep one format per file, and don't point OpenCode 1.x at the native V2 file.

## Models

| Model ref | Runs on | Context window |
| --- | --- | --- |
| `plan2api/claude/claude-sonnet-5` | Claude Pro or Max | 1,000,000 |
| `plan2api/claude/claude-opus-5-5` | Claude Pro or Max | 1,000,000 |
| `plan2api/claude/claude-haiku-4-5` | Claude Pro or Max | 200,000 |
| `plan2api/openai/gpt-6.1-sol` | ChatGPT Plus or Pro (Codex) | 500,000 (Codex default) |
| `plan2api/gemini/gemini-4-argon-high` | Google AI Pro or Ultra | 1,048,576 |
| `plan2api/gemini/gemini-3.8-flash-high` | Google AI Pro or Ultra | 1,048,576 |
| `plan2api/kimi/k3` | Kimi Code membership | 1,048,576 |

Want another Plan2API model? Add its id under `models`. `GET https://api.plan2api.com/v1/models` lists all of them with `context_length`, and the [API reference](https://plan2api.com/docs) has the full table.

## What works

- Streaming and function tool calls on every model, so OpenCode's tools and subagents run as normal.
- `reasoning_effort` (or `reasoning.effort`): `none` to `max` on Claude and Gemini, `low`, `high` or `max` on Kimi, passed through on Codex.
- Image input on Claude, Gemini and Kimi models.
- JSON mode on Codex, Gemini and Kimi. Claude answers a JSON `response_format` with `400 unsupported_parameter`.
- Chat Completions only: use `@ai-sdk/openai-compatible` (or `@opencode/ai/providers/openai-compatible` on V2), because `@ai-sdk/openai` calls `/v1/responses`, which Plan2API does not serve.

## Troubleshooting

| What you see | Why | Fix |
| --- | --- | --- |
| `401` `invalid_api_key` | `PLAN2API_API_KEY` isn't set where OpenCode runs requests | OpenCode 2: `opencode service set env PLAN2API_API_KEY p2a_...`; OpenCode 1.x: export it before launching |
| `404` `not_found` | The provider uses `@ai-sdk/openai` (Responses API), or `baseURL` is missing `/v1` | Use `@ai-sdk/openai-compatible` and `https://api.plan2api.com/v1` |
| `Model unavailable: plan2api/...` (OpenCode 2) | The model ref doesn't match a key in the `models` map | Use a key from the config, such as `plan2api/claude/claude-sonnet-5` |
| `403` `no_credentials` | The model belongs to a provider you haven't connected | Connect it at plan2api.com, or pick a `claude/` model |
| `400` `model_not_found` | The id sent to Plan2API is wrong | In `opencode.json` the model keys are the Plan2API ids; in the native V2 file, check `modelID` |
| `400` `unsupported_parameter` | JSON `response_format` on a Claude model | Use Codex, Gemini or Kimi for JSON mode |
| `429` `subscription_quota_exhausted` | Your Claude plan hit its 5-hour or weekly limit | Wait for the `Retry-After` time, or connect a second Claude account so requests fail over automatically |
| `429` `rate` or `inflight` | Parallel subagents beyond the Free plan's 1 request per second (4 at once) | Move to Pro: 10 per second, bursts of 600, 16 at once |
| Long sessions never compact | The model has no `limit.context` | Add `limit` like the Claude entries |

## FAQ

### Does OpenCode support Claude Pro or Max subscriptions?

Not out of the box anymore: OpenCode dropped its bundled Claude Pro/Max plugins in 1.3.0. Add Plan2API as a custom provider with this repo's `opencode.json` and your Claude subscription powers OpenCode again.

### Can I use Claude Code OAuth in OpenCode?

You don't need it. Plan2API signs in with your Claude account on Anthropic's own page once and gives OpenCode a standard `p2a_` key, so OpenCode only ever sees an OpenAI-compatible provider.

### Can I use ChatGPT Plus in OpenCode?

Yes. OpenCode has its own Sign in with ChatGPT, and through Plan2API, GPT also sits in the same provider as Claude: connect ChatGPT and pick `plan2api/openai/gpt-6.1-sol`.

### Does this work with OpenCode 2?

Yes. OpenCode 2 reads the `provider` block in `opencode.json` as-is, and `v2-native/opencode.jsonc` has the native V2 version. Set the key with `opencode service set env PLAN2API_API_KEY p2a_your_key`.

### Where does opencode.json go?

In your project root (or `.opencode/opencode.json`) for one project, or `~/.config/opencode/opencode.json` for all of them.

### Why @ai-sdk/openai-compatible and not @ai-sdk/openai?

`@ai-sdk/openai` talks to OpenAI's Responses API at `/v1/responses`, which Plan2API doesn't serve. `@ai-sdk/openai-compatible` uses `/v1/chat/completions`.

### What does it cost?

Plan2API Free is $0 with one connected subscription and no card. Pro is $12 a month for 3 subscriptions and rate limits that keep up with parallel subagents; Scale is $29 a month for 25. See [pricing](https://plan2api.com/pricing).

## More guides

- [OpenCode + Plan2API guide](https://plan2api.com/integrations/opencode)
- [OpenCode vs Claude Code](https://plan2api.com/opencode-vs-claude-code)
- [Use your Claude subscription as an API](https://plan2api.com/claude-subscription-api)
- [Claude Pro and Max usage limits](https://plan2api.com/claude-usage-limits)
- [Claude Code Router + Plan2API](https://plan2api.com/integrations/claude-code-router)
- [Plan2API API reference](https://plan2api.com/docs)

Same setup, other clients:

- [Cline, Roo Code and Kilo Code with a Claude subscription](https://github.com/bitgate/cline-claude-subscription)
- [SillyTavern with a Claude subscription](https://github.com/bitgate/sillytavern-claude-subscription)
- [Janitor AI Claude proxy](https://github.com/bitgate/janitor-ai-claude-proxy)
- [Open WebUI with a ChatGPT subscription](https://github.com/bitgate/open-webui-chatgpt-subscription)
