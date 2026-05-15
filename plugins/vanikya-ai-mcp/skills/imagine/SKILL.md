---
name: imagine
description: Use this skill when the user asks to generate images, videos, vector graphics (SVG), or lottie animations; edit an existing image; enhance a prompt; browse past generations; manage assets; or estimate generation costs. Covers all Vanikya Imagine tools.
version: 1.1.0
---

# Vanikya Imagine

Generate images, videos, vectors, and lottie animations using the user's Vanikya credits.

## Output Types

| Type | Description | `output_type` | `imagine_list_models` `type` filter |
|---|---|---|---|
| Raster image | Standard image (PNG/JPG) | `"image"` (default) | (omit) |
| Vector | SVG graphic | `"vector"` | `"vector"` |
| Lottie | Animated lottie JSON | `"lottie"` | `"lottie"` |
| Video | Short video clip | n/a (uses `imagine_generate_video`) | `"video"` |

## Available Tools

| Tool | Purpose |
|---|---|
| `imagine_list_models` | List available models — pass `type` for video/vector/lottie |
| `imagine_generate_image` | Generate raster image, vector, or lottie (pass `output_type`) |
| `imagine_generate_video` | Generate a video |
| `imagine_edit_generation` | Synchronously edit an existing image variation/version |
| `imagine_enhance_prompt` | Expand a short prompt into a detailed generation prompt |
| `imagine_estimate_credits` | Estimate credit cost before generating |
| `imagine_get_generation` | Fetch a specific generation by ID — only on explicit user request |
| `imagine_list_generations` | List past generations (filterable by format) |
| `imagine_update_generation_tags` | Add tags to a generation |
| `imagine_list_assets` | List reusable assets |
| `imagine_get_asset` | Get a specific asset |
| `imagine_import_asset_from_url` | Import an external image URL as a reusable asset |

## Important: Generation is Async, but the UI Handles Polling

`imagine_generate_image` and `imagine_generate_video` return immediately with `status: "pending"`. **The Vanikya chat UI renders an interactive progress view that polls for completion on its own.** Do NOT call `imagine_get_generation` after starting a generation — let the UI track it.

Only call `imagine_get_generation` when the user **explicitly** asks to check or retrieve a specific generation (e.g. "what was the result of generation X?").

## Working with Assets

Assets are reusable images stored in the user's Vanikya account (imported or saved from generations).

- `imagine_list_assets({ source?, format?, limit?, offset? })` — `source`: `"assets" | "generated"`. `format`: `"image" | "vector" | "lottie" | "video" | "all"`.
- `imagine_get_asset` — Fetch a specific asset by ID
- `imagine_import_asset_from_url({ url })` — Save an external image URL as a reusable asset (useful before editing)

Use assets as reference images in generation or editing workflows.

## Mandatory Workflow — Follow This Order

### Generating an image (raster)

1. **List available models** to get valid model IDs:
   ```
   imagine_list_models()
   ```
2. **Enhance the prompt** (unless the user already supplied a detailed prompt of ~50+ words):
   ```
   imagine_enhance_prompt({ prompt: "<user's raw prompt>" })
   ```
3. **Estimate credits** and confirm with the user:
   ```
   imagine_estimate_credits({ prompt: "<enhanced prompt>", models: ["<model_id>"] })
   ```
   If the user declines, stop. Offer a cheaper model or a revised prompt.
4. **Generate** — the UI tracks progress automatically:
   ```
   imagine_generate_image({ prompt: "<enhanced prompt>", models: ["<model_id>"] })
   ```
   Do NOT poll. Do NOT call `imagine_get_generation`. The chat UI handles it.

### Generating a video

1. **Fetch video models first** (video models are not in the default list):
   ```
   imagine_list_models({ type: "video" })
   ```
2. Estimate credits and confirm with the user.
3. Call `imagine_generate_video({ prompt, models: ["<video_model_id>"] })`.
4. The UI tracks progress automatically — do NOT poll.

### Generating a vector (SVG)

1. Fetch vector models:
   ```
   imagine_list_models({ type: "vector" })
   ```
2. Enhance prompt, estimate credits, confirm with the user.
3. Generate with the matching `output_type`:
   ```
   imagine_generate_image({
     prompt: "<enhanced prompt>",
     models: ["<vector_model_id>"],
     output_type: "vector"
   })
   ```
   Passing an image model with `output_type: "vector"` returns an error listing the correct vector models.
4. The UI tracks progress automatically — do NOT poll.

### Generating a lottie animation

1. Fetch lottie models:
   ```
   imagine_list_models({ type: "lottie" })
   ```
2. Enhance prompt, estimate credits, confirm with the user.
3. Generate:
   ```
   imagine_generate_image({
     prompt: "<enhanced prompt>",
     models: ["<lottie_model_id>"],
     output_type: "lottie"
   })
   ```
4. The UI tracks progress automatically — do NOT poll.
5. **Note:** Editing lottie generations is not supported. If the user wants changes, regenerate with a revised prompt.

### Editing an image

`imagine_edit_generation` is **synchronous** — it returns the edited image URL directly, no polling needed.

```
imagine_edit_generation({
  generationId: "<id>",
  variationId: "<id>",
  versionId: "<id>",
  editPrompt: "<edit instructions>"
})
```

Lottie editing is not supported — inform the user and offer to regenerate.

## Rules

- **Always** call `imagine_list_models` before generating to get valid model IDs — never guess a model ID.
- **Always** call `imagine_enhance_prompt` before generating unless the user's prompt is already detailed (~50+ words).
- **Always** call `imagine_estimate_credits` and show the user the cost before generating.
- **Never** call `imagine_get_generation` automatically after generating — the UI polls on its own. Only fetch a generation when the user explicitly asks.
- For vector and lottie output, pass the matching `output_type` AND a model returned by `imagine_list_models({ type: "<vector|lottie>" })`. For video, use `imagine_generate_video` with a video model.
