# CLIP Guided Diffusion
From [RiversHaveWings](https://twitter.com/RiversHaveWings).

Generate vibrant and detailed images using only text.


<img src="/images/photon.png" width="256"></img>
> "photon traveling at the speed of light"

[Gallery](/images/README.md)

<a href="https://colab.research.google.com/github/afiaka87/clip-guided-diffusion/blob/main/cgd_clip_selected_class.ipynb">Colab Notebook</a>

_Note:_ `prompt` has been changed from a positional argument to the keyword argument `--prompt`.
`--prompt_min` has been added to specify a prompt to penalize during generation.

---

### Installation
```sh
git clone https://github.com/afiaka87/clip-guided-diffusion.git
cd clip-guided-diffusion
python3 -m venv cgd_venv
source cgd_venv/bin/activate
(cgd_venv) $ pip install -r requirements.txt
(cgd_venv) $ git clone https://github.com/crowsonkb/guided-diffusion.git
(cgd_venv) $ python guided-diffusion/setup.py install
```

### Download checkpoints

You only need to download the checkpoint for the size you want to generate.
Checkpoints belong in the `./checkpoints` directory.

- 64: https://openaipublic.blob.core.windows.net/diffusion/jul-2021/64x64_diffusion.pt
- 128: https://openaipublic.blob.core.windows.net/diffusion/jul-2021/128x128_diffusion.pt
- 256: https://openaipublic.blob.core.windows.net/diffusion/jul-2021/256x256_diffusion.pt
- 512: https://openaipublic.blob.core.windows.net/diffusion/jul-2021/512x512_diffusion.pt

There is only one unconditional checkpoint. This one doesn't require a randomized class like the others do. Use `--class_cond False` to use.
- 256 (unconditional):  https://openaipublic.blob.core.windows.net/diffusion/jul-2021/256x256_diffusion_uncond.pt

### Quick start:

```sh
(cgd_venv) $ python cgd.py --image_size 256 --prompt "32K HUHD Mushroom"
Step 999, output 0:
00%|███████████████| 1000/1000 [00:00<12:30,  1.02it/s]
```
![](/images/32K_HUHD_Mushroom.png?raw=true)

Generations will be saved in the folder from `--prefix` (default:'./outputs')
with the format `{caption}/batch_idx_{j}_iteration_{i}.png`
The most recent generation will also be stored in the file `current.png` if you would like to
watch the generations in real time.

### Penalize a prompt
```sh
(cgd_venv) $ python cgd.py \
    --prompt "32K HUHD Mushroom" \
    --prompt_min "green grass"
```
<img src="images/32K_HUHD_Mushroom_MIN_green_grass.png" width="256"></img>


### Blending an existing image

This method will blend an image with the diffusion for a number of steps. 
You may need to tinker with `--skip_timesteps` to get the best results.
```sh
(cgd_venv) $ python cgd.py \
    --init_image=images/32K_HUHD_Mushroom.png \
    --skip_timesteps=500 \
    --prompt "A mushroom in the style of Vincent Van Gogh"
```
![](images/a_mushroom_in_the_style_of_vangogh.png?raw=true)

### Image size
- Default is 128px
- Available image sizes are `64, 128, 256, 512 pixels (square)`
- The 512x512 pixel checkpoint **requires a GPU with at least 12GB of VRAM.**
- CLIP_GUIDANCE_SCALE and TV_SCALE will require experimentation.
- the 64x64 diffusion checkpoint is challenging to work with and often results in an all-white or all-black image.
  - This is much less of an issue when using an existing image of some sort.
```sh
(cgd_venv) $ python cgd.py \
    --init_image=images/32K_HUHD_Mushroom.png \
    --skip_timesteps=500 \
    --image_size 64 \
    --prompt "8K HUHD Mushroom"
```
<img src="images/32K_HUHD_Mushroom_64.png?raw=true" width="256"></img>

```sh
(cgd_venv) $ python cgd.py --image_size 512 --prompt "8K HUHD Mushroom"
  ```
<img src="images/32K_HUHD_Mushroom_512.png?raw=true" width="360"></img>


> This code is currently under active development and is subject to frequent changes. Please file an issue if you have any constructive feedback, questions, or issues with the code or colab notebook.

## Full Usage:

```sh
(cgd_venv) $ python cgd.py --help
usage: cgd.py [-h] [--prompt PROMPT] [--prompt_min PROMPT_MIN] [--image_size IMAGE_SIZE] [--init_image INIT_IMAGE] [--skip_timesteps SKIP_TIMESTEPS] [--num_cutouts NUM_CUTOUTS] [--prefix PREFIX]
              [--batch_size BATCH_SIZE] [--clip_guidance_scale CLIP_GUIDANCE_SCALE] [--tv_scale TV_SCALE] [--seed SEED] [--save_frequency SAVE_FREQUENCY] [--device DEVICE]
              [--diffusion_steps DIFFUSION_STEPS] [--timestep_respacing TIMESTEP_RESPACING] [--cutout_power CUTOUT_POWER] [--clip_model CLIP_MODEL] [--class_cond CLASS_COND]
              [--custom_class CUSTOM_CLASS] [--clip_class_search]

optional arguments:
  -h, --help            show this help message and exit
  --prompt PROMPT       the prompt to reward (default: None)
  --prompt_min PROMPT_MIN
                        the prompt to penalize (default: None)
  --image_size IMAGE_SIZE
                        Diffusion image size. Must be one of [64, 128, 256, 512]. (default: 128)
  --init_image INIT_IMAGE
                        Blend an image with diffusion for n steps (default: None)
  --skip_timesteps SKIP_TIMESTEPS
                        Number of timesteps to blend image for. CLIP guidance occurs after this. (default: 0)
  --num_cutouts NUM_CUTOUTS, -cutn NUM_CUTOUTS
                        Number of randomly cut patches to distort from diffusion. (default: 64)
  --prefix PREFIX, --output_dir PREFIX
                        output directory (default: outputs)
  --batch_size BATCH_SIZE, -bs BATCH_SIZE
                        the batch size (default: 1)
  --clip_guidance_scale CLIP_GUIDANCE_SCALE, -cgs CLIP_GUIDANCE_SCALE
                        Scale for CLIP spherical distance loss. Default value varies depending on image size. (default: 1500)
  --tv_scale TV_SCALE, -tvs TV_SCALE
                        Scale for denoising loss. Disabled by default for 64 and 128 (default: 150)
  --seed SEED           Random number seed (default: 0)
  --save_frequency SAVE_FREQUENCY, -sf SAVE_FREQUENCY
                        Save frequency (default: 25)
  --device DEVICE       device to run on .e.g. cuda:0 or cpu (default: None)
  --diffusion_steps DIFFUSION_STEPS
                        Diffusion steps (default: 1000)
  --timestep_respacing TIMESTEP_RESPACING
                        Timestep respacing (default: 1000)
  --cutout_power CUTOUT_POWER, -cutpow CUTOUT_POWER
                        Cutout size power (default: 0.5)
  --clip_model CLIP_MODEL
                        clip model name. Should be one of: [ViT-B/16, ViT-B/32, RN50, RN101, RN50x4, RN50x16] (default: ViT-B/32)
  --class_cond CLASS_COND
                        Use class conditional. Required for image sizes other than 256 (default: True)
  --custom_class CUSTOM_CLASS
                        Custom class to use for image generation. Should be one of: [0-999] (default: None)
  --clip_class_search   Lookup imagenet class with CLIP rather than changing them throughout run. Use `--clip_class_search` on its own to enable. (default: False)
```