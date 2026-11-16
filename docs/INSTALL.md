# Install Wan 2.2

**Modified by Transformaiton.com.** This file was added by [Transformaiton.com](https://transformaiton.com/). It is a customer install guide for the public fork of [Wan-Video/Wan2.2](https://github.com/Wan-Video/Wan2.2). The Python code is the Wan team's code. [Transformaiton.com](https://transformaiton.com/) is not affiliated with or endorsed by the Wan team, Alibaba Group, or Alibaba Cloud.

This guide installs the smallest model, **TI2V-5B**, and points at the heavier checkpoints if you need them. Generation steps are in [USAGE.md](USAGE.md). A short overview is in the [README](../README.md).

The repository also contains [INSTALL.md](../INSTALL.md) at the root. That note covers `pip install .` and Poetry. The steps below follow the install path in the original Wan README: a virtual environment and `requirements.txt`.

## Before you start

You need:

- A Linux machine, or a Linux cloud GPU. The commands in the original project are written for that setup.
- Python 3.10, 3.11, or 3.12. `pyproject.toml` requires Python `>=3.10,<4.0`.
- An NVIDIA GPU with a working CUDA driver. `generate.py` runs on CUDA.
- Enough free disk for the checkpoint you choose, plus PyTorch and the Hugging Face download cache. Leave extra room beyond the checkpoint size. The download client keeps a cache while it writes the final files.
- `git` and `python3` on your `PATH`.

### Disk space for the weights

The weights are not in the git clone. Hugging Face reports the following repository storage. These figures include the checkpoint files published in each repo (the diffusion weights, the VAE, and the bundled text encoder). They can change. Check the model page before you download.

| Checkpoint | Hugging Face repo | Storage reported on the model API |
| --- | --- | --- |
| TI2V-5B (start here) | [Wan-AI/Wan2.2-TI2V-5B](https://huggingface.co/Wan-AI/Wan2.2-TI2V-5B) | about 54 GB |
| T2V-A14B | [Wan-AI/Wan2.2-T2V-A14B](https://huggingface.co/Wan-AI/Wan2.2-T2V-A14B) | about 241 GB |
| I2V-A14B | [Wan-AI/Wan2.2-I2V-A14B](https://huggingface.co/Wan-AI/Wan2.2-I2V-A14B) | about 126 GB |
| S2V-14B | [Wan-AI/Wan2.2-S2V-14B](https://huggingface.co/Wan-AI/Wan2.2-S2V-14B) | about 49 GB |
| Animate-14B | [Wan-AI/Wan2.2-Animate-14B](https://huggingface.co/Wan-AI/Wan2.2-Animate-14B) | about 73 GB |

ModelScope hosts the same checkpoints. Links are in the original README.

### GPU memory

Full hardware notes, including the Wan team's timing table, are in the [README](../README.md#hardware) and in [USAGE.md](USAGE.md#hardware). The short version:

- **TI2V-5B** is the model to install first. The original README says the single-GPU command runs on a GPU with at least **24 GB** of VRAM (an RTX 4090 is the example they give) when you pass `--offload_model True --convert_model_dtype --t5_cpu`. Their published measurement on one RTX 4090 at 720p peaks at about **23 GB**.
- The **14B** single-GPU examples are documented as needing at least **80 GB**. Speech-to-video uses the same 80 GB figure. Character animation has no VRAM number in the original README.

If you do not have that GPU, use a hosted demo or rent a card. See [Cheaper ways to run it](../README.md#cheaper-ways-to-run-it).

## 1. Clone this fork

```bash
git clone https://github.com/Transformaiton/Wan2.2.git
cd Wan2.2
```

The upstream project is [github.com/Wan-Video/Wan2.2](https://github.com/Wan-Video/Wan2.2). This fork adds the customer documentation. It does not change the model code.

## 2. Create a virtual environment

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip setuptools wheel
```

On later shells, run `source .venv/bin/activate` again from the repository directory before you install or generate.

## 3. Install PyTorch for your CUDA driver

`requirements.txt` asks for `torch>=2.4.0`. Install a PyTorch build that matches your driver before the rest of the requirements, using the selector at [pytorch.org/get-started/locally](https://pytorch.org/get-started/locally/). A CPU-only PyTorch build will not run the generator.

Confirm CUDA is visible:

```bash
python -c "import torch; print(torch.__version__, torch.cuda.is_available(), torch.cuda.get_device_name(0))"
```

You want `True` and the name of your GPU.

## 4. Install the remaining Python packages

The original README installs everything in one step, and asks you to install `flash_attn` last if that package fails to build:

```bash
pip install -r requirements.txt
```

`flash_attn` compiles against PyTorch and CUDA. If that line fails, install the other packages first, then install `flash_attn`:

```bash
grep -v '^flash_attn$' requirements.txt > /tmp/wan-requirements-no-flash.txt
pip install -r /tmp/wan-requirements-no-flash.txt
pip install flash_attn --no-build-isolation
```

`--no-build-isolation` is the same workaround already written for Poetry in the root [INSTALL.md](../INSTALL.md). It lets the build see the PyTorch you just installed. You also need a CUDA toolkit (`nvcc`) that matches the PyTorch build. The standard generator calls flash attention, so a missing `flash_attn` install will stop a run rather than silently fall back.

`requirements.txt` pins `transformers>=4.49.0,<=4.51.3` and `numpy>=1.23.5,<2`. Keep those pins. A newer Transformers or NumPy 2.x is outside the range the upstream file was written for.

### Optional packages

Install these only for the matching task. They are not required for TI2V-5B.

```bash
# Speech-to-video, including CosyVoice text-to-speech
pip install -r requirements_s2v.txt

# Wan-Animate preprocessing and inference
pip install -r requirements_animate.txt
```

`requirements_animate.txt` installs SAM 2 from a pinned Git commit at `facebookresearch/sam2`. That install needs `git` and a working compiler.

## 5. Download TI2V-5B

From the repository root, with the virtual environment active:

```bash
pip install "huggingface_hub[cli]"
huggingface-cli download Wan-AI/Wan2.2-TI2V-5B --local-dir ./Wan2.2-TI2V-5B
```

A gated or rate-limited download is fixed by logging in:

```bash
huggingface-cli login
```

ModelScope is the other download path from the original README:

```bash
pip install modelscope
modelscope download Wan-AI/Wan2.2-TI2V-5B --local_dir ./Wan2.2-TI2V-5B
```

The license on the weight files is the license on the model card, which is separate from this git repository. See [Weights and license](#weights-and-license) below.

To download a larger checkpoint later, use the same command with a different repo id and directory:

| Task | Command target | Local directory |
| --- | --- | --- |
| Text-to-video, 14B MoE | `Wan-AI/Wan2.2-T2V-A14B` | `./Wan2.2-T2V-A14B` |
| Image-to-video, 14B MoE | `Wan-AI/Wan2.2-I2V-A14B` | `./Wan2.2-I2V-A14B` |
| Speech-to-video | `Wan-AI/Wan2.2-S2V-14B` | `./Wan2.2-S2V-14B` |
| Character animation | `Wan-AI/Wan2.2-Animate-14B` | `./Wan2.2-Animate-14B` |

## 6. Check that the install can start

```bash
python generate.py --help
```

Then follow the TI2V-5B quick start in [USAGE.md](USAGE.md#quick-start-ti2v-5b).

## Install trouble

**`flash_attn` fails to build.** Install PyTorch first, from the CUDA build that matches your driver. Install the other requirements. Then run `pip install flash_attn --no-build-isolation`. Confirm `nvcc` is installed. The original README gives the same order: other packages first, `flash_attn` last.

**`torch.cuda.is_available()` is false.** The PyTorch wheel is the CPU build, or the driver is not visible inside this shell. Reinstall PyTorch from the local selector. On a rented GPU, use the image that already pairs the driver and CUDA.

**`No matching distribution` for `torch`.** The Python version is outside 3.10–3.12, or `pip` is old. Recreate the venv with a supported Python and upgrade `pip`.

**Download stops or the checkpoint directory looks empty.** Check free disk, then run the Hugging Face command again. `huggingface-cli download` resumes. A login token is required if the client returns 401 or 403.

**`huggingface-cli: command not found`.** The package is `huggingface_hub[cli]`, and the script is on the venv's path only after `source .venv/bin/activate`.

**NumPy or Transformers import errors after you install something else.** Reinstall from `requirements.txt` so NumPy stays below 2 and Transformers stays at or below 4.51.3.

**Out of memory on the first generate.** That is a GPU-size issue, not an install failure. Use the TI2V-5B flags in [USAGE.md](USAGE.md#quick-start-ti2v-5b), or move to a 24 GB GPU.

## Weights and license

This git repository is Apache-2.0. See [LICENSE.txt](../LICENSE.txt) and [NOTICE](../NOTICE).

The weight files you download are published by Wan-AI. Each model card is under its own terms. As checked for this guide, the cards for TI2V-5B, T2V-A14B, I2V-A14B, S2V-14B, and Animate-14B are marked `apache-2.0`, and the original README states the same license plus the use conditions in that README. Read the card before you use a checkpoint. The card governs if it changes.

The checkpoints also include components credited in the original acknowledgements, including the umt5 text encoder. Those components keep the terms of their own projects.

## Contact

Install help for this fork: [info@transformaiton.com](mailto:info@transformaiton.com), [transformaiton.com](https://transformaiton.com/), [github.com/Transformaiton](https://github.com/Transformaiton).

The Wan research and product channels are the ones in the original README: [Discord](https://discord.gg/AKNgpMK4Yj) and the WeChat group linked from the [upstream README](https://github.com/Wan-Video/Wan2.2). Transformaiton.com does not operate those channels.
