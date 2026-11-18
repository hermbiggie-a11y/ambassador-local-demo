# Third-party notices

This repository is Dell Technologies' copy of Ambassador Local. The notices below are kept exactly as the upstream
project ships them. Folders that carry their own licence file keep it in place.

## Ambassador Local (the app, its scripts and recipes)

Source: https://github.com/MangelZabalaDevelop/ambassador-local, MIT. Dell's changes are covered by the same licence
(see `LICENSE`).

```
MIT License

Copyright (c) 2026 Miguel Zabala

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## llama.cpp chat page (`src-tauri/llama-ui/`)

The chat page llama-server brings, copied from llama.cpp. Its licence file stays at `src-tauri/llama-ui/LICENSE`.

```
MIT License

Copyright (c) 2023-2026 The ggml authors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## Lab logos (`src/assets/labs/`)

Upstream credit, kept as written:

> Lab logos from [Lobe Icons](https://github.com/lobehub/lobe-icons) (MIT). Dell, NVIDIA and other logos and names are trademarks of their respective owners.

## Downloaded at run time (not included in this repository)

The app and the recipes fetch these from their own publishers when you install or run a model. Each is pinned by
version, commit, digest or SHA256, and each stays under its own licence, as stated in the upstream recipe files:

| What | Source | Licence as stated upstream |
|---|---|---|
| llama.cpp engine builds | `github.com/ggml-org/llama.cpp` releases (version in `recipes/catalog.json`) | MIT |
| TensorFold engine | `github.com/bertholomus/TensorFold` @ `6366d2c` | see that repository |
| DeepSeek-V4.1 kernel kit | `github.com/bertholomus/deepseek-v4.1-tensorfold-tp2-2xgb10` @ `abe07c7` | Apache-2.0 |
| DeepSeek V4 Flash Vision serving recipe | GaelicThunder's serving recipe on GitHub (repository in `SRC_REPO`, `recipes/vllm/deepseek-v4-flash-vision-gb10/config.sh`) @ `aaff897` | Apache-2.0 |
| GLM-5.3-Flash container image | `ghcr.io/gitcommit90/glm-5.3-one-spark` (recipe `github.com/gitcommit90/glm-5.3-one-spark` @ `ddc28b1`) | MIT |
| GLM-5.3-Flash helper model | `incoai/GLM-5.3-Flash-DFlash2` on Hugging Face | CC BY-NC-ND 4.0 (noncommercial use only) |
| DeepSeek-V4.1-Flash-1017 weights | `LibertAIDAI/DeepSeek-V4.1-Flash-1017-EXL3-2.9bpw` and `deepseek-ai/DeepSeek-V4.1-Flash-1017` on Hugging Face | MIT |
| Other model weights | Hugging Face repositories named in `recipes/catalog.json` and each recipe's `config.sh` | each model's own licence |
| Container base images | `nvcr.io/nvidia/pytorch`, `ghcr.io/local-inference-lab/vllm` and the images in each recipe's `config.sh` | each image's own licence |

Read each model's licence before you use it, especially for commercial work.
