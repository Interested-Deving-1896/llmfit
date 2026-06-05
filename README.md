[update-readmes]   Mode: rewrite — migrating to template structure...
# llmfit

[![Built with Ona](https://ona.com/build-with-ona.svg)](https://app.ona.com/#https://github.com/Interested-Deving-1896/llmfit)

<!-- AI:start:what-it-does -->
_Description pending._
<!-- AI:end:what-it-does -->

## Architecture

<!-- AI:start:architecture -->
_Architecture documentation pending._
<!-- AI:end:architecture -->

## Install


### Windows
```sh
scoop install llmfit
```

If Scoop is not installed, follow the [Scoop installation guide](https://scoop.sh/).

### macOS / Linux

#### Homebrew
```sh
brew install llmfit
```

#### Quick install
```sh
curl -fsSL https://llmfit.axjns.dev/install.sh | sh
```

Downloads the latest release binary from GitHub and installs it to `/usr/local/bin` (or `~/.local/bin` if no sudo).

**Install to `~/.local/bin` without sudo:**
```sh
curl -fsSL https://llmfit.axjns.dev/install.sh | sh -s -- --local
```

### From source
```sh
git clone https://github.com/AlexsJones/llmfit.git
cd llmfit
cargo build --release
# binary is at target/release/llmfit
```

---

## Usage


### TUI (default)

```sh
llmfit
```

Launches the interactive terminal UI. Your system specs (CPU, RAM, GPU name, VRAM, backend) are shown at the top. Models are listed in a scrollable table sorted by composite score. Each row shows the model's score, estimated tok/s, best quantization for your hardware, run mode, memory usage, and use-case category.

| Key | Action |
|---|---|
| `Up` / `Down` or `j` / `k` | Navigate models |
| `/` | Enter search mode (partial match on name, provider, params, use case) |
| `Esc` or `Enter` | Exit search mode |
| `Ctrl-U` | Clear search |
| `f` | Cycle fit filter: All, Runnable, Perfect, Good, Marginal |
| `a` | Cycle availability filter: All, GGUF Avail, Installed |
| `s` | Cycle sort column: Score, Params, Mem%, Ctx, Date, Use Case |
| `t` | Cycle color theme (saved automatically) |
| `p` | Open Plan mode for selected model (hardware planning) |
| `P` | Open provider filter popup |
| `i` | Toggle installed-first sorting (any detected runtime provider) |
| `d` | Download selected model (provider picker when multiple are available) |
| `r` | Refresh installed models from runtime providers |
| `1`-`9` | Toggle provider visibility |
| `Enter` | Toggle detail view for selected model |
| `PgUp` / `PgDn` | Scroll by 10 |
| `g` / `G` | Jump to top / bottom |
| `q` | Quit |

### TUI Plan mode (`p`)

Plan mode inverts normal fit analysis: instead of asking "what fits my hardware?", it estimates "what hardware is needed for this model config?".

Use `p` on a selected row, then:

| Key | Action |
|---|---|
| `Tab` / `j` / `k` | Move between editable fields (Context, Quant, Target TPS) |
| `Left` / `Right` | Move cursor in current field |
| Type | Edit current field |
| `Backspace` / `Delete` | Remove characters |
| `Ctrl-U` | Clear current field |
| `Esc` or `q` | Exit Plan mode |

Plan mode shows estimates for:
- minimum and recommended VRAM/RAM/CPU cores
- feasible run paths (GPU, CPU offload, CPU-only)
- upgrade deltas to reach better fit targets

### Themes

Press `t` to cycle through 6 built-in color themes. Your selection is saved automatically to `~/.config/llmfit/theme` and restored on next launch.

| Theme | Description |
|---|---|
| **Default** | Original llmfit colors |
| **Dracula** | Dark purple background with pastel accents |
| **Solarized** | Ethan Schoonover's Solarized Dark palette |
| **Nord** | Arctic, cool blue-gray tones |
| **Monokai** | Monokai Pro warm syntax colors |
| **Gruvbox** | Retro groove palette with warm earth tones |

### CLI mode

Use `--cli` or any subcommand to get classic table output:

```sh
# Table of all models ranked by fit
llmfit --cli

# Only perfectly fitting models, top 5
llmfit fit --perfect -n 5

# Show detected system specs
llmfit system

# List all models in the database
llmfit list

# Search by name, provider, or size
llmfit search "llama 8b"

# Detailed view of a single model
llmfit info "Mistral-7B"

# Top 5 recommendations (JSON, for agent/script consumption)
llmfit recommend --json --limit 5

# Recommendations filtered by use case
llmfit recommend --json --use-case coding --limit 3

# Plan required hardware for a specific model configuration
llmfit plan "Qwen/Qwen3-4B-MLX-4bit" --context 8192
llmfit plan "Qwen/Qwen3-4B-MLX-4bit" --context 8192 --quant mlx-4bit
llmfit plan "Qwen/Qwen3-4B-MLX-4bit" --context 8192 --target-tps 25 --json

# Run as a node-level REST API (for cluster schedulers / aggregators)
llmfit serve --host 0.0.0.0 --port 8787
```

### REST API (`llmfit serve`)

`llmfit serve` starts an HTTP API that exposes the same fit/scoring data used by TUI/CLI, including filtering and top-model selection for a node.

```sh
# Liveness
curl http://localhost:8787/health

# Node hardware info
curl http://localhost:8787/api/v1/system

# Full fit list with filters
curl "http://localhost:8787/api/v1/models?min_fit=marginal&runtime=llamacpp&sort=score&limit=20"

# Key scheduling endpoint: top runnable models for this node
curl "http://localhost:8787/api/v1/models/top?limit=5&min_fit=good&use_case=coding"

# Search by model name/provider text
curl "http://localhost:8787/api/v1/models/Mistral?runtime=any"
```

Supported query params for `models`/`models/top`:

- `limit` (or `n`): max number of rows returned
- `perfect`: `true|false` (forces perfect-only when `true`)
- `min_fit`: `perfect|good|marginal|too_tight`
- `runtime`: `any|mlx|llamacpp`
- `use_case`: `general|coding|reasoning|chat|multimodal|embedding`
- `provider`: provider text filter (substring)
- `search`: free-text filter across name/provider/size/use-case
- `sort`: `score|tps|params|mem|ctx|date|use_case`
- `include_too_tight`: include non-runnable rows (default `false` on `/top`, `true` on `/models`)
- `max_context`: per-request context cap for memory estimation

Validate API behavior locally:

```sh
# spawn server automatically and run endpoint/schema/filter assertions
python3 scripts/test_api.py --spawn

# or test an already-running server
python3 scripts/test_api.py --base-url http://127.0.0.1:8787
```

### GPU memory override

GPU VRAM autodetection can fail on some systems (e.g. broken `nvidia-smi`, VMs, passthrough setups). Use `--memory` to manually specify your GPU's VRAM:

```sh
# Override with 32 GB VRAM
llmfit --memory=32G

# Megabytes also work (32000 MB ≈ 31.25 GB)
llmfit --memory=32000M

# Works with all modes: TUI, CLI, and subcommands
llmfit --memory=24G --cli
llmfit --memory=24G fit --perfect -n 5
llmfit --memory=24G system
llmfit --memory=24G info "Llama-3.1-70B"
llmfit --memory=24G recommend --json
```

Accepted suffixes: `G`/`GB`/`GiB` (gigabytes), `M`/`MB`/`MiB` (megabytes), `T`/`TB`/`TiB` (terabytes). Case-insensitive. If no GPU was detected, the override creates a synthetic GPU entry so models are scored for GPU inference.

### Context-length cap for estimation

Use `--max-context` to cap context length used for memory estimation (without changing each model's advertised maximum context):

```sh
# Estimate memory fit at 4K context
llmfit --max-context 4096 --cli

# Works with subcommands
llmfit --max-context 8192 fit --perfect -n 5
llmfit --max-context 16384 recommend --json --limit 5
```

If `--max-context` is not set, llmfit will use `OLLAMA_CONTEXT_LENGTH` when available.

### JSON output

Add `--json` to any subcommand for machine-readable output:

```sh
llmfit --json system     # Hardware specs as JSON
llmfit --json fit -n 10  # Top 10 fits as JSON
llmfit recommend --json  # Top 5 recommendations (JSON is default for recommend)
llmfit plan "Qwen/Qwen2.5-Coder-0.5B-Instruct" --context 8192 --json
```

`plan` JSON includes stable fields for:
- request (`context`, `quantization`, `target_tps`)
- estimated minimum/recommended hardware
- per-path feasibility (`gpu`, `cpu_offload`, `cpu_only`)
- upgrade deltas

---

## Configuration

<!-- Document configuration options here. This section is yours — the AI will not modify it. -->

## CI

<!-- AI:start:ci -->
_CI documentation pending._
<!-- AI:end:ci -->

## Mirror chain

<!-- AI:start:mirror-chain -->
This repo is maintained in [`Interested-Deving-1896/llmfit`](https://github.com/Interested-Deving-1896/llmfit) and mirrored through:

```
Interested-Deving-1896/llmfit  ──►  OpenOS-Project-OSP/llmfit  ──►  OpenOS-Project-Ecosystem-OOC/llmfit
```

Changes flow downstream automatically via the hourly mirror chain in
[`fork-sync-all`](https://github.com/Interested-Deving-1896/fork-sync-all).
Direct commits to OSP or OOC are detected and opened as PRs back to `Interested-Deving-1896`.
<!-- AI:end:mirror-chain -->

## Contributors

<!-- AI:start:contributors -->
_Contributors pending._
<!-- AI:end:contributors -->

## Origins

<!-- AI:start:origins -->
_Original project — no upstream fork._
<!-- AI:end:origins -->

## Resources

<!-- AI:start:resources -->
_No additional resource files found._
<!-- AI:end:resources -->

## License

<!-- AI:start:license -->
[MIT](https://github.com/Interested-Deving-1896/llmfit/blob/main/LICENSE) © 2026 [Interested-Deving-1896](https://github.com/Interested-Deving-1896)
<!-- AI:end:license -->
