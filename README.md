# IB Gateway Stable — automatic mirror

An automated mirror of the **Interactive Brokers IB Gateway stable** installers.

Every day a GitHub Actions workflow asks IBKR for the current stable version. If that
version has no release here yet, it downloads every platform's installer and publishes
them as a GitHub Release tagged with the version string (e.g. `10.50.1f`).
If the version is already published, nothing is downloaded.

> Not affiliated with or endorsed by Interactive Brokers. All installers are the
> unmodified files served by IBKR's official download servers.

## Download

Grab the files from the [Releases](../../releases/latest) page:

| Platform | File |
|---|---|
| macOS (Apple Silicon) | `ibgateway-stable-standalone-macos-arm.dmg` |
| macOS (Intel) | `ibgateway-stable-standalone-macosx-x64.dmg` |
| Windows x64 | `ibgateway-stable-standalone-windows-x64.exe` |
| Linux x64 | `ibgateway-stable-standalone-linux-x64.sh` |
| Linux arm64 | `ibgateway-stable-standalone-linux-arm.sh` |

Each release also contains `SHA256SUMS`. To verify a download:

```bash
sha256sum -c SHA256SUMS
```

The tag is the IBKR build version, so you can pin a specific version with a direct
asset URL:

```
https://github.com/<owner>/<repo>/releases/download/<version>/ibgateway-stable-standalone-linux-x64.sh
```

## How it works

[.github/workflows/ibkr-gateway-stable.yml](.github/workflows/ibkr-gateway-stable.yml)
runs on a daily schedule (02:13 UTC) and can also be started manually.

1. **Check** — fetches
   `https://download2.interactivebrokers.com/installers/ibgateway/stable-standalone/version.json`
   (a JSONP endpoint) and parses `buildVersion` and `buildDateTime` out of it.
2. **Compare** — uses the version string as the release tag. If a release with that
   tag already exists *and* has all expected assets, the run stops there. A release
   with missing assets (for example from an interrupted upload) is deleted and rebuilt.
3. **Publish** — downloads the five installers, rejects anything that is suspiciously
   small or turns out to be an HTML error page, writes `SHA256SUMS`, and creates the
   release with all assets attached at once.

Downloads happen only when a new version appears, so the routine daily run is a single
HTTP request.

## Run your own copy

1. Fork this repository (or create a new one and copy the workflow file).
2. Make sure Actions are enabled: **Settings → Actions → General → Allow all actions**.
3. Workflow permissions are set explicitly in the workflow (`contents: write`), so no
   personal access token is needed — the automatic `GITHUB_TOKEN` is used.
4. Open **Actions → IB Gateway Stable Release → Run workflow** to seed the first release.

### Manual runs

`workflow_dispatch` accepts a `force` input. When enabled, the release for the current
version is deleted and rebuilt even if it already looks complete.

## Notes

- **Scheduled workflows go dormant.** GitHub disables `schedule` triggers on
  repositories with no commits for 60 days. Push a commit or run the workflow manually
  occasionally if this repository is otherwise untouched.
- **Storage.** A full set of installers is roughly 1 GB per version. Old releases are
  kept indefinitely until you delete them; prune them on the Releases page if space
  matters.
- **Private repositories** consume Actions minutes (a full download takes about
  5–15 minutes). Public repositories are free and unmetered.
- **Mirror only.** If IBKR changes its download URLs or starts requiring cookies,
  the download step will fail loudly rather than publish a broken release.
