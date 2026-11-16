# Use Wan 2.2

**Modified by Transformaiton.com.** This file was added by [Transformaiton.com](https://transformaiton.com/). It explains how to run the Wan team's code in this fork. Transformaiton.com did not change the Python, and it is not affiliated with or endorsed by the Wan team, Alibaba Group, or Alibaba Cloud.

Install first: [INSTALL.md](INSTALL.md). The original technical README is preserved in the [README](../README.md#original-project-documentation).

Every still, clip, and timing number below comes from this repository or from the Wan team's project pages. This guide does not add generated samples.

## What you get

Wan 2.2 generates a short video from text, from a still image, from speech plus an image, or by transferring motion onto a character. The default length is a few seconds:

| Model | Default length in the config | Frame rate |
| --- | --- | --- |
| TI2V-5B | 121 frames (about 5 seconds) | 24 fps |
| T2V-A14B, I2V-A14B, S2V-14B | 81 frames (about 5 seconds) | 16 fps |
| Animate-14B | 77 frames | 30 fps |

`frame_num` has to be `4n+1` (5, 9, 13, …). A longer clip uses more memory and more time. The original README's speed claim for TI2V-5B is a 5-second 720p video.

## Which model to run

| Task flag | Checkpoint | Use it when |
| --- | --- | --- |
| `ti2v-5B` | `./Wan2.2-TI2V-5B` | Text-to-video or image-to-video at 720p. This is the small model and the right first run. |
| `t2v-A14B` | `./Wan2.2-T2V-A14B` | Text-to-video at 480p or 720p on a high-memory GPU. |
| `i2v-A14B` | `./Wan2.2-I2V-A14B` | Image-to-video at 480p or 720p on a high-memory GPU. |
| `s2v-14B` | `./Wan2.2-S2V-14B` | Speech-to-video. Needs an image and an audio file, or the optional text-to-speech extras. |
| `animate-14B` | `./Wan2.2-Animate-14B` | Character animation or replacement. Needs a preprocessing step first. |

Supported sizes are fixed in `wan/configs/__init__.py`:

| Task | Sizes you can pass to `--size` |
| --- | --- |
| `t2v-A14B`, `i2v-A14B` | `1280*720`, `720*1280`, `832*480`, `480*832` |
| `ti2v-5B` | `1280*704`, `704*1280` |
| `s2v-14B` | the 480p and 720p sizes above, plus `1024*704`, `704*1024`, `1280*704`, `704*1280` |
| `animate-14B` | `1280*720`, `720*1280` |

For image-to-video, `--size` sets the pixel area. The output aspect ratio follows the input image. The original README states that specifically.

## Quick start: TI2V-5B

Run these from the repository root, with the virtual environment active and `./Wan2.2-TI2V-5B` already downloaded. Quote the size. In bash an unquoted `*` is a wildcard.

### Text to video

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

That prompt is the official TI2V-5B example from the original README and from `generate.py`. Portrait orientation uses `'704*1280'` instead of `'1280*704'`.

### Image to video, using the official example photo

The still shipped for this command is `examples/i2v_input.JPG`: a white cat in sunglasses on a surfboard.

<img src="../examples/i2v_input.JPG" alt="Official Wan 2.2 example input: a white cat in sunglasses on a surfboard" width="480"/>

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

The prompt and the photo are the official image-to-video example. Passing `--image` switches TI2V-5B into image-to-video. Omitting `--image` keeps it as text-to-video.

### A product photo

Replace the image path and rewrite the prompt. This is a command to edit and run. This repository does not contain a rendered result for it.

```bash
python generate.py \
  --task ti2v-5B \
  --size '1280*704' \
  --ckpt_dir ./Wan2.2-TI2V-5B \
  --offload_model True \
  --convert_model_dtype \
  --t5_cpu \
  --image path/to/your-product-photo.jpg \
  --save_file product-clip.mp4 \
  --prompt "A handmade ceramic mug on a wooden table, soft daylight, tidy workshop background. The camera moves slowly closer. No text, no logo animation."
```

Use a clear photo of the real product. Describe one camera move and one setting. The model follows the prompt and the picture together, and it can change details, so check the clip before you publish it.

### Where the file goes

If you pass `--save_file`, the video is written there. If you do not, `generate.py` writes an `.mp4` in the current directory. The name starts with the task and the size, then a slice of the prompt, then a timestamp. The log line is `Saving generated video to ...`.

On a single RTX 4090, the Wan team's table measures about 9 minutes for one 720p TI2V-5B clip with these memory flags (534.7 seconds for text-to-video, 524.8 seconds for image-to-video), after warm-up. The first run is slower because the weights are loading.

### What those flags do

| Flag | Why the quick start sets it |
| --- | --- |
| `--offload_model True` | Moves model weights off the GPU between steps. This is how the 24 GB command fits. |
| `--convert_model_dtype` | Stores weights in the dtype from the config (`bfloat16` in the shared config). |
| `--t5_cpu` | Keeps the text encoder on CPU so the GPU can hold the video model. |
| `--ckpt_dir` | Folder from the download step. Required. |
| `--save_file` | Optional output path. |

On a GPU with at least 80 GB, the original README says you can drop `--offload_model True`, `--convert_model_dtype`, and `--t5_cpu` to run faster.

## Prompts for a small business

Write the prompt as a shot list: subject, motion, camera, light, background. One action per clip. The official examples are the pattern to copy.

- **Product video.** Name the product, the surface it sits on, one camera move, and the light. Say "no text" if you do not want lettering drawn into the frames.
- **Social ad.** One product, one action, one place, and a short camera move. A 5-second clip is a shot inside an ad, not a finished 15-second spot with a voiceover and titles. Cut it together in your editor.
- **Animate a product photo.** Pass the photo as `--image`. Describe a small motion that keeps the product recognizable. A slow push-in or a turn of the object is easier to review than a scene change.

Prompt extension is optional. The original README documents two methods, DashScope (`DASH_API_KEY`) and a local Qwen model (`--use_prompt_extend --prompt_extend_method local_qwen`). Both add another model and more memory. The quick start leaves extension off.

## Other tasks

The commands below are the single-GPU forms from the original README. Quote sizes when you paste them into bash. Multi-GPU forms, prompt extension, and the Diffusers snippets are in the [original project documentation](../README.md#original-project-documentation) and in the [upstream README](https://github.com/Wan-Video/Wan2.2).

### Text-to-video, T2V-A14B

```bash
python generate.py \
  --task t2v-A14B \
  --size '1280*720' \
  --ckpt_dir ./Wan2.2-T2V-A14B \
  --offload_model True \
  --convert_model_dtype \
  --prompt "Two anthropomorphic cats in comfy boxing gear and bright gloves fight intensely on a spotlighted stage."
```

The original README says this command can run on a GPU with at least 80 GB of VRAM. For 480p, use `'832*480'` or `'480*832'`.

### Image-to-video, I2V-A14B

```bash
python generate.py \
  --task i2v-A14B \
  --size '1280*720' \
  --ckpt_dir ./Wan2.2-I2V-A14B \
  --offload_model True \
  --convert_model_dtype \
  --image examples/i2v_input.JPG \
  --prompt "Summer beach vacation style, a white cat wearing sunglasses sits on a surfboard. The fluffy-furred feline gazes directly at the camera with a relaxed expression. Blurred beach scenery forms the background featuring crystal-clear waters, distant green hills, and a blue sky dotted with white clouds. The cat assumes a naturally relaxed posture, as if savoring the sea breeze and warm sunlight. A close-up shot highlights the feline's intricate details and the refreshing atmosphere of the seaside."
```

Same 80 GB note as T2V-A14B. The picture is the official example input.

### Speech-to-video, S2V-14B

Install `requirements_s2v.txt` first. The official single-GPU command uses the cat photo and `examples/talk.wav`:

```bash
python generate.py \
  --task s2v-14B \
  --size '1024*704' \
  --ckpt_dir ./Wan2.2-S2V-14B/ \
  --offload_model True \
  --convert_model_dtype \
  --prompt "Summer beach vacation style, a white cat wearing sunglasses sits on a surfboard." \
  --image examples/i2v_input.JPG \
  --audio examples/talk.wav
```

The original README says this needs at least 80 GB of VRAM. Without `--num_clip`, the video length follows the audio. The pose-driven command in the original README uses `examples/pose.png`, `examples/sing.MP3`, and `examples/pose.mp4`. Text-to-speech through CosyVoice is the `--enable_tts` path in that same section.

More speech-to-video examples are on the Wan team's page: [humanaigc.github.io/wan-s2v-webpage](https://humanaigc.github.io/wan-s2v-webpage).

### Character animation, Animate-14B

Animation takes a reference still and a motion video. Replacement takes a performance video and swaps in the character from the still. Both need a preprocessing pass. The inputs shipped in this repo are:

| Mode | Still | Video |
| --- | --- | --- |
| Animation | [examples/wan_animate/animate/image.jpeg](../examples/wan_animate/animate/image.jpeg) | [examples/wan_animate/animate/video.mp4](../examples/wan_animate/animate/video.mp4) |
| Replacement | [examples/wan_animate/replace/image.jpeg](../examples/wan_animate/replace/image.jpeg) | [examples/wan_animate/replace/video.mp4](../examples/wan_animate/replace/video.mp4) |

<img src="../examples/wan_animate/animate/image.jpeg" alt="Official Wan-Animate reference still" width="360"/>

<img src="../examples/wan_animate/replace/image.jpeg" alt="Official Wan-Animate replacement reference still" width="360"/>

The preprocessing and inference commands are long, and they depend on extra checkpoints under `./Wan2.2-Animate-14B/process_checkpoint`. Use the original README section "Run Wan-Animate" and the preprocessor notes in [wan/modules/animate/preprocess/UserGuider.md](../wan/modules/animate/preprocess/UserGuider.md). The Wan team's project page has further official examples: [humanaigc.github.io/wan-animate](https://humanaigc.github.io/wan-animate).

The original README says not to load LoRA weights trained on other Wan 2.2 models into Wan-Animate.

## Official examples already in the project

These files are inputs and demos from the Wan team. They are not outputs produced by Transformaiton.com.

| File | Role in the upstream commands |
| --- | --- |
| [examples/i2v_input.JPG](../examples/i2v_input.JPG) | Image-to-video and speech-to-video still. |
| [examples/talk.wav](../examples/talk.wav) | Speech-to-video audio. |
| [examples/zero_shot_prompt.wav](../examples/zero_shot_prompt.wav) | Voice prompt for the CosyVoice path. |
| [examples/pose.png](../examples/pose.png), [examples/pose.mp4](../examples/pose.mp4), [examples/sing.MP3](../examples/sing.MP3) | Pose-driven speech-to-video example. |
| [examples/Five Hundred Miles.png](<../examples/Five Hundred Miles.png>), [examples/Five Hundred Miles.MP3](<../examples/Five Hundred Miles.MP3>) | Extra example media shipped in `examples/`. The original README does not reference them in a command. |
| `examples/wan_animate/animate/` and `examples/wan_animate/replace/` | Animation and replacement inputs. |

The demo video embedded in the original README is hosted by the Wan project:

[https://github.com/user-attachments/assets/b63bfa58-d5d7-4de6-a1a2-98970b06d9a7](https://github.com/user-attachments/assets/b63bfa58-d5d7-4de6-a1a2-98970b06d9a7)

Further official demos: [wan.video](https://wan.video), the [TI2V-5B Hugging Face Space](https://huggingface.co/spaces/Wan-AI/Wan-2.2-5B), the [Animate Space](https://huggingface.co/spaces/Wan-AI/Wan2.2-Animate), and the [speech-to-video Space](https://huggingface.co/spaces/Wan-AI/Wan2.2-S2V).

## Hardware

The figure below is the Wan team's table, `assets/comp_effic.png`. Cells are **time in seconds / peak GPU memory in GB**. The test notes under the figure are from the original README and apply to every number.

<img src="../assets/comp_effic.png" alt="Wan team table: generation time in seconds and peak GPU memory in GB for TI2V-5B, T2V-A14B, and I2V-A14B" width="100%"/>

How the Wan team measured that table:

1. Multi-GPU 14B: `--ulysses_size 4/8 --dit_fsdp --t5_fsdp`. Multi-GPU 5B: `--ulysses_size 4/8 --offload_model True --convert_model_dtype --t5_cpu`. Single-GPU 14B: `--offload_model True --convert_model_dtype`. Single-GPU 5B: `--offload_model True --convert_model_dtype --t5_cpu`.
2. Distributed tests use the built-in FSDP and Ulysses implementations, with FlashAttention3 on Hopper GPUs.
3. Prompt extension was off.
4. Numbers are averages of multiple samples after warm-up.

Read the practical limits from that table and from the original README together:

- **TI2V-5B, 720p, one RTX 4090.** About 22.9 GB and 535 seconds for text-to-video, about 22.8 GB and 525 seconds for image-to-video. The written minimum for this command is 24 GB.
- **T2V-A14B and I2V-A14B, one GPU.** The published peaks are about 41 GB at 480p and about 60 GB at 720p on an H20, A100/A800, or H100/H800, with offload on. The original README still says the single-GPU example needs at least 80 GB. Plan for 80 GB.
- **The same 14B models on 4 GPUs.** About 41 GB per GPU at 480p and about 52 GB per GPU at 720p.
- **The same 14B models on 8 GPUs.** About 26 GB per GPU at 480p and about 37 GB per GPU at 720p.
- **S2V-14B.** Not in the table. The original README says at least 80 GB for the single-GPU command.
- **Animate-14B.** Not in the table, and the original README does not publish a VRAM figure. Treat it as a 14B job and prefer a high-memory GPU or the hosted demo.

An RTX 4090 row exists only for TI2V-5B. The table does not show the 14B models on a 24 GB card.

## Cheaper ways to run a clip

You do not have to own the GPU.

**Hosted demos already linked by the Wan team**

- TI2V-5B in the browser: [huggingface.co/spaces/Wan-AI/Wan-2.2-5B](https://huggingface.co/spaces/Wan-AI/Wan-2.2-5B)
- Wan's own site: [wan.video](https://wan.video)
- ComfyUI's Wan 2.2 tutorial: [English](https://docs.comfy.org/tutorials/video/wan/wan2_2), [Chinese](https://docs.comfy.org/zh-CN/tutorials/video/wan/wan2_2)
- Diffusers weights, if you already run a Diffusers pipeline: [TI2V-5B](https://huggingface.co/Wan-AI/Wan2.2-TI2V-5B-Diffusers), [T2V-A14B](https://huggingface.co/Wan-AI/Wan2.2-T2V-A14B-Diffusers), [I2V-A14B](https://huggingface.co/Wan-AI/Wan2.2-I2V-A14B-Diffusers), [Animate-14B](https://huggingface.co/Wan-AI/Wan2.2-Animate-14B-Diffusers)
- Speech-to-video demos: [Hugging Face](https://huggingface.co/spaces/Wan-AI/Wan2.2-S2V), [ModelScope](https://www.modelscope.cn/studios/Wan-AI/Wan2.2-S2V)
- Animation demos: [Hugging Face](https://huggingface.co/spaces/Wan-AI/Wan2.2-Animate), [ModelScope](https://www.modelscope.cn/studios/Wan-AI/Wan2.2-Animate)

**Rent a GPU by the hour.** For TI2V-5B, rent a 24 GB NVIDIA card. For the 14B single-GPU examples, rent an 80 GB card. Providers that commonly list these cards include RunPod, Vast.ai, Lambda, and the GPU instances on AWS, Google Cloud, and Azure. Compare the current hourly price on the provider's site. This fork does not operate those services and does not set their prices.

**Community projects listed in the original README** that aim at lower memory or faster inference include [LightX2V](https://github.com/ModelTC/LightX2V), [DiffSynth-Studio](https://github.com/modelscope/DiffSynth-Studio), [Kijai's ComfyUI WanVideoWrapper](https://github.com/kijai/ComfyUI-WanVideoWrapper), and [FastVideo](https://github.com/hao-ai-lab/FastVideo). Those are third-party projects. Read their own docs before you rely on them.

## Troubleshooting and FAQ

**The run prints `Please specify the checkpoint directory.`** `--ckpt_dir` is required. Point it at the folder you downloaded, for example `./Wan2.2-TI2V-5B`. Cloning the git repo does not download the weights.

**CUDA out of memory.** Stay on TI2V-5B and keep `--offload_model True --convert_model_dtype --t5_cpu`. Close other GPU programs. A 24 GB card is the documented floor for that command. The 14B single-GPU commands are documented at 80 GB.

**`Unsupport size`.** The size is not in the table for that task. TI2V-5B accepts only `1280*704` and `704*1280`. The other tasks do not use those 704 sizes, except S2V-14B, which accepts both families.

**The shell says `no matches found` for the size.** Quote it: `--size '1280*704'`.

**`flash_attn` is missing, or attention asserts that FlashAttention 2 is unavailable.** The video model calls flash attention directly. Finish the install in [INSTALL.md](INSTALL.md). Skipping the package is not a supported path for `generate.py`.

**The process sits at the start for a long time.** The checkpoint is loading. A TI2V-5B 720p clip is on the order of nine minutes of compute on one RTX 4090 after that, using the Wan team's measurement.

**Multi-GPU fails with an error about `ulysses_size` or FSDP.** `--ulysses_size` has to equal the number of processes (`--nproc_per_node` for a single machine). `t5_fsdp` and `dit_fsdp` are for the distributed launch only. The single-GPU quick start does not use them.

**Prompt extension fails.** Leave it off until the basic command works. DashScope needs `DASH_API_KEY`. The local Qwen path downloads another model and needs more GPU memory. International Alibaba Cloud accounts also set `DASH_API_URL`, as described in the original README.

**The product in the clip does not match the photo.** Use a sharper photo, a shorter motion ("the camera moves slowly closer"), and the same product words in the prompt. Generate more than one seed by re-running. `--base_seed` sets the seed when you need a repeatable result.

**Can this make a finished social ad?** It makes a short shot. Titles, a voiceover, and a cut are done in your editor. The default TI2V clip is about 5 seconds.

**Can I use the clips commercially?** The Wan team states that the models are under the Apache 2.0 license and that they claim no rights over generated contents, provided your use follows the license. The same text says you are accountable for the output, and that use must not involve illegal content, harm, personal information used to harm someone, misinformation, or targeting vulnerable people. Read [LICENSE.txt](../LICENSE.txt) and the model card for the checkpoint you ran. Transformaiton.com does not give legal advice.

**Who wrote the model?** The Wan team (Alibaba). Credits and the non-affiliation statement are in the [README](../README.md#credits).

## Contact

[info@transformaiton.com](mailto:info@transformaiton.com) · [transformaiton.com](https://transformaiton.com/) · [github.com/Transformaiton](https://github.com/Transformaiton)

Wan research and product questions go to the Wan team's [Discord](https://discord.gg/AKNgpMK4Yj) or the WeChat group linked from the [upstream README](https://github.com/Wan-Video/Wan2.2).
