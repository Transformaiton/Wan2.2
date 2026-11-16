# Wan 2.2 by Transformaiton.com

> **Modified by Transformaiton.com.** The customer guide in this README was written by [Transformaiton.com](https://transformaiton.com/). The Wan team's original README is preserved in full under [Original project documentation](#original-project-documentation). The Python model code is unchanged. See [NOTICE](NOTICE) for the list of files this fork adds or edits. [LICENSE.txt](LICENSE.txt) is the original Apache License, Version 2.0, and it is unchanged.

This repository is Transformaiton.com's public copy of **Wan 2.2**, an open video-generation model from the [Wan team](https://github.com/Wan-Video/Wan2.2) at Alibaba. It can build a short clip from a text prompt, from a still image, from speech, or by moving a character to match another video.

[Transformaiton.com](https://transformaiton.com/) is an AI consultancy for small businesses. We added this guide so a business can tell what the model does, what hardware it needs, and how to run the smallest checkpoint. We are not affiliated with, endorsed by, or sponsored by the Wan team, Alibaba Group, or Alibaba Cloud.

- Install, step by step: [docs/INSTALL.md](docs/INSTALL.md)
- Everyday use, prompts, and the other tasks: [docs/USAGE.md](docs/USAGE.md)
- Upstream code and the research write-up: [github.com/Wan-Video/Wan2.2](https://github.com/Wan-Video/Wan2.2) · [paper](https://arxiv.org/abs/2503.20314) · [wan.video](https://wan.video)

## What it does, in plain English

You describe a shot, or you hand the model a picture, and it writes a video of a few seconds. The smallest model, **TI2V-5B**, runs at 720p and 24 frames per second. Its default length is 121 frames, about 5 seconds. T2V-A14B, I2V-A14B, and S2V-14B default to 81 frames at 16 fps, also about 5 seconds. Animate-14B defaults to 77 frames at 30 fps.

Five checkpoints ship separately from this git repository. You download the one you need:

| Model | What you ask it for | Where a small business starts |
| --- | --- | --- |
| **TI2V-5B** | Text-to-video, or a still image plus text, at 720p | Start here. It is the one that fits a consumer 24 GB GPU. |
| **T2V-A14B** | Text-to-video at 480p or 720p | A larger MoE model. The single-GPU example is written for an 80 GB GPU. |
| **I2V-A14B** | A still image to video at 480p or 720p | Same hardware class as T2V-A14B. |
| **S2V-14B** | Speech (or generated speech) plus a still, to video | Single-GPU example is written for an 80 GB GPU. |
| **Animate-14B** | A character still plus a motion video, for animation or replacement | Advanced. Preprocessing comes first. No VRAM figure is published in the original README. |

The A14B models are a mixture-of-experts design: about 27 billion parameters in total, with about 14 billion active on each step. TI2V-5B is one 5-billion-parameter model with a higher-compression video autoencoder. The original README and the paper describe the architecture. This guide does not restate the research.

## What a small business can use it for

The useful output is a short shot you can review and then cut into a larger piece.

- **Product videos.** A mug, a garment, a tool, or a packaged good, with one camera move and one setting. Describe the product, the surface, the light, and the move.
- **Social ads.** A 5-second shot for a paid social post or a story. Titles, a voiceover, and a longer edit still happen in your editor. The model writes the shot.
- **Animating product photos.** Pass a real product photo with `--image` and describe a small motion, such as a slow move toward the product. The photo is the official image-to-video path. TI2V-5B and I2V-A14B both do this. TI2V-5B is the one to try first.

You are responsible for what you publish. Check the clip against the real product before it goes on an ad. The model can invent details. The Wan team's license text also limits harmful and illegal uses. That text is quoted under [Credits, license, and affiliation](#credits-license-and-affiliation).

## Official examples already in this project

The pictures and the demo below were published by the Wan team. Transformaiton.com did not generate them, and this fork does not add sample outputs of its own.

The still used by the image-to-video commands is `examples/i2v_input.JPG`.

<img src="examples/i2v_input.JPG" alt="Official Wan 2.2 example input: a white cat in sunglasses on a surfboard" width="480"/>

The animation and replacement examples use these reference stills, plus the motion videos next to them in the repo (`examples/wan_animate/animate/video.mp4` and `examples/wan_animate/replace/video.mp4`).

<img src="examples/wan_animate/animate/image.jpeg" alt="Official Wan-Animate reference still shipped in examples/wan_animate/animate/" width="360"/>
<img src="examples/wan_animate/replace/image.jpeg" alt="Official Wan-Animate replacement reference still shipped in examples/wan_animate/replace/" width="360"/>

The demo video from the Wan team's README:

<div align="center">
  <video src="https://github.com/user-attachments/assets/b63bfa58-d5d7-4de6-a1a2-98970b06d9a7" width="70%"></video>
</div>

More official clips are on [wan.video](https://wan.video), the [TI2V-5B Hugging Face Space](https://huggingface.co/spaces/Wan-AI/Wan-2.2-5B), the [animation project page](https://humanaigc.github.io/wan-animate), and the [speech-to-video project page](https://humanaigc.github.io/wan-s2v-webpage). A full list of the example files in this repo is in [docs/USAGE.md](docs/USAGE.md#official-examples-already-in-the-project).

## Hardware

<a id="hardware"></a>

Plan from the original README's written minimums and from the Wan team's measured table (`assets/comp_effic.png`). The table reports **seconds / peak GPU memory in GB**. The test setup is quoted in the original README: offload is on for the single-GPU numbers, prompt extension is off, and the figures are averages after warm-up.

<img src="assets/comp_effic.png" alt="Wan team table of generation time and peak GPU memory" width="100%"/>

| Model and job | What to budget |
| --- | --- |
| TI2V-5B, 720p, 1 GPU | At least **24 GB**. An RTX 4090 is the card named in the original README. Their measurement on one 4090 peaks at about **22.9 GB** (text, 535 seconds) or **22.8 GB** (image, 525 seconds). |
| TI2V-5B, 720p, 4 or 8 GPUs | The same table stays near **23 GB per GPU**, because that test still uses offload. Extra GPUs cut the time (about 227–231 seconds on 4 GPUs, about 157–160 seconds on 8). |
| T2V-A14B or I2V-A14B, 1 GPU | The example command is documented at **at least 80 GB**. Measured peaks with offload are about **41 GB** at 480p and about **60 GB** at 720p on H20, A100/A800, and H100/H800. Buy or rent to the 80 GB figure. |
| T2V-A14B or I2V-A14B, 4 GPUs | About **41 GB** per GPU at 480p and about **52 GB** per GPU at 720p. |
| T2V-A14B or I2V-A14B, 8 GPUs | About **26 GB** per GPU at 480p and about **37 GB** per GPU at 720p. |
| S2V-14B, 1 GPU | Documented at **at least 80 GB**. This model is not in the timing table. |
| Animate-14B | No VRAM number in the original README or the table. Treat it as a 14B model. |

The 4090 row in the table is only TI2V-5B. The table does not show a 14B model on a 24 GB card. A longer `frame_num` than the default uses more memory than these measurements.

Checkpoint downloads are large. Hugging Face reports about **54 GB** for TI2V-5B, **126 GB** for I2V-A14B, **241 GB** for T2V-A14B, **49 GB** for S2V-14B, and **73 GB** for Animate-14B. Details and the install steps are in [docs/INSTALL.md](docs/INSTALL.md).

## Cheaper ways to run it

<a id="cheaper-ways-to-run-it"></a>

**Try it in a browser before you buy a GPU.** The Wan team already hosts these:

- [TI2V-5B on Hugging Face Spaces](https://huggingface.co/spaces/Wan-AI/Wan-2.2-5B)
- [wan.video](https://wan.video)
- [ComfyUI Wan 2.2 tutorial](https://docs.comfy.org/tutorials/video/wan/wan2_2) (and the [Chinese tutorial](https://docs.comfy.org/zh-CN/tutorials/video/wan/wan2_2))
- Speech-to-video: [Hugging Face](https://huggingface.co/spaces/Wan-AI/Wan2.2-S2V) and [ModelScope](https://www.modelscope.cn/studios/Wan-AI/Wan2.2-S2V)
- Animation: [Hugging Face](https://huggingface.co/spaces/Wan-AI/Wan2.2-Animate) and [ModelScope](https://www.modelscope.cn/studios/Wan-AI/Wan2.2-Animate)
- Diffusers packages, if you already use Diffusers: [TI2V-5B](https://huggingface.co/Wan-AI/Wan2.2-TI2V-5B-Diffusers), [T2V-A14B](https://huggingface.co/Wan-AI/Wan2.2-T2V-A14B-Diffusers), [I2V-A14B](https://huggingface.co/Wan-AI/Wan2.2-I2V-A14B-Diffusers)

**Rent a GPU by the hour** if the demo is enough to prove the look and you want to run the real checkpoint. A 24 GB NVIDIA card is the documented starting point for TI2V-5B. An 80 GB card matches the written single-GPU examples for the 14B models. RunPod, Vast.ai, Lambda, and the GPU instances on AWS, Google Cloud, and Azure commonly list cards in this range. Compare the current hourly price with the provider. Transformaiton.com does not operate those services and does not set their prices.

The original README also lists community projects aimed at lower memory or faster runs, including [LightX2V](https://github.com/ModelTC/LightX2V), [DiffSynth-Studio](https://github.com/modelscope/DiffSynth-Studio), and [Kijai's ComfyUI wrapper](https://github.com/kijai/ComfyUI-WanVideoWrapper). They are third-party. Read their docs.

## Install

The full procedure, including the PyTorch, `flash_attn`, and download steps, is [docs/INSTALL.md](docs/INSTALL.md). The short path on Linux:

```bash
git clone https://github.com/Transformaiton/Wan2.2.git
cd Wan2.2
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
```

Install a CUDA build of PyTorch 2.4 or newer from [pytorch.org](https://pytorch.org/get-started/locally/), then:

```bash
pip install -r requirements.txt
pip install "huggingface_hub[cli]"
huggingface-cli download Wan-AI/Wan2.2-TI2V-5B --local-dir ./Wan2.2-TI2V-5B
```

If `flash_attn` fails to compile, install the other requirements first and `flash_attn` last. The original README says the same thing. The root [INSTALL.md](INSTALL.md) is the existing short note for `pip install .` and Poetry. Either install path can be used. The customer steps follow `requirements.txt`.

## Quick start: TI2V-5B

Run this from the repository root after the install. The size is quoted so the shell does not treat `*` as a wildcard. The prompt is the official example.

```bash
python generate.py \
  --task ti2v-5B \
  --size '1280*704' \
  --ckpt_dir ./Wan2.2-TI2V-5B \
  --offload_model True \
  --convert_model_dtype \
  --t5_cpu \
  --prompt "Two anthropomorphic cats in comfy boxing gear and bright gloves fight intensely on a spotlighted stage"
```

The same model animates a still when you add `--image`. This command uses the official cat photo and the official prompt:

```bash
python generate.py \
  --task ti2v-5B \
  --size '1280*704' \
  --ckpt_dir ./Wan2.2-TI2V-5B \
  --offload_model True \
  --convert_model_dtype \
  --t5_cpu \
  --image examples/i2v_input.JPG \
  --prompt "Summer beach vacation style, a white cat wearing sunglasses sits on a surfboard. The fluffy-furred feline gazes directly at the camera with a relaxed expression. Blurred beach scenery forms the background featuring crystal-clear waters, distant green hills, and a blue sky dotted with white clouds. The cat assumes a naturally relaxed posture, as if savoring the sea breeze and warm sunlight. A close-up shot highlights the feline's intricate details and the refreshing atmosphere of the seaside."
```

The video is written in the current directory unless you set `--save_file`. On one RTX 4090, expect on the order of nine minutes for a 720p clip once the weights are loaded, which matches both the original "under 9 minutes" statement and the table above.

To point the same command at your own product photo, change `--image` and `--prompt`, and set `--save_file product-clip.mp4`. A worked version of that edit is in [docs/USAGE.md](docs/USAGE.md#a-product-photo). Commands for the 14B models, speech-to-video, and animation are in that file and in the original documentation below.

## Troubleshooting and FAQ

**The script asks for a checkpoint directory.** The weights are a separate download. `--ckpt_dir ./Wan2.2-TI2V-5B` has to point at the folder from `huggingface-cli download`.

**CUDA out of memory.** Use the TI2V-5B command above, including `--offload_model True --convert_model_dtype --t5_cpu`. That is the 24 GB setup. The 14B single-GPU examples are written for 80 GB.

**`flash_attn` fails to install, or a run says FlashAttention is unavailable.** Install every other package in `requirements.txt`, then install `flash_attn`. The generator calls flash attention. A missing install stops the run. Steps are in [docs/INSTALL.md](docs/INSTALL.md).

**`Unsupport size` or a shell glob error.** TI2V-5B accepts only `'1280*704'` and `'704*1280'`. Keep the quotes.

**The first launch looks stuck.** The checkpoint is loading from disk. After that, a 720p TI2V-5B clip is about nine minutes on one RTX 4090 in the Wan team's measurement.

**Can I skip the GPU and still see a video?** Use the hosted Spaces and [wan.video](https://wan.video) linked above. The local `generate.py` path uses CUDA.

**Who owns the output, and can I put it in an ad?** The Wan team says the models are Apache 2.0 and that they claim no rights over generated contents, as long as your use follows the license. You are accountable for the clip. Read the license section below and the model card. Transformaiton.com does not give legal advice.

More answers, including prompt-extension failures and multi-GPU errors, are in [docs/USAGE.md](docs/USAGE.md#troubleshooting-and-faq).

## Credits, license, and affiliation

<a id="credits"></a>

**Wan team (Alibaba).** Wan 2.2 was created by the Wan team. The code in this repository, the example media, the efficiency table, and the original README are theirs. The Python file headers read "Copyright 2024-2025 The Alibaba Wan Team Authors."

- Code: [github.com/Wan-Video/Wan2.2](https://github.com/Wan-Video/Wan2.2)
- Project: [wan.video](https://wan.video)
- Models: [huggingface.co/Wan-AI](https://huggingface.co/Wan-AI/) and [ModelScope Wan-AI](https://modelscope.cn/organization/Wan-AI)
- Paper: [Wan: Open and Advanced Large-Scale Video Generative Models](https://arxiv.org/abs/2503.20314)
- English user guide linked from the upstream README: [User Guide (English)](https://alidocs.dingtalk.com/i/nodes/EpGBa2Lm8aZxe5myC99MelA2WgN7R35y)

Please cite the Wan paper if you use the model in research. The BibTeX is in the original README.

**This fork.** The customer text at the top of this README, [docs/INSTALL.md](docs/INSTALL.md), [docs/USAGE.md](docs/USAGE.md), and [NOTICE](NOTICE) were added by Transformaiton.com. Transformaiton.com is not affiliated with, endorsed by, or sponsored by the Wan team, Alibaba Group, or Alibaba Cloud. Product names belong to their owners.

**Code license.** This repository is licensed under the Apache License, Version 2.0. The full text is [LICENSE.txt](LICENSE.txt), unchanged. Apache 2.0 lets you use, modify, and redistribute the code, including commercial use, if you keep the license, state your changes, and keep the original notices. This README is the change notice for the documentation. The model code was not modified.

**Weight license.** The checkpoints are not in this git repository. Wan-AI publishes them, and each model card carries the weight license. As checked for this guide, the cards for [Wan2.2-TI2V-5B](https://huggingface.co/Wan-AI/Wan2.2-TI2V-5B), [Wan2.2-T2V-A14B](https://huggingface.co/Wan-AI/Wan2.2-T2V-A14B), [Wan2.2-I2V-A14B](https://huggingface.co/Wan-AI/Wan2.2-I2V-A14B), [Wan2.2-S2V-14B](https://huggingface.co/Wan-AI/Wan2.2-S2V-14B), and [Wan2.2-Animate-14B](https://huggingface.co/Wan-AI/Wan2.2-Animate-14B) are marked Apache-2.0. The original README states the same license and adds the use conditions: the authors claim no rights over your generated contents, and you are accountable for use that must not involve illegal content, harm, personal information used to cause harm, misinformation, or targeting vulnerable people. Read [LICENSE.txt](LICENSE.txt) and the model card you download. If a model card changes, the card governs the weights. Checkpoints also bundle components credited in the original acknowledgements, including the umt5 text encoder. Those components keep their own terms.

The Wan team's logo, included here as attribution:

<img src="assets/logo.png" alt="Wan project logo, published by the Wan team" width="280"/>

## Contact

<a id="contact"></a>

Transformaiton.com helps small businesses put AI to work. Questions about this guide and this fork:

- Website: [https://transformaiton.com/](https://transformaiton.com/)
- Email: [info@transformaiton.com](mailto:info@transformaiton.com)
- GitHub: [https://github.com/Transformaiton](https://github.com/Transformaiton)
- This repository: [https://github.com/Transformaiton/Wan2.2](https://github.com/Transformaiton/Wan2.2)

Questions for the Wan research or product teams go to the channels in the original README: [Discord](https://discord.gg/AKNgpMK4Yj) and the [WeChat group](https://gw.alicdn.com/imgextra/i2/O1CN01tqjWFi1ByuyehkTSB_!!6000000000015-0-tps-611-1279.jpg). Transformaiton.com does not operate those channels.

## Original project documentation

The Wan team's README follows, unchanged, including their install notes, model table, every generation command, the efficiency write-up, the citation, and their own contact links.

<details>
<summary>Original project documentation (Wan team README)</summary>

# Wan2.2

<p align="center">
    <img src="assets/logo.png" width="400"/>
<p>

<p align="center">
    💜 <a href="https://wan.video"><b>Wan</b></a> &nbsp&nbsp ｜ &nbsp&nbsp 🖥️ <a href="https://github.com/Wan-Video/Wan2.2">GitHub</a> &nbsp&nbsp  | &nbsp&nbsp🤗 <a href="https://huggingface.co/Wan-AI/">Hugging Face</a>&nbsp&nbsp | &nbsp&nbsp🤖 <a href="https://modelscope.cn/organization/Wan-AI">ModelScope</a>&nbsp&nbsp | &nbsp&nbsp 📑 <a href="https://arxiv.org/abs/2503.20314">Paper</a> &nbsp&nbsp | &nbsp&nbsp 📑 <a href="https://wan.video/welcome?spm=a2ty_o02.30011076.0.0.6c9ee41eCcluqg">Blog</a> &nbsp&nbsp |  &nbsp&nbsp 💬  <a href="https://discord.gg/AKNgpMK4Yj">Discord</a>&nbsp&nbsp
    <br>
    📕 <a href="https://alidocs.dingtalk.com/i/nodes/jb9Y4gmKWrx9eo4dCql9LlbYJGXn6lpz">使用指南(中文)</a>&nbsp&nbsp | &nbsp&nbsp 📘 <a href="https://alidocs.dingtalk.com/i/nodes/EpGBa2Lm8aZxe5myC99MelA2WgN7R35y">User Guide(English)</a>&nbsp&nbsp | &nbsp&nbsp💬 <a href="https://gw.alicdn.com/imgextra/i2/O1CN01tqjWFi1ByuyehkTSB_!!6000000000015-0-tps-611-1279.jpg">WeChat(微信)</a>&nbsp&nbsp
<br>

-----

[**Wan: Open and Advanced Large-Scale Video Generative Models**](https://arxiv.org/abs/2503.20314) <be>


We are excited to introduce **Wan2.2**, a major upgrade to our foundational video models. With **Wan2.2**, we have focused on incorporating the following innovations:

- 👍 **Effective MoE Architecture**: Wan2.2 introduces a Mixture-of-Experts (MoE) architecture into video diffusion models. By separating the denoising process cross timesteps with specialized powerful expert models, this enlarges the overall model capacity while maintaining the same computational cost.

- 👍 **Cinematic-level Aesthetics**: Wan2.2 incorporates meticulously curated aesthetic data, complete with detailed labels for lighting, composition, contrast, color tone, and more. This allows for more precise and controllable cinematic style generation, facilitating the creation of videos with customizable aesthetic preferences.

- 👍 **Complex Motion Generation**: Compared to Wan2.1, Wan2.2 is trained on a significantly larger data, with +65.6% more images and +83.2% more videos. This expansion notably enhances the model's generalization across multiple dimensions such as motions,  semantics, and aesthetics, achieving TOP performance among all open-sourced and closed-sourced models. 

- 👍 **Efficient High-Definition Hybrid TI2V**:  Wan2.2 open-sources a 5B model built with our advanced Wan2.2-VAE that achieves a compression ratio of **16×16×4**. This model supports both text-to-video and image-to-video generation at 720P resolution with 24fps and can also run on consumer-grade graphics cards like 4090. It is one of the fastest **720P@24fps** models currently available, capable of serving both the industrial and academic sectors simultaneously.


## Video Demos

<div align="center">
  <video src="https://github.com/user-attachments/assets/b63bfa58-d5d7-4de6-a1a2-98970b06d9a7" width="70%" poster=""> </video>
</div>

## 🔥 Latest News!!
* Nov 13, 2025: 👋 Wan2.2-Animate-14B has been integrated into Diffusers ([PR](https://github.com/huggingface/diffusers/pull/12526),[Weights](https://huggingface.co/Wan-AI/Wan2.2-Animate-14B-Diffusers)). Thanks to all community contributors. Enjoy!

* Sep 19, 2025: 💃 We introduct **[Wan2.2-Animate-14B](https://humanaigc.github.io/wan-animate)**, an unified model for character animation and replacement with holistic movement and expression replication. We released the [model weights](#model-download) and [inference code](#run-wan-animate). And you can try it on [wan.video](https://wan.video/), [ModelScope Studio](https://www.modelscope.cn/studios/Wan-AI/Wan2.2-Animate) or [HuggingFace Space](https://huggingface.co/spaces/Wan-AI/Wan2.2-Animate)!
* Aug 26, 2025: 🎵 We introduce **[Wan2.2-S2V-14B](https://humanaigc.github.io/wan-s2v-webpage)**, an audio-driven cinematic video generation model, including [inference code](#run-speech-to-video-generation), [model weights](#model-download), and [technical report](https://humanaigc.github.io/wan-s2v-webpage/content/wan-s2v.pdf)! Now you can try it on [wan.video](https://wan.video/),  [ModelScope Gradio](https://www.modelscope.cn/studios/Wan-AI/Wan2.2-S2V) or [HuggingFace Gradio](https://huggingface.co/spaces/Wan-AI/Wan2.2-S2V)!
* Jul 28, 2025: 👋 We have open a [HF space](https://huggingface.co/spaces/Wan-AI/Wan-2.2-5B) using the TI2V-5B model. Enjoy!
* Jul 28, 2025: 👋 Wan2.2 has been integrated into ComfyUI ([CN](https://docs.comfy.org/zh-CN/tutorials/video/wan/wan2_2) | [EN](https://docs.comfy.org/tutorials/video/wan/wan2_2)). Enjoy!
* Jul 28, 2025: 👋 Wan2.2's T2V, I2V and TI2V have been integrated into Diffusers ([T2V-A14B](https://huggingface.co/Wan-AI/Wan2.2-T2V-A14B-Diffusers) | [I2V-A14B](https://huggingface.co/Wan-AI/Wan2.2-I2V-A14B-Diffusers) | [TI2V-5B](https://huggingface.co/Wan-AI/Wan2.2-TI2V-5B-Diffusers)). Feel free to give it a try!
* Jul 28, 2025: 👋 We've released the inference code and model weights of **Wan2.2**.
* Sep 5, 2025: 👋 We add text-to-speech synthesis support with [CosyVoice](https://github.com/FunAudioLLM/CosyVoice) for Speech-to-Video generation task.


## Community Works
If your research or project builds upon [**Wan2.1**](https://github.com/Wan-Video/Wan2.1) or [**Wan2.2**](https://github.com/Wan-Video/Wan2.2), and you would like more people to see it, please inform us.

- [Prompt Relay](https://github.com/GordonChen19/Prompt-Relay), a plug-and-play, inference-time method for temporal control in video generation. Prompt Relay improves video quality and gives users precise control over what happens at each moment in the video. Visit their [webpage](https://gordonchen19.github.io/Prompt-Relay/) for more details.
- [Helios](https://github.com/PKU-YuanGroup/Helios), a breakthrough video generation model base on **Wan2.1** that achieves minute-scale, high-quality video synthesis at 19.5 FPS on a single H100 GPU (about 10 FPS on a single Ascend NPU) —without relying on conventional long video anti-drifting strategies or standard video acceleration techniques. Visit their [webpage](https://pku-yuangroup.github.io/Helios-Page/) for more details.
- [LightX2V](https://github.com/ModelTC/LightX2V), a lightweight and efficient video generation framework that integrates **Wan2.1** and **Wan2.2**, supporting multiple engineering acceleration techniques for fast inference. [LightX2V-HuggingFace](https://huggingface.co/lightx2v), offers a variety of Wan-based step-distillation models, quantized models, and lightweight VAE models.
- [HuMo](https://github.com/Phantom-video/HuMo) proposed a unified, human-centric framework based on **Wan** to produce high-quality, fine-grained, and controllable human videos from multimodal inputs—including text, images, and audio. Visit their [webpage](https://phantom-video.github.io/HuMo/) for more details.
- [FastVideo](https://github.com/hao-ai-lab/FastVideo) includes distilled **Wan** models with sparse attention that significanly speed up the inference time. 
- [Cache-dit](https://github.com/vipshop/cache-dit) offers Fully Cache Acceleration support for **Wan2.2** MoE with DBCache, TaylorSeer and Cache CFG. Visit their [example](https://github.com/vipshop/cache-dit/blob/main/examples/pipeline/run_wan_2.2.py) for more details.
- [Kijai's ComfyUI WanVideoWrapper](https://github.com/kijai/ComfyUI-WanVideoWrapper) is an alternative implementation of **Wan** models for ComfyUI. Thanks to its Wan-only focus, it's on the frontline of getting cutting edge optimizations and hot research features, which are often hard to integrate into ComfyUI quickly due to its more rigid structure.
- [DiffSynth-Studio](https://github.com/modelscope/DiffSynth-Studio) provides comprehensive support for **Wan 2.2**, including low-GPU-memory layer-by-layer offload, FP8 quantization, sequence parallelism, LoRA training, full training.
- [TalkVerse](https://zhenzhiwang.github.io/talkverse/), built on **Wan2.2-TI2V-5B**, enables minute-long audio-driven human video generation. It provides a **[large-scale dataset](https://huggingface.co/datasets/zhenzhiwang/TalkVerse)**, together with [training and inference code and model weights](https://github.com/snap-research/TalkVerse), to support reproducible research and development, especially for researchers with limited computational budgets.


## 📑 Todo List
- Wan2.2 Text-to-Video
    - [x] Multi-GPU Inference code of the A14B and 14B models
    - [x] Checkpoints of the A14B and 14B models
    - [x] ComfyUI integration
    - [x] Diffusers integration
- Wan2.2 Image-to-Video
    - [x] Multi-GPU Inference code of the A14B model
    - [x] Checkpoints of the A14B model
    - [x] ComfyUI integration
    - [x] Diffusers integration
- Wan2.2 Text-Image-to-Video
    - [x] Multi-GPU Inference code of the 5B model
    - [x] Checkpoints of the 5B model
    - [x] ComfyUI integration
    - [x] Diffusers integration
- Wan2.2-S2V Speech-to-Video
    - [x] Inference code of Wan2.2-S2V
    - [x] Checkpoints of Wan2.2-S2V-14B
    - [x] ComfyUI integration
    - [x] Diffusers integration
- Wan2.2-Animate Character Animation and Replacement
    - [x] Inference code of Wan2.2-Animate
    - [x] Checkpoints of Wan2.2-Animate
    - [x] ComfyUI integration
    - [x] Diffusers integration

## Run Wan2.2

#### Installation
Clone the repo:
```sh
git clone https://github.com/Wan-Video/Wan2.2.git
cd Wan2.2
```

Install dependencies:
```sh
# Ensure torch >= 2.4.0
# If the installation of `flash_attn` fails, try installing the other packages first and install `flash_attn` last
pip install -r requirements.txt
# If you want to use CosyVoice to synthesize speech for Speech-to-Video Generation, please install requirements_s2v.txt additionally
pip install -r requirements_s2v.txt
```


#### Model Download

| Models              | Download Links                                                                                                                              | Description |
|--------------------|---------------------------------------------------------------------------------------------------------------------------------------------|-------------|
| T2V-A14B    | 🤗 [Huggingface](https://huggingface.co/Wan-AI/Wan2.2-T2V-A14B)    🤖 [ModelScope](https://modelscope.cn/models/Wan-AI/Wan2.2-T2V-A14B)    | Text-to-Video MoE model, supports 480P & 720P |
| I2V-A14B    | 🤗 [Huggingface](https://huggingface.co/Wan-AI/Wan2.2-I2V-A14B)    🤖 [ModelScope](https://modelscope.cn/models/Wan-AI/Wan2.2-I2V-A14B)    | Image-to-Video MoE model, supports 480P & 720P |
| TI2V-5B     | 🤗 [Huggingface](https://huggingface.co/Wan-AI/Wan2.2-TI2V-5B)     🤖 [ModelScope](https://modelscope.cn/models/Wan-AI/Wan2.2-TI2V-5B)     | High-compression VAE, T2V+I2V, supports 720P |
| S2V-14B     | 🤗 [Huggingface](https://huggingface.co/Wan-AI/Wan2.2-S2V-14B)     🤖 [ModelScope](https://modelscope.cn/models/Wan-AI/Wan2.2-S2V-14B)     | Speech-to-Video model, supports 480P & 720P |
| Animate-14B | 🤗 [Huggingface](https://huggingface.co/Wan-AI/Wan2.2-Animate-14B) 🤖 [ModelScope](https://www.modelscope.cn/models/Wan-AI/Wan2.2-Animate-14B)  | Character animation and replacement | |



> 💡Note: 
> The TI2V-5B model supports 720P video generation at **24 FPS**.


Download models using huggingface-cli:
``` sh
pip install "huggingface_hub[cli]"
huggingface-cli download Wan-AI/Wan2.2-T2V-A14B --local-dir ./Wan2.2-T2V-A14B
```

Download models using modelscope-cli:
``` sh
pip install modelscope
modelscope download Wan-AI/Wan2.2-T2V-A14B --local_dir ./Wan2.2-T2V-A14B
```

#### Run Text-to-Video Generation

This repository supports the `Wan2.2-T2V-A14B` Text-to-Video model and can simultaneously support video generation at 480P and 720P resolutions.


##### (1) Without Prompt Extension

To facilitate implementation, we will start with a basic version of the inference process that skips the [prompt extension](#2-using-prompt-extention) step.

- Single-GPU inference

``` sh
python generate.py  --task t2v-A14B --size 1280*720 --ckpt_dir ./Wan2.2-T2V-A14B --offload_model True --convert_model_dtype --prompt "Two anthropomorphic cats in comfy boxing gear and bright gloves fight intensely on a spotlighted stage."
```

> 💡 This command can run on a GPU with at least 80GB VRAM.

> 💡If you encounter OOM (Out-of-Memory) issues, you can use the `--offload_model True`, `--convert_model_dtype` and `--t5_cpu` options to reduce GPU memory usage.


- Multi-GPU inference using FSDP + DeepSpeed Ulysses

  We use [PyTorch FSDP](https://docs.pytorch.org/docs/stable/fsdp.html) and [DeepSpeed Ulysses](https://arxiv.org/abs/2309.14509) to accelerate inference.


``` sh
torchrun --nproc_per_node=8 generate.py --task t2v-A14B --size 1280*720 --ckpt_dir ./Wan2.2-T2V-A14B --dit_fsdp --t5_fsdp --ulysses_size 8 --prompt "Two anthropomorphic cats in comfy boxing gear and bright gloves fight intensely on a spotlighted stage."
```


##### (2) Using Prompt Extension

Extending the prompts can effectively enrich the details in the generated videos, further enhancing the video quality. Therefore, we recommend enabling prompt extension. We provide the following two methods for prompt extension:

- Use the Dashscope API for extension.
  - Apply for a `dashscope.api_key` in advance ([EN](https://www.alibabacloud.com/help/en/model-studio/getting-started/first-api-call-to-qwen) | [CN](https://help.aliyun.com/zh/model-studio/getting-started/first-api-call-to-qwen)).
  - Configure the environment variable `DASH_API_KEY` to specify the Dashscope API key. For users of Alibaba Cloud's international site, you also need to set the environment variable `DASH_API_URL` to 'https://dashscope-intl.aliyuncs.com/api/v1'. For more detailed instructions, please refer to the [dashscope document](https://www.alibabacloud.com/help/en/model-studio/developer-reference/use-qwen-by-calling-api?spm=a2c63.p38356.0.i1).
  - Use the `qwen-plus` model for text-to-video tasks and `qwen-vl-max` for image-to-video tasks.
  - You can modify the model used for extension with the parameter `--prompt_extend_model`. For example:
```sh
DASH_API_KEY=your_key torchrun --nproc_per_node=8 generate.py  --task t2v-A14B --size 1280*720 --ckpt_dir ./Wan2.2-T2V-A14B --dit_fsdp --t5_fsdp --ulysses_size 8 --prompt "Two anthropomorphic cats in comfy boxing gear and bright gloves fight intensely on a spotlighted stage" --use_prompt_extend --prompt_extend_method 'dashscope' --prompt_extend_target_lang 'zh'
```

- Using a local model for extension.

  - By default, the Qwen model on HuggingFace is used for this extension. Users can choose Qwen models or other models based on the available GPU memory size.
  - For text-to-video tasks, you can use models like `Qwen/Qwen2.5-14B-Instruct`, `Qwen/Qwen2.5-7B-Instruct` and `Qwen/Qwen2.5-3B-Instruct`.
  - For image-to-video tasks, you can use models like `Qwen/Qwen2.5-VL-7B-Instruct` and `Qwen/Qwen2.5-VL-3B-Instruct`.
  - Larger models generally provide better extension results but require more GPU memory.
  - You can modify the model used for extension with the parameter `--prompt_extend_model` , allowing you to specify either a local model path or a Hugging Face model. For example:

``` sh
torchrun --nproc_per_node=8 generate.py  --task t2v-A14B --size 1280*720 --ckpt_dir ./Wan2.2-T2V-A14B --dit_fsdp --t5_fsdp --ulysses_size 8 --prompt "Two anthropomorphic cats in comfy boxing gear and bright gloves fight intensely on a spotlighted stage" --use_prompt_extend --prompt_extend_method 'local_qwen' --prompt_extend_target_lang 'zh'
```


#### Run Image-to-Video Generation

This repository supports the `Wan2.2-I2V-A14B` Image-to-Video model and can simultaneously support video generation at 480P and 720P resolutions.


- Single-GPU inference
```sh
python generate.py --task i2v-A14B --size 1280*720 --ckpt_dir ./Wan2.2-I2V-A14B --offload_model True --convert_model_dtype --image examples/i2v_input.JPG --prompt "Summer beach vacation style, a white cat wearing sunglasses sits on a surfboard. The fluffy-furred feline gazes directly at the camera with a relaxed expression. Blurred beach scenery forms the background featuring crystal-clear waters, distant green hills, and a blue sky dotted with white clouds. The cat assumes a naturally relaxed posture, as if savoring the sea breeze and warm sunlight. A close-up shot highlights the feline's intricate details and the refreshing atmosphere of the seaside."
```

> This command can run on a GPU with at least 80GB VRAM.

> 💡For the Image-to-Video task, the `size` parameter represents the area of the generated video, with the aspect ratio following that of the original input image.


- Multi-GPU inference using FSDP + DeepSpeed Ulysses

```sh
torchrun --nproc_per_node=8 generate.py --task i2v-A14B --size 1280*720 --ckpt_dir ./Wan2.2-I2V-A14B --image examples/i2v_input.JPG --dit_fsdp --t5_fsdp --ulysses_size 8 --prompt "Summer beach vacation style, a white cat wearing sunglasses sits on a surfboard. The fluffy-furred feline gazes directly at the camera with a relaxed expression. Blurred beach scenery forms the background featuring crystal-clear waters, distant green hills, and a blue sky dotted with white clouds. The cat assumes a naturally relaxed posture, as if savoring the sea breeze and warm sunlight. A close-up shot highlights the feline's intricate details and the refreshing atmosphere of the seaside."
```

- Image-to-Video Generation without prompt

```sh
DASH_API_KEY=your_key torchrun --nproc_per_node=8 generate.py --task i2v-A14B --size 1280*720 --ckpt_dir ./Wan2.2-I2V-A14B --prompt '' --image examples/i2v_input.JPG --dit_fsdp --t5_fsdp --ulysses_size 8 --use_prompt_extend --prompt_extend_method 'dashscope'
```

> 💡The model can generate videos solely from the input image. You can use prompt extension to generate prompt from the image.

> The process of prompt extension can be referenced [here](#2-using-prompt-extention).

#### Run Text-Image-to-Video Generation

This repository supports the `Wan2.2-TI2V-5B` Text-Image-to-Video model and can support video generation at 720P resolutions.


- Single-GPU Text-to-Video inference
```sh
python generate.py --task ti2v-5B --size 1280*704 --ckpt_dir ./Wan2.2-TI2V-5B --offload_model True --convert_model_dtype --t5_cpu --prompt "Two anthropomorphic cats in comfy boxing gear and bright gloves fight intensely on a spotlighted stage"
```

> 💡Unlike other tasks, the 720P resolution of the Text-Image-to-Video task is `1280*704` or `704*1280`.

> This command can run on a GPU with at least 24GB VRAM (e.g, RTX 4090 GPU).

> 💡If you are running on a GPU with at least 80GB VRAM, you can remove the `--offload_model True`, `--convert_model_dtype` and `--t5_cpu` options to speed up execution.


- Single-GPU Image-to-Video inference
```sh
python generate.py --task ti2v-5B --size 1280*704 --ckpt_dir ./Wan2.2-TI2V-5B --offload_model True --convert_model_dtype --t5_cpu --image examples/i2v_input.JPG --prompt "Summer beach vacation style, a white cat wearing sunglasses sits on a surfboard. The fluffy-furred feline gazes directly at the camera with a relaxed expression. Blurred beach scenery forms the background featuring crystal-clear waters, distant green hills, and a blue sky dotted with white clouds. The cat assumes a naturally relaxed posture, as if savoring the sea breeze and warm sunlight. A close-up shot highlights the feline's intricate details and the refreshing atmosphere of the seaside."
```

> 💡If the image parameter is configured, it is an Image-to-Video generation; otherwise, it defaults to a Text-to-Video generation.

> 💡Similar to Image-to-Video, the `size` parameter represents the area of the generated video, with the aspect ratio following that of the original input image.


- Multi-GPU inference using FSDP + DeepSpeed Ulysses

```sh
torchrun --nproc_per_node=8 generate.py --task ti2v-5B --size 1280*704 --ckpt_dir ./Wan2.2-TI2V-5B --dit_fsdp --t5_fsdp --ulysses_size 8 --image examples/i2v_input.JPG --prompt "Summer beach vacation style, a white cat wearing sunglasses sits on a surfboard. The fluffy-furred feline gazes directly at the camera with a relaxed expression. Blurred beach scenery forms the background featuring crystal-clear waters, distant green hills, and a blue sky dotted with white clouds. The cat assumes a naturally relaxed posture, as if savoring the sea breeze and warm sunlight. A close-up shot highlights the feline's intricate details and the refreshing atmosphere of the seaside."
```

> The process of prompt extension can be referenced [here](#2-using-prompt-extention).

#### Run Speech-to-Video Generation

This repository supports the `Wan2.2-S2V-14B` Speech-to-Video model and can simultaneously support video generation at 480P and 720P resolutions.

- Single-GPU Speech-to-Video inference

```sh
python generate.py  --task s2v-14B --size 1024*704 --ckpt_dir ./Wan2.2-S2V-14B/ --offload_model True --convert_model_dtype --prompt "Summer beach vacation style, a white cat wearing sunglasses sits on a surfboard."  --image "examples/i2v_input.JPG" --audio "examples/talk.wav"
# Without setting --num_clip, the generated video length will automatically adjust based on the input audio length

# You can use CosyVoice to generate audio with --enable_tts
python generate.py  --task s2v-14B --size 1024*704 --ckpt_dir ./Wan2.2-S2V-14B/ --offload_model True --convert_model_dtype --prompt "Summer beach vacation style, a white cat wearing sunglasses sits on a surfboard."  --image "examples/i2v_input.JPG" --enable_tts --tts_prompt_audio "examples/zero_shot_prompt.wav" --tts_prompt_text "希望你以后能够做的比我还好呦。" --tts_text "收到好友从远方寄来的生日礼物，那份意外的惊喜与深深的祝福让我心中充满了甜蜜的快乐，笑容如花儿般绽放。"
```

> 💡 This command can run on a GPU with at least 80GB VRAM.

- Multi-GPU inference using FSDP + DeepSpeed Ulysses

```sh
torchrun --nproc_per_node=8 generate.py --task s2v-14B --size 1024*704 --ckpt_dir ./Wan2.2-S2V-14B/ --dit_fsdp --t5_fsdp --ulysses_size 8 --prompt "Summer beach vacation style, a white cat wearing sunglasses sits on a surfboard." --image "examples/i2v_input.JPG" --audio "examples/talk.wav"
```

- Pose + Audio driven generation

```sh
torchrun --nproc_per_node=8 generate.py --task s2v-14B --size 1024*704 --ckpt_dir ./Wan2.2-S2V-14B/ --dit_fsdp --t5_fsdp --ulysses_size 8 --prompt "a person is singing" --image "examples/pose.png" --audio "examples/sing.MP3" --pose_video "./examples/pose.mp4" 
```

> 💡For the Speech-to-Video task, the `size` parameter represents the area of the generated video, with the aspect ratio following that of the original input image.

> 💡The model can generate videos from audio input combined with reference image and optional text prompt.

> 💡The `--pose_video` parameter enables pose-driven generation, allowing the model to follow specific pose sequences while generating videos synchronized with audio input.

> 💡The `--num_clip` parameter controls the number of video clips generated, useful for quick preview with shorter generation time.

Please visit our project page to see more examples and learn about the scenarios suitable for this model.

#### Run Wan-Animate 

Wan-Animate takes a video and a character image as input, and generates a video in either "animation" or "replacement" mode. 

1. animation mode： The model generates a video of the character image that mimics the human motion in the input video.
2. replacement mode: The model replaces the character image with the input video.

Please visit our [project page](https://humanaigc.github.io/wan-animate) to see more examples and learn about the scenarios suitable for this model.

##### (1) Preprocessing 
The input video should be preprocessed into several materials before be feed into the inference process.  Please refer to the following processing flow, and more details about preprocessing can be found in [UserGuider](https://github.com/Wan-Video/Wan2.2/blob/main/wan/modules/animate/preprocess/UserGuider.md).

* For animation
```bash
python ./wan/modules/animate/preprocess/preprocess_data.py \
    --ckpt_path ./Wan2.2-Animate-14B/process_checkpoint \
    --video_path ./examples/wan_animate/animate/video.mp4 \
    --refer_path ./examples/wan_animate/animate/image.jpeg \
    --save_path ./examples/wan_animate/animate/process_results \
    --resolution_area 1280 720 \
    --retarget_flag \
    --use_flux
```
* For replacement
```bash
python ./wan/modules/animate/preprocess/preprocess_data.py \
    --ckpt_path ./Wan2.2-Animate-14B/process_checkpoint \
    --video_path ./examples/wan_animate/replace/video.mp4 \
    --refer_path ./examples/wan_animate/replace/image.jpeg \
    --save_path ./examples/wan_animate/replace/process_results \
    --resolution_area 1280 720 \
    --iterations 3 \
    --k 7 \
    --w_len 1 \
    --h_len 1 \
    --replace_flag
```
##### (2) Run in animation mode 

* Single-GPU inference 

```bash
python generate.py --task animate-14B --ckpt_dir ./Wan2.2-Animate-14B/ --src_root_path ./examples/wan_animate/animate/process_results/ --refert_num 1
```

* Multi-GPU inference using FSDP + DeepSpeed Ulysses

```bash
python -m torch.distributed.run --nnodes 1 --nproc_per_node 8 generate.py --task animate-14B --ckpt_dir ./Wan2.2-Animate-14B/ --src_root_path ./examples/wan_animate/animate/process_results/ --refert_num 1 --dit_fsdp --t5_fsdp --ulysses_size 8
```

* Diffusers Pipeline

```python
from diffusers import WanAnimatePipeline
from diffusers.utils import export_to_video, load_image, load_video

device = "cuda:0"
dtype = torch.bfloat16
model_id = "Wan-AI/Wan2.2-Animate-14B-Diffusers"
pipe = WanAnimatePipeline.from_pretrained(model_id torch_dtype=dtype)
pipe.to(device)

seed = 42
prompt = "People in the video are doing actions."

# Animation
image = load_image("/path/to/animate/reference/image/src_ref.png")
pose_video = load_video("/path/to/animate/pose/video/src_pose.mp4")
face_video = load_video("/path/to/animate/face/video/src_face.mp4")

animate_video = pipe(
    image=image,
    pose_video=pose_video,
    face_video=face_video,
    prompt=prompt,
    mode="animate",
    segment_frame_length=77,  # clip_len in original code
    prev_segment_conditioning_frames=1,  # refert_num in original code
    guidance_scale=1.0,
    num_inference_steps=20,
    generator=torch.Generator(device=device).manual_seed(seed),
).frames[0]
export_to_video(animate_video, "diffusers_animate.mp4", fps=30)
```

##### (3) Run in replacement mode 

* Single-GPU inference 

```bash
python generate.py --task animate-14B --ckpt_dir ./Wan2.2-Animate-14B/ --src_root_path ./examples/wan_animate/replace/process_results/ --refert_num 1 --replace_flag --use_relighting_lora 
```

* Multi-GPU inference using FSDP + DeepSpeed Ulysses

```bash
python -m torch.distributed.run --nnodes 1 --nproc_per_node 8 generate.py --task animate-14B --ckpt_dir ./Wan2.2-Animate-14B/ --src_root_path ./examples/wan_animate/replace/process_results/src_pose.mp4  --refert_num 1 --replace_flag --use_relighting_lora --dit_fsdp --t5_fsdp --ulysses_size 8
```

* Diffusers Pipeline

```python
# create pipeline as in the Animation code ☝️

# Replacement
image = load_image("/path/to/replace/reference/image/src_ref.png")
pose_video = load_video("/path/to/replace/pose/video/src_pose.mp4")
face_video = load_video("/path/to/replace/face/video/src_face.mp4")
background_video = load_video("/path/to/replace/background/video/src_bg.mp4")
mask_video = load_video("/path/to/replace/mask/video/src_mask.mp4")

replace_video = pipe(
    image=image,
    pose_video=pose_video,
    face_video=face_video,
    background_video=background_video,
    mask_video=mask_video,
    prompt=prompt,
    mode="replace",
    segment_frame_length=77,  # clip_len in original code
    prev_segment_conditioning_frames=1,  # refert_num in original code
    guidance_scale=1.0,
    num_inference_steps=20,
    generator=torch.Generator(device=device).manual_seed(seed),
).frames[0]
export_to_video(replace_video, "diffusers_replace.mp4", fps=30)
```

> 💡 If you're using **Wan-Animate**, we do not recommend using LoRA models trained on `Wan2.2`, since weight changes during training may lead to unexpected behavior.

## Computational Efficiency on Different GPUs

We test the computational efficiency of different **Wan2.2** models on different GPUs in the following table. The results are presented in the format: **Total time (s) / peak GPU memory (GB)**.


<div align="center">
    <img src="assets/comp_effic.png" alt="" style="width: 80%;" />
</div>

> The parameter settings for the tests presented in this table are as follows:
> (1) Multi-GPU: 14B: `--ulysses_size 4/8 --dit_fsdp --t5_fsdp`, 5B: `--ulysses_size 4/8 --offload_model True --convert_model_dtype --t5_cpu`; Single-GPU: 14B: `--offload_model True --convert_model_dtype`, 5B: `--offload_model True --convert_model_dtype --t5_cpu`
(--convert_model_dtype converts model parameter types to config.param_dtype);
> (2) The distributed testing utilizes the built-in FSDP and Ulysses implementations, with FlashAttention3 deployed on Hopper architecture GPUs;
> (3) Tests were run without the `--use_prompt_extend` flag;
> (4) Reported results are the average of multiple samples taken after the warm-up phase.


-------

## Introduction of Wan2.2

**Wan2.2** builds on the foundation of Wan2.1 with notable improvements in generation quality and model capability. This upgrade is driven by a series of key technical innovations, mainly including the Mixture-of-Experts (MoE) architecture, upgraded training data, and high-compression video generation.

##### (1) Mixture-of-Experts (MoE) Architecture

Wan2.2 introduces Mixture-of-Experts (MoE) architecture into the video generation diffusion model. MoE has been widely validated in large language models as an efficient approach to increase total model parameters while keeping inference cost nearly unchanged. In Wan2.2, the A14B model series adopts a two-expert design tailored to the denoising process of diffusion models: a high-noise expert for the early stages, focusing on overall layout; and a low-noise expert for the later stages, refining video details. Each expert model has about 14B parameters, resulting in a total of 27B parameters but only 14B active parameters per step, keeping inference computation and GPU memory nearly unchanged.

<div align="center">
    <img src="assets/moe_arch.png" alt="" style="width: 90%;" />
</div>

The transition point between the two experts is determined by the signal-to-noise ratio (SNR), a metric that decreases monotonically as the denoising step $t$ increases. At the beginning of the denoising process, $t$ is large and the noise level is high, so the SNR is at its minimum, denoted as ${SNR}_{min}$. In this stage, the high-noise expert is activated. We define a threshold step ${t}_{moe}$ corresponding to half of the ${SNR}_{min}$, and switch to the low-noise expert when $t<{t}_{moe}$.

<div align="center">
    <img src="assets/moe_2.png" alt="" style="width: 90%;" />
</div>

To validate the effectiveness of the MoE architecture, four settings are compared based on their validation loss curves. The baseline **Wan2.1** model does not employ the MoE architecture. Among the MoE-based variants, the **Wan2.1 & High-Noise Expert** reuses the Wan2.1 model as the low-noise expert while uses the  Wan2.2's high-noise expert, while the **Wan2.1 & Low-Noise Expert** uses Wan2.1 as the high-noise expert and employ the Wan2.2's low-noise expert. The **Wan2.2 (MoE)** (our final version) achieves the lowest validation loss, indicating that its generated video distribution is closest to ground-truth and exhibits superior convergence.


##### (2) Efficient High-Definition Hybrid TI2V
To enable more efficient deployment, Wan2.2 also explores a high-compression design. In addition to the 27B MoE models, a 5B dense model, i.e., TI2V-5B, is released. It is supported by a high-compression Wan2.2-VAE, which achieves a $T\times H\times W$ compression ratio of $4\times16\times16$, increasing the overall compression rate to 64 while maintaining high-quality video reconstruction. With an additional patchification layer, the total compression ratio of TI2V-5B reaches $4\times32\times32$. Without specific optimization, TI2V-5B can generate a 5-second 720P video in under 9 minutes on a single consumer-grade GPU, ranking among the fastest 720P@24fps video generation models. This model also natively supports both text-to-video and image-to-video tasks within a single unified framework, covering both academic research and practical applications.


<div align="center">
    <img src="assets/vae.png" alt="" style="width: 80%;" />
</div>



##### Comparisons to SOTAs
We compared Wan2.2 with leading closed-source commercial models on our new Wan-Bench 2.0, evaluating performance across multiple crucial dimensions. The results demonstrate that Wan2.2 achieves superior performance compared to these leading models.


<div align="center">
    <img src="assets/performance.png" alt="" style="width: 90%;" />
</div>

## Citation
If you find our work helpful, please cite us.

```
@article{wan2025,
      title={Wan: Open and Advanced Large-Scale Video Generative Models}, 
      author={Team Wan and Ang Wang and Baole Ai and Bin Wen and Chaojie Mao and Chen-Wei Xie and Di Chen and Feiwu Yu and Haiming Zhao and Jianxiao Yang and Jianyuan Zeng and Jiayu Wang and Jingfeng Zhang and Jingren Zhou and Jinkai Wang and Jixuan Chen and Kai Zhu and Kang Zhao and Keyu Yan and Lianghua Huang and Mengyang Feng and Ningyi Zhang and Pandeng Li and Pingyu Wu and Ruihang Chu and Ruili Feng and Shiwei Zhang and Siyang Sun and Tao Fang and Tianxing Wang and Tianyi Gui and Tingyu Weng and Tong Shen and Wei Lin and Wei Wang and Wei Wang and Wenmeng Zhou and Wente Wang and Wenting Shen and Wenyuan Yu and Xianzhong Shi and Xiaoming Huang and Xin Xu and Yan Kou and Yangyu Lv and Yifei Li and Yijing Liu and Yiming Wang and Yingya Zhang and Yitong Huang and Yong Li and You Wu and Yu Liu and Yulin Pan and Yun Zheng and Yuntao Hong and Yupeng Shi and Yutong Feng and Zeyinzi Jiang and Zhen Han and Zhi-Fan Wu and Ziyu Liu},
      journal = {arXiv preprint arXiv:2503.20314},
      year={2025}
}
```

## License Agreement
The models in this repository are licensed under the Apache 2.0 License. We claim no rights over the your generated contents, granting you the freedom to use them while ensuring that your usage complies with the provisions of this license. You are fully accountable for your use of the models, which must not involve sharing any content that violates applicable laws, causes harm to individuals or groups, disseminates personal information intended for harm, spreads misinformation, or targets vulnerable populations. For a complete list of restrictions and details regarding your rights, please refer to the full text of the [license](LICENSE.txt).


## Acknowledgements

We would like to thank the contributors to the [SD3](https://huggingface.co/stabilityai/stable-diffusion-3-medium), [Qwen](https://huggingface.co/Qwen), [umt5-xxl](https://huggingface.co/google/umt5-xxl), [diffusers](https://github.com/huggingface/diffusers) and [HuggingFace](https://huggingface.co) repositories, for their open research.



## Contact Us
If you would like to leave a message to our research or product teams, feel free to join our [Discord](https://discord.gg/AKNgpMK4Yj) or [WeChat groups](https://gw.alicdn.com/imgextra/i2/O1CN01tqjWFi1ByuyehkTSB_!!6000000000015-0-tps-611-1279.jpg)!


</details>
