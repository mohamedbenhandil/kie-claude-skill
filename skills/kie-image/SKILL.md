---
name: kie-image
description: Generate images, video, music, speech or chat with ANY model on KIE.ai (GPT Image, Nano Banana, Seedream, Flux, Kling, Veo, Wan, Runway, Suno, ElevenLabs...). Optimizes the prompt, calls the right endpoint, polls, downloads and shows the result inline. Use when the user asks to generate/make an image, logo, video, music or voice "with kie" or names a KIE model.
---

# KIE generation (any model)

## Setup
- API key from env `KIE_API_KEY` (user sets once: `setx KIE_API_KEY "<key>"` on Windows, then reopens the app; `export` on macOS/Linux). Never write the key into files, prompts or replies. If unset, ask the user to set it.
- Header on every KIE call: `Authorization: Bearer $KIE_API_KEY`. Base: `https://api.kie.ai`.
- Balance: `GET /api/v1/chat/credit`.

## Find ANY model (do not rely on memory)
1. `curl -s https://docs.kie.ai/llms.txt` is the full index. Entries look like `Image Models > Google [Title](url.md)`; categories: Image, Video, Music/Audio, Chat. Grep it for the model name the user said (e.g. `grep -i "veo\|kling"`).
2. Fetch the model's doc page (`<url>.md`, English, not `/cn/`). Read the OpenAPI YAML: the `model` enum value, `input` properties (required fields, enums, limits), and the endpoint path. Never guess parameter names or values; a model's `aspect_ratio`/`resolution` enums differ per model.
3. Most models live under `https://docs.kie.ai/market/...` and share one flow (below). A few families have dedicated docs/endpoints (Suno `suno-api`, Veo3 `veo3-api`, Runway `runway-api`, Flux Kontext, 4o Image, Chat models): follow that page's own endpoint and its own status/result endpoint.
4. Several size/quality or text-to-X vs image-to-X variants have different model ids. Pick the one that matches the request (text-to-image vs image-to-image, etc.).

## Market flow (unified, verified for image models)
1. `POST /api/v1/jobs/createTask` with `{"model":"<id>","input":{...}}` -> `data.taskId`. Optional `callBackUrl` (not needed; poll instead).
2. Poll `GET /api/v1/jobs/recordInfo?taskId=ID` every 5s (video/music: longer, 10-15s, up to several minutes) until `data.state` = `success` (or `fail`: read `failMsg`). Result URLs: `resultJson` -> `resultUrls`. `creditsConsumed` shows the cost.
3. Download to the project's `output/` folder and display (images: `SendUserFile` with `display: "render"`, several files in one call; video/audio: attach and give the path).
4. Run several jobs in parallel with background `&` + `wait`.

Verified models so far: `gpt-image-2-5-flare-text-to-image` (`prompt`, `aspect_ratio`, `resolution` 1K/2K/4K) and `nano-banana-2-1` (`prompt`, `image_input[]`, `aspect_ratio`, `resolution`, `output_format`). ~3 credits per 1K image. Pricing for other models: check the doc page or tell the user it is unknown and ask before expensive jobs (video, 4K, music).

## Reference images / files
- Upload a local file: `POST https://kieai.redpandaai.co/api/file-stream-upload` multipart `file=@path`, `uploadPath=images/ref`, `fileName=x.png` -> `data.downloadUrl` (temporary). Pass that URL where the model expects a URL (`image_input`, `image_urls`, ...; check the doc page). Also available: base64 and URL upload (docs: file-upload-api).
- KNOWN BUG: Nano Banana 2.1 can return the reference image UNCHANGED when `image_input` is given with a "recreate/variation" prompt. After generating, compare file hashes or look at the image; if identical, regenerate text-only (omit `image_input`) with a detailed description. Photo edits that preserve a brand logo worked with `image_input`; still verify the text.

## Quality rules learned
- Optimize the user's prompt (subject, style, palette with hex colors, composition, "no text/logos" when relevant) and use the user's exact model, resolution and ratio.
- Look at at least one output when text, logos or "no X" constraints matter, and say plainly when something was not checked.
- Text inside images (names, Arabic) is unreliable: tell the user to verify spelling; Arabic needs a native speaker.
- Logos: "flat vector, thin monoline, solid background", hex colors; "bold" reads as not luxurious; one concept per variant unless asked to combine. Output is PNG, not vector (needs SVG vectorizing for web/app).
- Web hero: subjects on the right, empty left for titles, "nothing cropped".
- Keep replies terse (user saves tokens): model, resolution, ratio, one line per variant. Reply in the user's language (French or English).
