# Ambassador Local: recipes, building and releases

For people who change the catalog or build the app from source. Official, signed releases are published by the
original author at [MangelZabalaDevelop/ambassador-local](https://github.com/MangelZabalaDevelop/ambassador-local/releases);
Dell builds and signs no installers.

## Adding recipes

Everything lives in [`recipes/catalog.json`](../recipes/catalog.json). It points to
[`recipes/catalog.schema.json`](../recipes/catalog.schema.json), so editors like VS Code autocomplete every field and
flag mistakes while you type.

- `engine`: the pinned llama.cpp version and its files per OS/architecture. When you change it, copy that version's
  chat page for vLLM and TensorFold with `scripts/vendor-llama-ui.sh` (`cargo test` checks the versions match), and
  check that **Open chat** still opens without asking for the API key: the app writes the key where llama.cpp's chat
  page keeps its settings (`LlamaUi.config` in its local storage, see [`src-tauri/src/chat.rs`](../src-tauri/src/chat.rs)).
- `hardware`: the program's workstations. `gpu_match` is used to detect them with `nvidia-smi`, and only the
  workstation detected on the machine can be used. `"available": false` shows it as *Coming soon* (not tested yet).
  `"units": 2` is for linked systems (two GB10), and `"image"` reuses another workstation's photo. Product images go in
  `public/hardware/<id>.webp`.
- `families`: the lab cards (name, series, description, color). Logos go in `src/assets/labs/<id>.svg`.
- `models`: each model with its GGUF files and one `recipe` per hardware id. A model without a recipe for a workstation
  is hidden on that workstation.

What users see in the model table (no jargon, all icons):

| Field | Where | Shown as |
|---|---|---|
| `size_class` | model | Size: `small` (4–14B), `medium` (24–32B), `large` (~70B, up to 96 GB), `xlarge` (two linked GB10) |
| `intelligence` (1–5) | model | Light bulbs |
| `best_for` (1–3 labels) | model | Tags, e.g. `["Chat", "Coding"]` |
| `speed` (1–5) | recipe | Lightning bolts, per workstation |
| `tokens_per_sec` | recipe | Average tokens/s you measured on that workstation (optional; shows "Not measured yet") |
| `ctx_size` | recipe | Memory: books (8K = 1 … over 64K = 5) and approximate words |

Allowed recipe fields:

| Field | llama-server | Default |
|---|---|---|
| `speed` (1–5) | — (only for the table) | required |
| `tokens_per_sec` | — (only for the table) | not measured |
| `ctx_size` (512–262144) | `-c` | required |
| `quant` + `files` | `-m` (other GGUF for this workstation) | the model's `quant` and `files` |
| `mtp.draft_n_max` (1–16) | `--spec-type draft-mtp --spec-draft-n-max` (MTP speculative decoding) | off |
| `mtp.draft` | `-md` (separate `mtp-*.gguf`; leave out when the MTP head is inside the model) | — |
| `vllm` | Linux only: serve with vLLM instead of llama.cpp (see below) | off |
| `tensorfold` | GB10 ×2 only: serve with TensorFold instead of llama.cpp (see below) | off |
| `tuned` | — (the **Miguel Tuned** label: a recipe tuned by hand) | `false` |
| `recommended` | — (listed first in its lab, with a crown) | `false` |
| `gpu_layers` | `-ngl` | 999 |
| `flash_attn` | `-fa on/off` | `true` |
| `mmap` | `--no-mmap` when `false` | `true` |
| `temp`, `top_p`, `top_k`, `min_p` | sampling | llama.cpp defaults |

The app always runs one slot (`-np 1`): one user gets the whole context and the full speed.

### vLLM (NVFP4)

Some tuned recipes are NVFP4 models served by vLLM. A Linux recipe can carry a `vllm` block instead of GGUF files:

| Field | Meaning |
|---|---|
| `image` | Container image pinned by digest (`repo@sha256:…`) |
| `image_id` | The image's `Id`, as `docker image inspect` shows it for the same image; accepted when the digest is missing (an image copied with `docker save/load` keeps only its id) |
| `repo`, `revision` | Hugging Face weights pinned to a commit; each GB10 needs them in its Hugging Face cache |
| `size` | Bytes of those weights, per GB10 |
| `recipe` | Folder name under [`recipes/vllm/`](../recipes/vllm) with `run.sh` and `stop.sh`; required on a single workstation |

`ctx_size` is `--max-model-len` here and can go up to 1M. Those models have `"files": []`.

Run starts these models like any other. First the app checks the machine (on two GB10: the second GB10 on the
ConnectX cable, passwordless SSH to it, and everything below on both): Docker with the NVIDIA runtime, the pinned
image, complete weights, and a free GPU. Then it writes the recipe folder's scripts, which it carries inside, to
`~/.cache/ambassador-local/<engine>/<recipe>/recipe/` and runs `run.sh` with the port, the API key file and the model
id. Stop, or closing the app, runs `stop.sh`. A new recipe folder must also be listed in
[`src-tauri/src/recipe.rs`](../src-tauri/src/recipe.rs); `cargo test` checks it.

The app doesn't set up the cluster: the cluster setup in NVIDIA Sync does.

### TensorFold on two GB10

Some models run best on an engine with no public image. A `tensorfold` block points to a folder in
[`recipes/tensorfold/`](../recipes/tensorfold) with a Dockerfile (every source pinned by commit or digest) and the
scripts that build the image, download and check the weights on both GB10 (`prepare.sh`) and serve the model
(`run.sh`):

| Field | Meaning |
|---|---|
| `image` | Local tag that the folder's `build.sh` gives the image (`name:tag`) |
| `engine_commit` | TensorFold commit the image is built from; the cluster check reads it from the image label |
| `recipe` | Folder name under `recipes/tensorfold/` |
| `repo`, `revision`, `size` | As in `vllm` |

The cluster check also confirms that `prepare.sh` finished on both GB10. `cargo test` checks that the folder's
`config.sh` and `Dockerfile` pin the same image, engine, repo and revision as the catalog.

Each file needs a URL pinned to a commit (`/resolve/<commit-sha>/file.gguf`), its `sha256` and its `size`.
Hugging Face lists them at `https://huggingface.co/api/models/<repo>/tree/main` (`lfs.oid` is the sha256).

Update the [Tested on](../README.md#tested-on) table and the
[Which model should I pick?](../README.md#which-model-should-i-pick) charts:

```bash
npm run models-table
```

Validate the catalog:

```bash
cd src-tauri && cargo test
```

Real end-to-end test (downloads the engine and the first model, starts it and asks it something):

```bash
cd src-tauri && cargo test --release -- --ignored --nocapture
```

## Building from source

Requirements: Node 22+, stable Rust and the [Tauri prerequisites](https://tauri.app/start/prerequisites/) for your OS.

```bash
npm install
npm run tauri dev
```

Installer:

```bash
npm run tauri build
```

## Shipping an update

Users get a new version (new models or features) when the original author publishes a release at
[MangelZabalaDevelop/ambassador-local](https://github.com/MangelZabalaDevelop/ambassador-local/releases), the endpoint
set in `plugins.updater.endpoints` in `tauri.conf.json`. The steps below describe how such a release is made, for
anyone building their own fork with their own signing key and endpoint. The upstream project's GitHub Actions release
workflow is not included in this copy, so each step is run by hand.

1. Bump `version` in `src-tauri/tauri.conf.json`, `src-tauri/Cargo.toml`, `src-tauri/Cargo.lock` and `package.json` /
   `package-lock.json`.
2. Run the tests (`cd src-tauri && cargo test`), then build and sign the installers for Windows x64, Linux x64 and
   Linux ARM64 (GB10) with `npm run tauri build`, with the signing key in `TAURI_SIGNING_PRIVATE_KEY` (the contents of
   the private key file) and its password in `TAURI_SIGNING_PRIVATE_KEY_PASSWORD`.
3. Put the `.sig` files from the three builds in one folder and write `latest.json` from them with
   [`scripts/updater-json.mjs`](../scripts/updater-json.mjs):
   `node scripts/updater-json.mjs <folder> <owner>/<repo> <tag>`.
4. Create a draft release for the tag, attach the installers and `latest.json`, and publish it. The app reads
   `releases/latest/download/latest.json`; drafts are invisible to it, so the app keeps offering the previous version
   until you publish. The release notes you write are shown in the app.

Never commit the private key, and keep a backup: if it is lost, installed apps can no longer receive updates. The
matching public key is `plugins.updater.pubkey` in `tauri.conf.json`.

The repository must be public for users' apps to reach the releases.
