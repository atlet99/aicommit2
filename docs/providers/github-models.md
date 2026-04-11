# <a href="https://github.com/marketplace/models" target="_blank">GitHub Models</a>

## 📌 Important Note

**Before configuring, please review:**

- [Configuration Guide](../../README.md#configuration) - How to configure providers
- [General Settings](../../README.md#general-settings) - Common settings applicable to all providers

> [!WARNING]
> **Deprecated:** The **GitHub Models** provider is deprecated in favor of the **Copilot SDK** provider. The `gh models list` command is no longer supported, and the REST API access will be disabled in upcoming releases. Users should migrate to the Copilot SDK documented in [Copilot SDK](./copilot-sdk.md).

**GitHub Models is separate from GitHub Copilot UI features.**

- **GitHub Copilot**: IDE/chat coding assistant experience
- **GitHub Models**: REST API access to hosted models via `https://models.github.ai`

_aicommit2_ integrates with the official **GitHub Models API**, not private Copilot endpoints.

## 🚀 Quick Setup

### Option 1: Automatic login (recommended)

```sh
aicommit2 github-login
```

This command:

1. Authenticates with **GitHub CLI** (`gh`).
2. Stores the token in `GITHUB_MODELS.key`.
3. Verifies access to the GitHub Models API.

### Option 2: Manual token setup

1. Create a **GitHub Personal Access Token (PAT)**.
2. Grant the permission `models: read`.
3. Configure the token:

```sh
aicommit2 github-login --token github_pat_xxxxxxxxxxxxxxxxxxxx
# or
aicommit2 config set GITHUB_MODELS.key="github_pat_xxxxxxxxxxxxxxxxxxxx"
```

## PAT Generation (Important)

Create a token from GitHub settings:

- Fine-grained token (recommended): [github.com/settings/personal-access-tokens/new](https://github.com/settings/personal-access-tokens/new)
- Classic token page: [github.com/settings/tokens](https://github.com/settings/tokens)

You can find additional information at the link: [GitHub personal access tokens documentation](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens).

Token must have `models: read` access, otherwise requests typically fail with `403`.

`models: read` is for GitHub Models API usage. It is different from Copilot CLI/SDK token requirements (`Copilot Requests`).

Supported token prefixes in _aicommit2_ include:

- `github_pat_...`
- `ghp_...`
- `gho_...`
- `ghu_...`
- `ghs_...`
- `ghr_...`

## Prerequisites (for `github-login`)

- [GitHub CLI](https://cli.github.com/) installed
- Browser access for web login

Install GitHub CLI:

```sh
# macOS
brew install gh

# Windows (winget)
winget install GitHub.cli

# Windows (choco)
choco install gh

# Linux (Debian/Ubuntu)
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install gh
```

## Model ID Format (Required)

`GITHUB_MODELS.model` must follow the **`publisher/model`** pattern, e.g.:

```text
openai/gpt-4o-mini
openai/gpt-5
openai/gpt-5-chat
meta/llama-3.3-70b-instruct
```

Short IDs such as `gpt-4o-mini` are **not** accepted by the GitHub Models provider in *aicommit2*.

## Copilot Plan Limits (Important)

Model availability is tied to your **GitHub Copilot** subscription:

* **Copilot Free** – limited to a few thousand inline suggestions and a small number of premium requests per month. Model choice is restricted.
* **Paid / Student plans** – unlimited suggestions and chat for the models included in the plan (e.g., `gpt-5`, `gpt-4o`). Premium request quotas apply to higher‑tier models.

GitHub may change model access without notice, so the list of usable models can differ between the UI and the `models.github.ai` API. If you encounter errors such as:

* `400 unavailable_model`
* `403` (permission issue)
* `429` (rate‑limit)

verify your token and plan, or run a live discovery using the Copilot SDK (see the [Copilot SDK documentation](./copilot-sdk.md)).

## Validated Snapshot (April 2, 2026)

The following results were validated with a real `GITHUB_TOKEN` on `2026-04-02` using:

- `X-GitHub-Api-Version: 2026-03-10`
- catalog probe + throttled dry-run commit generation

### Probe result (`catalog` text models)

- Total probed models: `41`
- `200 OK`: `29`
- `400`: `7` (`unavailable_model` / `unknown_model`)
- `403`: `5`

### Dry-run commit generation result (for `200 OK` models)

- Total dry-run models: `29`
- `OK (generated)`: `23`
- `FAIL (no_valid_commit_message)`: `6`

### Models that generated valid commit messages (`OK`)

- OpenAI: `openai/gpt-4.1`, `openai/gpt-4.1-mini`, `openai/gpt-4.1-nano`, `openai/gpt-4o`, `openai/gpt-4o-mini`
- Cohere: `cohere/cohere-command-a`, `cohere/cohere-command-r-08-2024`, `cohere/cohere-command-r-plus-08-2024`
- DeepSeek: `deepseek/deepseek-v3-0324`
- Meta: `meta/llama-3.2-11b-vision-instruct`, `meta/llama-3.2-90b-vision-instruct`, `meta/llama-3.3-70b-instruct`, `meta/llama-4-maverick-17b-128e-instruct-fp8`, `meta/llama-4-scout-17b-16e-instruct`, `meta/meta-llama-3.1-405b-instruct`, `meta/meta-llama-3.1-8b-instruct`
- Mistral AI: `mistral-ai/codestral-2501`, `mistral-ai/ministral-3b`, `mistral-ai/mistral-medium-2505`, `mistral-ai/mistral-small-2503`
- xAI: `xai/grok-3`
- Microsoft: `microsoft/phi-4`, `microsoft/phi-4-multimodal-instruct`

### Models that responded but failed commit JSON validation (`FAIL`)

- `deepseek/deepseek-r1`
- `deepseek/deepseek-r1-0528`
- `xai/grok-3-mini`
- `microsoft/phi-4-mini-instruct`
- `microsoft/phi-4-mini-reasoning`
- `microsoft/phi-4-reasoning`

### Important interpretation

- This snapshot is **token-specific** and **time-specific**.
- A model can appear in `catalog/models` and still fail at inference time for your token (`unavailable_model`, `403`, etc.).
- A model can return `200` but still fail _aicommit2_ commit parsing if it does not return valid commit JSON.
- This is currently the most reliable way to find working models for your account: **probe + throttled dry-run**.

### Reproduce safely (low rate-limit risk)

1. Fetch catalog with `X-GitHub-Api-Version: 2026-03-10`.
2. Probe only text-generation models with a delay between requests (for example, `sleep 2`).
3. Keep only models with `HTTP 200`.
4. Run commit `--dry-run` checks sequentially with throttling (for example, `sleep 12`) and retry on `429`.

Example catalog command:

```bash
curl -sSL \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  https://models.github.ai/catalog/models > catalog.json
```

## Usage Examples

### Basic usage

```sh
aicommit2 github-login
aicommit2 config set GITHUB_MODELS.model="openai/gpt-5"

git add .
aicommit2
```

### Advanced configuration

```sh
aicommit2 config set \
  GITHUB_MODELS.key="github_pat_xxxxxxxxxxxxxxxxxxxx" \
  GITHUB_MODELS.model="openai/gpt-5" \
  GITHUB_MODELS.temperature=0.7 \
  GITHUB_MODELS.maxTokens=1024 \
  GITHUB_MODELS.locale="en" \
  GITHUB_MODELS.generate=3 \
  GITHUB_MODELS.topP=0.95
```

## Settings

| Setting | Description | Default |
| ------- | ----------- | ------- |
| `key` | GitHub token | - |
| `model` | GitHub Models model ID (`publisher/model`) | `openai/gpt-4o-mini` |

## Discover Available Models

**Note:** The `gh models list` command is **deprecated** and no longer maintained. The REST API endpoint for model discovery will be disabled in favor of the **Copilot SDK** workflow. For the current list of supported models, refer to the [Copilot SDK documentation](./copilot-sdk.md).

If you still need to query the legacy catalog (while it remains available), you can use the following REST call, but be aware it will be removed in future releases:

```bash
curl -L \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  https://models.github.ai/catalog/models
```

See the official docs: <https://docs.github.com/en/rest/models/inference>.

## API Version

_aicommit2_ uses GitHub API version header:

```text
X-GitHub-Api-Version: 2026-03-10
```

## Troubleshooting

### 401 Unauthorized

Token is invalid/expired:

```sh
aicommit2 github-login
```

### 403 Forbidden

Token lacks `models: read` permission.

### 404 Model not found

Model unavailable for your account or incorrect model ID. Check `gh models list`.

### 422 Validation error

Model ID format is invalid. Use `publisher/model` (example: `openai/gpt-4o-mini`).

## References

- [GitHub Models quickstart](https://docs.github.com/en/enterprise-cloud@latest/github-models/quickstart)
- [GitHub Models inference REST API](https://docs.github.com/en/rest/models/inference)
- [GitHub Copilot plans](https://github.com/features/copilot/plans)
- [GitHub Copilot billing & requests](https://docs.github.com/en/copilot/concepts/billing/copilot-requests)
- [GitHub REST API breaking changes (2026-03-10)](https://docs.github.com/en/rest/about-the-rest-api/breaking-changes?apiVersion=2026-03-10)
- [gh-models extension](https://github.com/github/gh-models)
