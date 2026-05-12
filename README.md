# 🐇 BunnyNet OpenAPI specification

Repository to keep track of changes in the [OpenAPI specifications](https://docs.bunny.net/openapi) of the [Bunny.net API](https://docs.bunny.net/reference/bunnynet-api-overview).

## 🗂️ Data

Data is available at [toshy.github.io/bunnynet-openapi-specification](https://toshy.github.io/bunnynet-openapi-specification/manifest.json).

```text
https://toshy.github.io/bunnynet-openapi-specification/
├── manifest.json
├── core-api-v1.0.0.json
├── edge-scripting-api-v1.0.0.json
├── edge-storage-api-v1.0.0.json
├── magic-containers-api-v1.0.0.json
├── origin-errors-api-v1.0.0.json
├── shield-api-v1.0.0.json
└── stream-api-v1.0.0.json
```

## ⌛ Schedule

1. A cron schedule every sixth hour (`0 0,6,12,18 * * *` ) triggers workflow [`openapi.yml`](.github/workflows/openapi.yml) (actual workflow execution time may be [delayed](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#schedule)).
2. OpenAPI files are (temporarily) downloaded, and hashes are computed and compared by using the [`manifest.json`](./data/manifest.json).
3. Any changes will be automatically committed and a new [release](https://github.com/ToshY/bunnynet-openapi-specification/releases) will be created.

## ❕ License

This repository comes with a [BSD 3-Clause License](./LICENSE).
