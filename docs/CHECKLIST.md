# Ambassador Local: setup, run and reset checklist

The full checklist for installing, running, connecting and resetting Ambassador Local on a Dell Pro Max Tower T2 or
Dell Pro Precision GB10. The short version is in the [README](../README.md#setup).

## 1. Before you start

- [ ] Your system is one it was built and run on (see [Hardware](../README.md#hardware)):
  - Dell Pro Max Tower T2 with an NVIDIA RTX PRO 6000 Blackwell Workstation Edition, Windows 11 Pro or Ubuntu 24.04.
  - Dell Pro Precision GB10 with NVIDIA DGX OS 7, one unit or two linked over ConnectX-7.
- [ ] The NVIDIA driver is installed and `nvidia-smi` shows the GPU. The app uses `nvidia-smi` to recognise the
  workstation, and only the workstation detected on the machine can be used.
- [ ] Enough free disk for the model you want: the [Tested on](../README.md#tested-on) table lists the disk space each
  recipe's weights need.
- [ ] Linux: `apt-get` and `python3` are present (both come with Ubuntu and DGX OS).
- [ ] Windows: 64-bit Windows.

Only for the container recipes (vLLM and TensorFold, Linux only):

- [ ] Docker with the NVIDIA container runtime, usable without sudo.
- [ ] The recipe's image pulled (`docker pull` the image in the recipe's `config.sh`) and its weights in the Hugging
  Face cache, pinned to the commit in the recipe's README (`hf download <repo> --revision <commit>`).
- [ ] A free GPU: stop any other model first.

Only for two linked Dell Pro Precision GB10:

- [ ] The two units are linked by the direct ConnectX cable, with the cluster setup from NVIDIA Sync done
  (passwordless SSH between them). The app does not set up the cluster.
- [ ] Both units have the same interface and RDMA device names.
- [ ] For the TensorFold recipe, `prepare.sh` has finished on both units (see
  [`recipes/tensorfold/deepseek-v41-flash-1017/README.md`](../recipes/tensorfold/deepseek-v41-flash-1017/README.md)).

## 2. Install

**Windows:** open PowerShell and paste

```powershell
irm https://raw.githubusercontent.com/MangelZabalaDevelop/ambassador-local/main/install.ps1 | iex
```

**Linux** (Dell Pro Max Tower T2 with Ubuntu, Dell Pro Precision GB10 with DGX OS): open a terminal and paste

```bash
curl -fsSL https://raw.githubusercontent.com/MangelZabalaDevelop/ambassador-local/main/install.sh | bash
```

The installers and releases are the original author's official, signed ones
([MangelZabalaDevelop/ambassador-local](https://github.com/MangelZabalaDevelop/ambassador-local/releases)); Dell builds
and signs no installers.

- [ ] The script found the latest official release and printed its name and size.
- [ ] "Checking SHA256" passed. The script compares the download with the SHA256 GitHub publishes for that file; if
  they differ it stops and installs nothing.
- [ ] Windows installs for your user (no admin). Linux installs with `apt`, which asks for your password.
- [ ] **Ambassador Local** opens from the Start menu or the app menu (Linux: or run `ambassador-local`).

If the release cannot be reached: check the network, then download the installer by hand from the official
[Releases](https://github.com/MangelZabalaDevelop/ambassador-local/releases).

## 3. Pick a model

- [ ] **System → Workstation → Lab → Model.** The system and workstation are detected for you.
- [ ] Every model shows the speed measured on that workstation. **Miguel Tuned** marks recipes tuned and tested by
  hand; the crown marks the recommended one.
- [ ] Not sure which one? Use the charts in [Which model should I pick?](../README.md#which-model-should-i-pick): pick
  the highest group that still feels fast enough. For chat, anything above about 20 tokens/s already writes faster
  than you can read; for coding agents, faster is better.

## 4. Run

- [ ] Press **Run**. The app downloads what it needs (each file checked against the SHA256 in the catalog; a file that
  does not match is deleted and never run), loads the model on the GPU and shows **Live**.
- [ ] Container recipes: the app first checks the machine (on two GB10: the second unit on the ConnectX cable,
  passwordless SSH to it, and the checks below on both): Docker with the NVIDIA runtime, the pinned image, complete
  weights and a free GPU. Fix whatever the Run screen reports, then press **Run** again.
- [ ] Two GB10 with llama.cpp: the app runs on one unit and borrows the second one's memory with llama.cpp's
  `ggml-rpc-server`, bound only to the direct ConnectX cable's IP.

## 5. Use it

When it says **Live**:

- [ ] **Open chat**, next to *Stop server*: llama.cpp's chat page in its own window, with the API key already set. It
  closes when the model stops. vLLM and TensorFold only have the API, so for them the app serves the same page on
  `127.0.0.1` and passes the chat to the model ([`src-tauri/src/proxy.rs`](../src-tauri/src/proxy.rs)). With llama.cpp
  models the page also opens in a browser (the endpoint URL without `/v1`) and asks for the key once.
- [ ] **Endpoint URL** and **API key**, in *Connect apps*: paste them into any OpenAI-compatible app. The key stays the
  same every time.
- [ ] **Hermes:** run `hermes setup`, choose *Custom OpenAI-compatible endpoint*, then paste the URL and the key.
- [ ] **OpenCode** and other coding agents: add an OpenAI-compatible provider with the same URL and key. For example,
  in `opencode.json`:

```json
{
  "provider": {
    "local": {
      "npm": "@ai-sdk/openai-compatible",
      "options": { "baseURL": "http://127.0.0.1:8080/v1", "apiKey": "{env:AMBASSADOR_KEY}" },
      "models": { "qwen38-flash-next-fp4": {} }
    }
  },
  "model": "local/qwen38-flash-next-fp4"
}
```

Use the port from the endpoint URL and the model id the app shows; put the API key in `AMBASSADOR_KEY`.

## 6. Stop and reset

- [ ] **Stop server** stops the model. Closing the app also stops it, and with container recipes runs the recipe's
  `stop.sh` (on both GB10 when there are two). If a container recipe was started by hand and left running, run its
  `stop.sh`.
- [ ] To start clean, stop the model, close the app and open it again.
- [ ] To remove everything: click **Uninstall Ambassador Local…** at the bottom of the sidebar. The app stops the
  running model, closes and removes itself: on Windows with its own uninstaller (Start menu shortcut included), on
  Ubuntu and DGX OS with `apt` (Ubuntu asks for your password), and as an AppImage by deleting the file. Leave **Also
  delete downloaded models and settings** checked to free their space too. Weights in the Hugging Face cache and Docker
  images stay, since other tools may use them.
- [ ] Without the app: on Windows, *Settings → Apps → Ambassador Local → Uninstall*; on Linux,
  `sudo apt remove ambassador-local`.

Files live in the app data folder (Windows: `%LOCALAPPDATA%\io.github.ambassadorlocal.app`, Linux:
`~/.local/share/io.github.ambassadorlocal.app`). Container recipes work in `~/.cache/ambassador-local/<engine>/<recipe>/`.

## 7. Security

- **Network access is your choice.** With *Allow other devices on my network* (on by default, so Hermes or other apps
  on another PC can connect) the server listens on your local network; turn it off and it listens on **127.0.0.1
  only**. Either way every request needs the **API key** the app generates. On Windows the firewall may ask once whether
  `llama-server` may accept connections.
- Downloads come only from `github.com` (official llama.cpp builds) and `huggingface.co`, redirects included.
- Every file is checked against the **SHA256** in the catalog. If it doesn't match, it is deleted and never run.
- Recipes are **data, not commands**: they can only set the allowed fields (see [DEVELOPING.md](DEVELOPING.md)). The
  app always sets `--host` itself (127.0.0.1, or your network when you allow it) and adds `--jinja`.
- No admin rights, no telemetry.
- **GB10 ×2 with llama.cpp:** the app reaches the second GB10 over the passwordless SSH that NVIDIA's cluster setup
  leaves between the two, copies the same engine there if it is missing, and starts the RPC server bound **only to the
  direct ConnectX cable's IP** (never your home or office network). It is stopped when you stop the model or close the
  app, and also if the app crashes.
- **Container recipes (vLLM, TensorFold):** images are pinned by digest or built from pinned sources, and weights by
  commit. The app checks the machine before it starts anything, runs the recipe's own scripts (which it carries
  inside), and stops the containers, on both GB10 when there are two, when you stop the model or close the app.
- **Updates are optional and signed.** At startup the app asks GitHub once whether the original project's official
  releases have a newer one
  (nothing is downloaded or sent). It installs only when you click **Update** and confirm, and only a package signed
  with the project's key: the public key is built into the app, so a tampered or third-party package is rejected. New
  models come with the update, since the catalog is part of the app.
