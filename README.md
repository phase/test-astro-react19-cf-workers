Reproduction steps for https://github.com/facebook/react/issues/31827

```bash
bun create astro test-astro-react19-cf-workers && cd test-astro-react19-cf-workers

bunx astro add react

# https://docs.astro.build/en/guides/integrations-guide/cloudflare/#installation
bunx astro add cloudflare
```

Change the on-demand rendering mode to `"server"` in `astro.config.mjs`

```bash
# Tell Cloudflare not to publish the worker's source.
# Wrangler will hard error if this isn't present.
echo "_worker.js" >> public/.assetsignore
```

Make `./wrangler.toml` with the following:

```toml
name = "test-astr-react19-cf-workers"
main = "dist/_worker.js"
compatibility_flags = ["nodejs_compat"]
compatibility_date = "2024-12-20"
assets = { directory = "./dist/" }

[observability.logs]
enabled = true
```

```bash
bunx astro build && bunx wrangler deploy
```

**Crashes with the following logs:**

```
building client (vite)
21:26:30 [vite] ✓ 21 modules transformed.
21:26:30 [vite] dist/_astro/client.CZXlMYiT.js  185.87 kB │ gzip: 58.75 kB
21:26:30 [vite] ✓ built in 266ms

prerendering static routes
21:26:30 ✓ Completed in 8ms.

21:26:30 [build] Rearranging server assets...
21:26:30 [build] Server built in 960ms
21:26:30 [build] Complete!

⛅️ wrangler 3.99.0
-------------------

🌀 Building list of assets...
🌀 Starting asset upload...
🌀 Found 5 new or modified static assets to upload. Proceeding with upload...
+ /_routes.json
+ /_astro/astro.Dm8K3lV8.svg
+ /favicon.svg
+ /_astro/client.CZXlMYiT.js
+ /_astro/background.BPKAcmfN.svg
Uploaded 1 of 5 assets
Uploaded 3 of 5 assets
Uploaded 5 of 5 assets
✨ Success! Uploaded 5 files (1.65 sec)

Total Upload: 779.06 KiB / gzip: 162.88 KiB

✘ [ERROR] A request to the Cloudflare API (/accounts/b5126ece2490f0088cd76c17ad917532/workers/scripts/test-astr-react19-cf-workers) failed.

 Uncaught ReferenceError: MessageChannel is not defined
   at null.<anonymous>
 (file:///Users/jado/projects/web/test-astro-react19-cf-workers/dist/_worker.js/renderers.mjs:6530:16)
 in requireReactDomServer_browser_production
   at null.<anonymous>
 (file:///Users/jado/projects/web/test-astro-react19-cf-workers/dist/_worker.js/renderers.mjs:12527:8)
 in requireServer_browser
   at null.<anonymous>
 (file:///Users/jado/projects/web/test-astro-react19-cf-workers/dist/_worker.js/renderers.mjs:12539:29)
 in dist/_worker.js/renderers.mjs
   at null.<anonymous> (_worker.js:18:59) in __init
   at null.<anonymous>
 (file:///Users/jado/projects/web/test-astro-react19-cf-workers/dist/_worker.js/index.js:2:1)
  [code: 10021]
```
