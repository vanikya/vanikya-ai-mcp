---
name: creative-insights
description: Use this skill when the user asks for creative or visual analysis of images, branding feedback, visual critique, creative scores, design quality assessment, batch analysis of multiple images for creative insights, ad creative analysis, marketing image review, or product photo feedback.
version: 1.1.0
---

# Vanikya Creative Insights

Get AI-powered creative and visual analysis of images using Vanikya's creative insights tools.

## Cost

Each creative insight costs **0.75 credits**. Always inform the user of this cost before running analysis.

## Available Tools

| Tool | Purpose |
|---|---|
| `creative_insights_create` | Analyze a single asset (synchronous — returns the result directly) |
| `creative_insights_batch_create` | Analyze up to 10 assets at once (async — returns job ids) |
| `creative_insights_batch_status` | Poll status of batch jobs by ids |
| `creative_insights_get` | Get a specific insight result by id |
| `creative_insights_list` | List past creative insights |
| `creative_insights_delete` | Delete an insight |

## Inputs Use `assetId`, Not URLs

Insights are run against Vanikya **assets** (uploaded or generated). If the user has an external image, first import it:

```
imagine_import_asset_from_url({ url: "<external url>" })
```

Then use the returned asset id. To analyze a Vanikya generation, use its asset id from `imagine_list_assets({ source: "generated" })` or `imagine_get_generation`.

## Workflow — Single Image

1. **Inform the user** of the 0.75 credit cost and confirm.
2. **Run analysis** (synchronous — returns the completed insight in one call, no polling needed):
   ```
   creative_insights_create({ assetId: "<asset_id>", prompt: "<optional context>" })
   ```
3. Present the analysis to the user.

## Workflow — Multiple Images

1. **Inform the user** of the total cost (0.75 credits × number of images) and confirm.
2. **Start batch** — returns an `ids` array:
   ```
   creative_insights_batch_create({ assetIds: ["<id1>", "<id2>", ...], prompt: "<optional>" })
   ```
   Max 10 assets per batch.
3. **Poll batch status** with the returned ids using exponential backoff (3s, 6s, 12s, 24s…):
   ```
   creative_insights_batch_status({ ids: ["<id1>", "<id2>", ...] })
   ```
   Stop after 5 retries.
4. Once complete, retrieve full results:
   ```
   creative_insights_get({ id: "<insight_id>" })
   ```

## Browsing Past Insights

```
creative_insights_list({ limit: 10, offset: 0 })
```

## Deleting an Insight

Only delete when the user explicitly requests it:
```
creative_insights_delete({ id: "<insight_id>" })
```

Always confirm with the user before deleting — deletions are permanent.

## Rules

- **Always** inform the user of the credit cost (0.75 per image) and get confirmation before running.
- Inputs are **asset ids**, not external URLs. Import external URLs first via `imagine_import_asset_from_url`.
- `creative_insights_create` is **synchronous** — do not poll. `creative_insights_batch_create` is async — poll `creative_insights_batch_status` with the returned ids.
- Present insights in a structured format — highlight key creative strengths and improvement areas.
