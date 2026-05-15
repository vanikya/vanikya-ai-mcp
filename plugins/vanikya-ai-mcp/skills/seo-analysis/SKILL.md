---
name: seo-analysis
description: Use this skill when the user asks to analyze SEO of a URL or website, run an SEO audit, get SEO insights or recommendations, check a page's search engine optimization, batch-analyze multiple URLs for SEO, or compare the SEO of multiple pages.
version: 1.1.0
---

# Vanikya SEO Analysis

Analyze web pages for SEO quality using Vanikya's AI-powered SEO analysis tools.

## Available Tools

| Tool | Purpose |
|---|---|
| `seo_analysis_create` | Run SEO analysis on a single URL (synchronous — returns the result directly) |
| `seo_analysis_batch_create` | Run SEO analysis on up to 10 URLs at once (async — returns job ids) |
| `seo_analysis_batch_status` | Poll status of batch jobs by ids |
| `seo_analysis_get` | Get a specific analysis result by id |
| `seo_analysis_list` | List past SEO analyses |
| `seo_analysis_delete` | Delete an analysis |

## Workflow — Single URL

1. Confirm the URL with the user.
2. **Run analysis** (synchronous — returns the completed result in one call, no polling needed):
   ```
   seo_analysis_create({ url: "<url>" })
   ```
   Typically completes in 15–45 seconds.
3. Present the results to the user — highlight critical issues first.

## Workflow — Multiple URLs

1. **Start batch** — returns an `ids` array (max 10 URLs):
   ```
   seo_analysis_batch_create({ urls: ["<url1>", "<url2>", ...] })
   ```
2. **Poll batch status** with the returned ids using exponential backoff (3s, 6s, 12s, 24s…):
   ```
   seo_analysis_batch_status({ ids: ["<id1>", "<id2>", ...] })
   ```
   Stop after 5 retries.
3. Once complete, retrieve full results:
   ```
   seo_analysis_get({ id: "<analysis_id>" })
   ```

## Browsing Past Analyses

```
seo_analysis_list({ limit: 10, offset: 0 })
```

Returns the user's past analyses. Use `seo_analysis_get` to fetch full results for any entry.

## Deleting an Analysis

Only delete when the user explicitly requests it:
```
seo_analysis_delete({ id: "<analysis_id>" })
```

Always confirm with the user before deleting — deletions are permanent.

## Rules

- Always confirm the URL with the user before running analysis.
- `seo_analysis_create` is **synchronous** — do not poll. `seo_analysis_batch_create` is async — poll `seo_analysis_batch_status` with the returned ids.
- Present results in a structured, readable format — highlight critical issues first.
