# git-commit-ai

AI-powered commit message generator for lazygit and git.

Stop typing commit messages. Stage your changes, press a key, and let AI describe
what you did.

## Features

- **9 AI providers** — OpenAI, DeepSeek, Anthropic Claude, Gemini, Grok, Qwen,
  MiniMax, GLM, Kimi
- **Three lazygit workflows**:
  - `c` — one-key: generate → commit (fastest)
  - `K` — generate → edit in editor → commit (flexible)
  - `<c-p>` — switch AI provider on the fly
- **Zero external dependencies** — pure Python 3 stdlib
- **API keys from environment variables** — set once in `.zshrc`, never touch
  config files
- **Works outside lazygit too** — standalone CLI and `prepare-commit-msg` hook
- **MIT licensed**

## Project structure

```
git-commit-ai/
├── git-commit-ai              ← main script
├── lazygit-config.yml         ← lazygit integration snippet
├── prepare-commit-msg         ← optional git hook
├── config.json                ← example configuration
├── LICENSE                    ← MIT license
└── README.md                  ← this file
```

## Installation

Clone the repo and symlink the script:

```bash
git clone https://github.com/ipunkid/git-commit-ai.git ~/git-commit-ai
ln -s ~/git-commit-ai/git-commit-ai ~/.local/bin/git-commit-ai
```

Make sure `~/.local/bin` is in your `$PATH`:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
```

### Update

```bash
cd ~/git-commit-ai && git pull
```

The symlink means the updated script is immediately available.

## Setup

### 1. Set your API key

Export the environment variable for your preferred provider:

```bash
export DEEPSEEK_API_KEY="sk-xxxxxxxxxxxxxxxx"
```

Add it to `~/.zshrc` so it persists. The script auto-detects keys by provider
name (e.g. `$OPENAI_API_KEY`, `$ANTHROPIC_API_KEY`, etc.).

### 2. Configure lazygit

lazygit reads its config from the directory printed by `lazygit --print-config-dir`:
- **Linux**: `~/.config/lazygit/config.yml`
- **macOS** (default): `~/Library/Application Support/lazygit/config.yml`

Find yours and add the snippet from `lazygit-config.yml`:

```yaml
customCommands:
  - key: 'c'
    context: 'files'
    command: 'git-commit-ai commit'
    description: 'Commit with AI message'
    loadingText: 'Generating commit message...'
    output: 'popup'
  - key: 'K'
    context: 'files'
    command: 'git-commit-ai edit'
    description: 'Generate, edit & commit'
    output: 'terminal'
  - key: '<c-p>'
    context: 'files'
    prompts:
      - type: 'menu'
        title: 'AI Provider'
        key: 'Provider'
        options:
          - name: 'deepseek'
          - name: 'openai'
          - name: 'anthropic'
          - name: 'gemini'
          - name: 'grok'
          - name: 'qwen'
          - name: 'minimax'
          - name: 'glm'
          - name: 'kimi'
    command: 'git-commit-ai set-provider {{.Form.Provider}}'
    description: 'Switch AI provider'
```

To use `~/.config/lazygit/config.yml` on macOS instead, set `XDG_CONFIG_HOME`:

```bash
export XDG_CONFIG_HOME="$HOME/.config"
```

### 3. (Optional) Global git hook

For automatic commit messages when using `git commit` in the terminal:

```bash
mkdir -p ~/.config/git/hooks
cp prepare-commit-msg ~/.config/git/hooks/
chmod +x ~/.config/git/hooks/prepare-commit-msg
git config --global core.hooksPath ~/.config/git/hooks
```

## Usage

### In lazygit

| Key | Action | Description |
|-----|--------|-------------|
| `c` | **AI commit** | Generate message from staged diff and commit immediately |
| `K` | **AI edit** | Generate, open editor to tweak, then commit |
| `<c-p>` | **Provider** | Select a different AI provider from a menu |
| `C` | _(built-in)_ | Manual commit with editor |
| `w` | _(built-in)_ | Commit without hooks |

### CLI

```bash
git-commit-ai              # Print generated message to stdout
git-commit-ai gen          # Same as above
git-commit-ai commit       # Generate + git commit
git-commit-ai edit         # Generate → editor → commit
git-commit-ai list         # Show all providers and key status
git-commit-ai set-provider <name>   # Switch provider
git-commit-ai config       # Show current configuration
git-commit-ai help         # Show help
```

### Switching providers

In lazygit, press `<c-p>` and select from the menu. Or from the command line:

```bash
git-commit-ai set-provider anthropic
git-commit-ai set-provider openai
```

The setting persists in `~/.config/git-commit-ai/config.json`.

## Supported providers

| Provider | Env variable | Default model | API format |
|----------|-------------|---------------|------------|
| OpenAI | `OPENAI_API_KEY` | `gpt-4.1-mini` | OpenAI |
| DeepSeek | `DEEPSEEK_API_KEY` | `deepseek-v4-flash` | OpenAI |
| Grok (xAI) | `GROK_API_KEY` | `grok-4.3` | OpenAI |
| Qwen (Alibaba) | `QWEN_API_KEY` | `qwen-plus` | OpenAI |
| MiniMax | `MINIMAX_API_KEY` | `minimax-m2.5` | OpenAI |
| GLM (Zhipu) | `GLM_API_KEY` | `glm-5` | OpenAI |
| Kimi (Moonshot) | `KIMI_API_KEY` | `kimi-k2.6` | OpenAI |
| Claude (Anthropic) | `ANTHROPIC_API_KEY` | `claude-sonnet-4-6` | Anthropic |
| Gemini (Google) | `GEMINI_API_KEY` | `gemini-2.5-flash` | Gemini |

API keys are resolved in this order:

1. **Environment variable** — `$DEEPSEEK_API_KEY`, `$OPENAI_API_KEY`, etc.
   (preferred)
2. **Config file** `~/.config/git-commit-ai/config.json` — supports both
   `"${ENV_VAR}"` references and plain text values

## Configuration file

First run auto-generates `~/.config/git-commit-ai/config.json`:

```json
{
  "provider": "deepseek",
  "providers": {
    "openai": {
      "api_key": "${OPENAI_API_KEY}",
      "model": "gpt-4.1-mini",
      "endpoint": "https://api.openai.com/v1"
    },
    "deepseek": {
      "api_key": "${DEEPSEEK_API_KEY}",
      "model": "deepseek-v4-flash",
      "endpoint": "https://api.deepseek.com/v1"
    }
  }
}
```

Most users never need to touch it — just set the environment variable.

## Adding a custom provider

If your provider uses the OpenAI-compatible API, add one entry to the config:

```json
{
  "providers": {
    "my-custom": {
      "api_key": "${MY_CUSTOM_API_KEY}",
      "model": "my-model",
      "endpoint": "https://api.my-custom.com/v1"
    }
  }
}
```

Then switch to it: `git-commit-ai set-provider my-custom`.

## Requirements

- Python 3.8+
- git
- An API key from one of the supported providers

## License

MIT — see [LICENSE](LICENSE).
