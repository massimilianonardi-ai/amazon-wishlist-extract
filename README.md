# amazon-wishlist-extract

Deterministic extraction of an Amazon.it wishlist through the **provider-neutral `web-control 2`** facility. Uses only standard page new/navigate/inspect/scroll/capture/close, not Playwright/CDP, Electron APIs, credentials, Gmail or Google Sheets.

## Usage

Requires an installed Node.js 26 runtime and a running `web-control 2` provider service with an authenticated browser profile (when Amazon requires login).

```sh
srv start web-control
amazon-wishlist-extract 'https://www.amazon.it/hz/wishlist/ls/<WISHLIST-ID>' > wishlist.json
```

The released archive is distributed through the corresponding `pkg-catalog` package and exposes `amazon-wishlist-extract` as the package command. For direct checkout development: `npm install`; then `node src/probe.mjs '<URL>'`.

Reusable JavaScript API:

```js
import { extract } from './src/amazon-wishlist-extract.mjs';
const observation = await extract(url, { scrollWaitMs: 1500, stableRounds: 3, maxRounds: 80 });
if (!observation.complete || observation.blocked) {
  // Preserve evidence; do not use a partial listing as authoritative inventory.
}
```

The API accepts an explicit Amazon.it wishlist URL and the options `webControlCommand`, `scrollWaitMs`, `stableRounds`, `maxRounds`, `timeoutMs`. It returns the same structured observation as the CLI. For a blocked/incomplete site page it returns a result with `complete: false`; for a browser-command failure it throws and attaches any available `error.observation`. CLI exit codes: 0 for stable observation, 1 for blocked/incomplete, 2 for browser/transport failure.

Result includes page metadata, `complete`, `completionReason`, `blocked`/`blockSignals`, `failure`, discovered unique ASINs, price evidence in EUR cents or explicit unverified reason, visible purchase action, and per-round scroll/loading diagnostics. Persistent spinner markup alone requires an extended stable-bottom interval and is disclosed as `stable-bottom-spinner-markers`; that observation is **not proof of a complete Amazon inventory**. Human authentication and challenges are not bypassed. The command creates/closes only its own transient page; the provider profile remains separate.

## Development and release

`mk.json` defines check/test/build using the canonical RumiAI `mk` engine; `npm test` exercises the local parser and API failure contracts, while GitHub Actions exercises the real Electron provider against a local lazy-loaded wishlist fixture. The portable release archive bundles the pinned JavaScript dependencies, excludes development-only tests and includes the CLI and API.

The extractor does **not** implement price history, monitoring targets, alerts, spreadsheets, email or scheduling. Those are separate application concerns. No user wishlist identifiers or browser profiles are bundled.
