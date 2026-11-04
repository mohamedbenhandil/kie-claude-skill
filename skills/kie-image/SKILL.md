---
name: kie-image
description: Generate images with KIE.ai (GPT Image 2.5 Flare, Nano Banana 2.1) from a prompt or reference image, optimize the prompt, and show results inline. Use when the user asks to generate/make an image, logo, hero, packaging or mockup "with kie", "gpt 2.5" or "nano banana".
---

# KIE image generation

## Setup
- API key comes from env `KIE_API_KEY` (set once with `setx KIE_API_KEY "<key>"`, then reopen the app). Never write the key into files, prompts or replies. If unset, ask the user to set it.
- Base: `https://api.kie.ai`, header `Authorization: Bearer $KIE_API_KEY`.
- Credits check: `GET /api/v1/chat/credit`.

## Models (verified)
| User says | model id | endpoint input |
|---|---|---|
| gpt 2.5 / gpt image 2.5 | `gpt-image-2-5-flare-text-to-image` | `prompt`, `aspect_ratio`, `resolution` (1K/2K/4K) |
| nano banana 2.1 | `nano-banana-2-1` | `prompt`, `image_input[]` (URLs), `aspect_ratio`, `resolution`, `output_format` (png) |
Others exist (Sunburst variant, GPT Image 2, Nano Banana Pro). Look up docs at `https://docs.kie.ai/llms.txt` when the user names another model.
Cost seen: ~3 credits per 1K image.

## Workflow
1. Take the user's prompt, optimize it (subject, style, palette, composition, "no text/logos" when relevant). Use the user's exact model, resolution and aspect ratio.
2. Submit: `POST /api/v1/jobs/createTask` with `{"model":..., "input":{...}}` -> `data.taskId`.
3. Poll `GET /api/v1/jobs/recordInfo?taskId=ID` every 5s until `state` is `success` (or `fail`, read `failMsg`). Image URL is in `resultJson` (`resultUrls`).
4. Download to the project's `output/` folder, then show it with `SendUserFile` (`display: "render"`) so it appears big. Several images: send all in one call. Run several jobs in parallel with background `&` + `wait`.
5. Reply tersely: model, resolution, ratio, mood/what each variant is. No long explanations (user wants to save tokens).

## Reference images (image-to-image / keep a logo)
- Upload local file: `POST https://kieai.redpandaai.co/api/file-stream-upload` multipart `file=@path`, `uploadPath=images/ref`, `fileName=x.png` -> `data.downloadUrl`. Pass that URL in `image_input`.
- KNOWN BUG: Nano Banana 2.1 often returns the reference image UNCHANGED when `image_input` is given with a strong "recreate" prompt (happened for logo variations). After generating, compare hashes/Read the image; if identical to the reference, regenerate text-only (omit `image_input`) with a detailed description.
- Photo scenes where a brand logo must be preserved worked well with `image_input` and "keep logo exact" wording; still verify the text.

## Quality rules learned
- Always Read/inspect at least one output when text, logos or "no X" constraints matter, and say plainly if unchecked.
- Text in images (names, Arabic) is unreliable: warn the user to verify spelling; Arabic needs a native check.
- For logos: say "flat vector, thin monoline, solid background", give hex colors; avoid "bold" (user said too heavy = not luxury). Keep variants to one concept each unless asked to combine.
- Hero images for web: put subjects on the right and keep the left empty for titles; ask for "nothing cropped".
- Outputs are PNG, not vector; mention SVG vectorization is needed for web/app logos.
- Reply in the user's language (French or English).
