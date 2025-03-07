# Video Model Type Support In SwarmUI

| Model | Year | Author | Scale | Type | Quality/Status |
| ----  | ---- | ---- | ---- | ---- | ---- |
[Stable Video Diffusion](#stable-video-diffusion) | 2023 | Stability AI | 1B Unet | Image2Video | Outdated |
[Hunyuan Video](#hunyuan-video) | 2024 | Tencent | 12B MMDiT | Text2Video and Image2Video variants | Modern, High Quality |
[Genmo Mochi 1](#genmo-mochi-1-text2video) | 2024 | Genmo | 10B DiT | Text2Video | Modern, Decent |
[Lightricks LTX Video](#lightricks-ltx-video) | 2024 | Lightricks | 3B DiT | Text/Image 2Video | Modern, Fast but ugly |
[Nvidia Cosmos](#nvidia-cosmos) | 2025 | NVIDIA | Various | Text/Image/Video 2Video | Modern, Very slow, mixed quality |

**Unsupported:**
- Below are some video models that are not natively supported in SwarmUI's `Generate` tab, but are available to use via the `Comfy Workflow` and `Simple` tabs:
    - [CogVideoX](https://github.com/THUDM/CogVideo) (Tsinghua University, 2024, 2B & 5B DiT, Text/Image 2Video) is a decent video model, but unfortunately ComfyUI support is limited to [very hacky comfy nodes based on diffusers](https://github.com/kijai/ComfyUI-CogVideoXWrapper) which can not be easily integrated in SwarmUI's workflow generator.

## Demo Gifs

- Video demos included below are seed `1` of the prompt `wide shot, video of a cat with mixed black and white fur, walking in the middle of an open roadway, carrying a cardboard sign that says "Meow I'm a Cat". In the distance behind is a green road sign that says "Model Testing Street"` ran on each model.
- For all models, "standard parameters" are used.
    - Steps is set to 20 for all models.
    - Frame count is set as model default.
    - CFG is set appropriate to the model.
    - Resolution is model default.
    - FPS is model default.
    - Note that outputs are converted and shrunk to avoid wasting too much space / processor power on the docs page.
- For image2video models, an era-appropriate text2image model is used and noted.
- This is just the image test prompt from [Model Support](/docs/Model%20Support.md) but I swapped 'photo' to 'video', 'sitting' to 'walking', and 'holding' to 'carrying'. Goal is to achieve the same test as the image prompt does, but with a request for motion.
- All generations are done on the base model of the relevant class, not on any finetune/lora/etc. Finetunes are likely to significantly change the qualitative capabilities, but unlikely to significantly change general ability to understand and follow prompts.
- At time of writing, Hunyuan Video is the only properly good model. LTXV is really fast though.

# Video Models

## Stable Video Diffusion

![svd11_out](https://github.com/user-attachments/assets/ebeb3419-2c96-4746-863c-85ae4bc250d6)

*(SVD XT 1.1, Generated using SDXL 1.0 Base as the Text2Image model)*

- SVD models are supported via the `Image To Video` parameter group. Like XL, video by default uses enhanced inference settings (better sampler and larger sigma value).
- The model has no native text2video, so do not select it as your main model.
- You can do image2video by using an Init Image and setting Creativity to 0.
- You can replicate text2video by just using a normal image model (eg SDXL) as the first-frame generator.
- This model was released after SDXL, but was built based on SDv2.

## Hunyuan Video

![hunyuan-video](https://github.com/user-attachments/assets/12d898c4-d9c8-447e-99b3-42ad0f0eb16d)

### Hunyuan Video Basic Install

- Hunyuan Video is supported natively in SwarmUI as a Text-To-Video model.
- Use the Comfy Org repackaged model <https://huggingface.co/Comfy-Org/HunyuanVideo_repackaged/blob/main/split_files/diffusion_models/hunyuan_video_t2v_720p_bf16.safetensors>
    - Save to the `diffusion_models` folder
- Or use the gguf models from city96 <https://huggingface.co/city96/HunyuanVideo-gguf/tree/main>
    - `Q6_K` is near identical to full precision and is recommended for 24 gig cards, `Q4_K_M` is recommended if you have low VRAM, results are still very close, other variants shouldn't be used normally
    - Save to the `diffusion_models` folder, then load up Swarm and click the `☰` hamburger menu on the model, then `Edit Metadata`, and set the `Architecture:` field to `Hunyuan Video` (this *might* autodetect but not guaranteed so double-check it)
- The text encoders (T5-XXL, and LLaVA-LLaMA3) and VAE will be automatically downloaded.
- When selected, the `Text To Video` parameter group will become visible

### Hunyuan Video Parameters

- **Resolution:** The model is trained for 1280x720 (960x960) or 960x544 (720x720) resolutions or other aspect ratios of the same total pixel count
    - Using a lower resolution, like 848x480, can work with only some quality loss, and much lower mem/gen time.
- **FPS:** The model is trained for 24 fps (cannot be changed, editing the FPS value will just give you 'slowmo' outputs)
- **FrameCount (Length):** The model supports dynamic frame counts (eg 73 or 129 is solid), so you can pick the duration you want via the `Text2Video Frames` parameter.
    - Multiples of 4 plus 1 (4, 9, 13, 17, ...) are required due to the 4x temporal compression in the Hunyuan VAE.
    - The input parameter will automatically round if you enter an invalid value.
    - For quick generations, `25` is a good short frame count that creates about 1 second of video.
        - Use `49` for 2 seconds, `73` for 3 seconds, `97` for 4 seconds, `121` for 5 seconds, `145` for 6 seconds, 
    - Supposedly, a frame count of 201 yields a perfect looping video (about 8.5 seconds long).
- **Guidance Scale:** Hunyuan Video is based on the Flux Dev architecture, and has similar requirements.
    - Set the core `CFG Scale` parameter to 1.
    - You can use the `Flux Guidance Scale` parameter on this model (for Hunyuan Video, unlike Flux Dev, this value is embedded from CFG scale, and so prefers values around 6).
        - For "FastVideo" raise it up to 10.
- **Sigma Shift:** Leave `Sigma Shift` disabled for regular Hunyuan Video, but for "FastVideo" enable it and raise it to 17.

### Hunyuan Video Performance / Optimization

- Hunyuan Video is very GPU and memory intensive, especially the VAE
    - Even on an RTX 4090, this will max out your VRAM and will be very slow to generate. (the GGUF models help reduce this)
- The VAE has a harsh memory requirement that may limit you from high duration videos.
    - VAE Tiling is basically mandatory for consumer GPUs. You can configure both image space tiling, and video frame tiling, with the parameters under `Advanced Sampling`.
    - If you do not manually enable VAE Tiling, Swarm will automatically enable it at 256 with 64 overlap, and temporal 32 frames with 4 overlap. (Because the memory requirements without tiling are basically impossible. You can set the tiling values very very high if you want to make the tile artifacts invisible and you have enough memory to handle it).
- By default the BF16 version of the model will be loaded in FP8. To change this, use the `Preferred DType` advanced parameter.
    - FP8 noticeably changes results compared to BF16, but lets it run much much faster.
    - The GGUF versions of the model are highly recommended, as they get much closer to original and very close performance to fp8.
        - GGUF Q6_K is nearly identical to BF16.

### Hunyuan Video Additional Notes

- You can use Hunyuan Video as a Text2Image model by setting `Text2Video Frames` to `1`.
    - The base model as an image generator performs like a slightly dumber version of Flux Dev.

### FastVideo

- FastVideo is a version of Hunyuan Video trained for lower step counts (as low as 6)
- You can get the FastVideo fp8 from Kijai <https://huggingface.co/Kijai/HunyuanVideo_comfy/blob/main/hunyuan_video_FastVideo_720_fp8_e4m3fn.safetensors>
    - Save to the `diffusion_models` folder
- Or the gguf FastVideo from city96 <https://huggingface.co/city96/FastHunyuan-gguf/tree/main>
    - Save to the `diffusion_models` folder, then load up Swarm and click the `☰` hamburger menu on the model, then `Edit Metadata`, and set the `Architecture:` field to `Hunyuan Video` (this *might* autodetect but not guaranteed so double-check it)
- Set the advanced `Sigma Shift` param to a high value around 17
- Set the Flux Guidance at a higher than normal value as well (eg 10).
- Not adjusting these values well will yield terribly distorted results. Swarm does not automate these for FastVideo currently!

### SkyReels Text2Video

- SkyReels is a finetune of Hunyuan video produced by SkyWorkAI, see their repo [here](https://github.com/SkyworkAI/SkyReels-V1)
- You can download a SkyReels Text2Video fp8 model from here <https://huggingface.co/Kijai/SkyReels-V1-Hunyuan_comfy/blob/main/skyreels_hunyuan_t2v_fp8_e4m3fn.safetensors>
    - Save to the `diffusion_models` folder
- Broadly used like any other Hunyuan Video model
- This model prefers you use real CFG Scale of `6`, and set `Flux Guidance` value to `1`
- Their docs say you should prefix prompts with `FPS-24, ` as this was trained in. In practice the differences seem to be minor.
- `Sigma Shift` default value is `7`, you do not need to edit it

### SkyReels Image2Video

- You can download a SkyReels Image2Video fp8 model from here <https://huggingface.co/Kijai/SkyReels-V1-Hunyuan_comfy/blob/main/skyreels_hunyuan_i2v_fp8_e4m3fn.safetensors>
    - Save to the `diffusion_models` folder
- Use via the `Image To Video` param group
- This model prefers you use real CFG Scale of around `4` to `6`, and set `Flux Guidance` value to `1`
- Their docs say you should prefix prompts with `FPS-24, ` as this was trained in. In practice the differences seem to be minor.
- The model seems to be pretty hit-or-miss as to whether it creates a video of your image, or just "transitions" from your image to something else based on the prompt.
- The model seems to have visual quality artifacts
    - Set Video Steps higher, at least `30`, to reduce these
- `Sigma Shift` default value is `7`, you do not need to edit it

## Genmo Mochi 1 (Text2Video)

![mochi](https://github.com/user-attachments/assets/4d64443e-c46f-415d-8203-17f6aa0f4cc5)

- Genmo Mochi 1 is supported natively in SwarmUI as a Text-To-Video model.
- You can get either the all-in-one checkpoint <https://huggingface.co/Comfy-Org/mochi_preview_repackaged/tree/main/all_in_one>
    - save to `Stable-Diffusion` folder
- Or get the DiT only variant <https://huggingface.co/Comfy-Org/mochi_preview_repackaged/tree/main/split_files/diffusion_models> (FP8 Scaled option recommended)
    - save to `diffusion_models` folder
- The text encoder (T5-XXL) and VAE will be automatically downloaded
    - You can also set these manually if preferred
- When selected, the `Text To Video` parameter group will become visible
- Mochi is very GPU and memory intensive, especially the VAE
- Standard CFG values, eg `7`.
- The model is trained for 24 fps, and frame counts dynamic anywhere up to 200. Multiples of 6 plus 1 (7, 13, 19, 25, ...) are required due to the 6x temporal compression in the Mochi VAE. The input parameter will automatically round if you enter an invalid value.
- The VAE has a harsh memory requirement that may limit you from high duration videos.
    - To reduce VRAM impact and fit on most normal GPUs, set `VAE Tile Size` to `160` or `128`, and `VAE Tile Overlap` to `64` or `96`. There will be a slightly noticeable tiling pattern on the output, but not too bad at 160 and 96.
    - If you have a lot of VRAM (eg 4090) and want to max quality but can't quite fit the VAE without tiling, Tile Size 480 Overlap 32 will tile the VAE in just two chunks to cut the VAE VRAM usage significantly while retaining near perfect quality.

## Lightricks LTX Video

![ltxv](https://github.com/user-attachments/assets/23e51754-79c6-47cd-9840-e65ec24fac1f)

*(LTX-Video 0.9.1, Text2Video, CFG=7 because 3 was really bad)*

- Lightricks LTX Video ("LTXV") is supported natively in SwarmUI as a Text-To-Video and also as an Image-To-Video model.
    - The text2video is not great quality compared to other models, but the image2video functionality is popular.
- Download <https://huggingface.co/Lightricks/LTX-Video/blob/main/ltx-video-2b-v0.9.safetensors>
    - save to `Stable-Diffusion` folder
    - The text encoder (T5-XXL) and VAE will be automatically downloaded
        - You can also set these manually if preferred
- When selected in the Models list, the `Text To Video` parameter group will become visible
- The model is trained for 24 fps but supports custom fps values, and frame counts dynamic anywhere up to 257. Multiples of 8 plus 1 (9, 17, 25, 33, 41, ...) are required due to the 8x temporal compression in the LTXV VAE. The input parameter will automatically round if you enter an invalid value.
- Recommended CFG=3, and very very long descriptive prompts.
    - Seriously this model will make a mess with short prompts.
    - Example prompt (from ComfyUI's reference workflow):
        - Prompt: `best quality, 4k, HDR, a tracking shot of a beautiful scene of the sea waves on the beach`
        - Negative Prompt: `low quality, worst quality, deformed, distorted, disfigured, motion smear, motion artifacts, fused fingers, bad anatomy, weird hand, ugly`
- You can use it as an Image-To-Video model
    - Select an image model and configure usual generation parameters
    - Select the LTXV model under the `Image To Video` group's `Video Model` parameter
    - Set `Video FPS` to `24` and `Video CFG` to `3`, set `Video Frames` to a higher value eg `97`
    - Pay attention that your prompt is used for both the image, and video stages
        - You may wish to generate the image once, then do the video separately
        - To do that, set the image as an `Init Image`, and set `Creativity` to `0`

## NVIDIA Cosmos

![cosmos-7b](https://github.com/user-attachments/assets/d4e4047e-1706-4f61-b560-3fe6f70783c6)

*(Cosmos 7B Text2World)*

### Nvidia Cosmos Basic Install

- NVIDIA Cosmos Text2World and Video2World (image2video) has initial support in SwarmUI.
- Cosmos Autoregressive is not yet supported.
- You can download the models from here: <https://huggingface.co/mcmonkey/cosmos-1.0/tree/main>
    - pick 7B (small) or 14B (large) - 7B needs less memory/time, but has worse quality. 14B needs more but has better quality. Both will be very slow even on a 4090.
    - Text2World takes a prompt and generates a video (as a base model in Swarm), Video2World takes text+an image and generates a video (via the Image To Video param group in Swarm).
    - Save to `diffusion_models`
- The text encoder is old T5-XXL v1, not the same T5-XXL used by other models.
    - It will be automatically downloaded.
- The VAE will be automatically downloaded.

### Nvidia Cosmos Parameters

- **Prompt:** Cosmos responds poorly to standard prompts, as it was trained for very long LLM-generated prompts.
- **FPS:** The model is trained for 24 FPS, but supports any value in a range from 12 to 40.
- **Resolution:** The model is trained for 1280x704 but works at other resolutions, including 960x960 as base square res.
    - Cannot go below 704x704.
- **Frame Count:** The model is trained only for 121 frames. Some of the model variants work at lower frame counts with quality loss, but generally you're stuck at exactly 121.
- **CFG and Steps:** Nvidia default recommends CFG=7 and Steps=35
- **Performance:** The models are extremely slow. Expect over 10 minutes for a single video even on a 4090.
