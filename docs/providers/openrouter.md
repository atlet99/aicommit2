# <a href="https://openrouter.ai/" target="_blank">OpenRouter</a>

## 📌 Important Note

**Before configuring, please review:**

- [Configuration Guide](../../README.md#configuration) - How to configure providers
- [General Settings](../../README.md#general-settings) - Common settings applicable to all providers

## Example Configuration

### `config.ini`

If you prefer a config file over `aicommit2 config set`, this is a good starting point:

```ini
logging=true
generate=1
locale=en
type=conventional
maxTokens=4096
temperature=0.2
topP=0.9

[OPENROUTER]
key=YOUR_OPENROUTER_API_KEY
model=stepfun/step-3.5-flash:free
url=https://openrouter.ai
path=/api/v1/chat/completions
systemPromptPath=prompts/aicommit_prompt.txt
responseFormat='{"type":"json_object"}'
provider='{"allow_fallbacks":true,"require_parameters":false}'
reasoning={}
```

If `systemPromptPath` is relative, it is resolved from the config file directory.
For example, the snippet above expects a file like `prompts/aicommit_prompt.txt`
next to the config file.

### Basic Setup

```sh
aicommit2 config set OPENROUTER.key="your-api-key"
aicommit2 config set OPENROUTER.model="openrouter/auto"
aicommit2 config set OPENROUTER.responseFormat='{"type":"json_object"}'
aicommit2 config set OPENROUTER.provider='{"allow_fallbacks":true,"require_parameters":false}'
```

### Specific Model

```sh
aicommit2 config set OPENROUTER.key="your-api-key" \
    OPENROUTER.model="anthropic/claude" \
    OPENROUTER.temperature=0.7 \
    OPENROUTER.maxTokens=4000 \
    OPENROUTER.locale="en" \
    OPENROUTER.generate=3 \
    OPENROUTER.topP=0.9
```

## Settings

| Setting | Description      | Default |
| ------- | ---------------- | ------- |
| `key`   | API key          | - |
| `model` | Model to use     | `openrouter/auto` |
| `url`   | API endpoint URL | `https://openrouter.ai` |
| `path`  | API path         | `/api/v1/chat/completions` |
| `responseFormat` | OpenRouter `response_format` payload object | - |
| `provider` | OpenRouter routing controls payload object | - |
| `reasoning` | OpenRouter reasoning payload object | - |
| `topP` | Nucleus sampling probability | `0.9` |
| `temperature` | Sampling temperature | `0.7` |
| `maxTokens` | Maximum tokens to generate | `4096` |

## Notes

- The provider uses the OpenAI chat-completions contract behind the scenes.
- OpenRouter-specific routing headers are sent automatically.
- If you want more routing control, set `OPENROUTER.provider` directly in config.
- If you want structured output, set `OPENROUTER.responseFormat` to a JSON object.
- If you want reasoning-specific controls, set `OPENROUTER.reasoning` to a JSON object.
- `aicommit2 doctor` also checks OpenRouter catalog reachability and can suggest which `OPENROUTER.responseFormat` or `OPENROUTER.reasoning` options to revisit when the chosen model does not advertise those capabilities.

## Configuration

#### OPENROUTER.key

Your OpenRouter API key. Set it via the `key` property (or the environment variable `OPENROUTER_API_KEY`).

```sh
aicommit2 config set OPENROUTER.key="your-api-key"
```

#### OPENROUTER.model

Default: `openrouter/auto`

Use a model slug that OpenRouter exposes, such as:

- `openrouter/auto`
- `anthropic/claude`
- any other model slug listed in the OpenRouter catalog

```sh
aicommit2 config set OPENROUTER.model="anthropic/claude"
```

#### OPENROUTER.url

Default: `https://openrouter.ai`

The service base URL.

```sh
aicommit2 config set OPENROUTER.url="https://openrouter.ai"
```

#### OPENROUTER.path

Default: `/api/v1/chat/completions`

The chat completions path used by the OpenAI SDK.

#### OPENROUTER.topP

Controls nucleus sampling. Accepts a decimal between 0 and 1 (default `0.9`).

```sh
aicommit2 config set OPENROUTER.topP=0.9
```

#### OPENROUTER.temperature

Controls randomness of the model output. Accepts a decimal between 0 and 2 (default `0.7`).

```sh
aicommit2 config set OPENROUTER.temperature=0.7
```

#### OPENROUTER.maxTokens

Maximum number of tokens the model may generate (default `4096`).

```sh
aicommit2 config set OPENROUTER.maxTokens=4096
```
