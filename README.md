# Liwip Store SDK releases

The packed Liwip Store SDK for each partner, published as GitHub Releases.
There is no npm registry: the installer downloads the packages straight from
a release and checks them against its `SHA256SUMS`. Source, issues and CI live
in a private repository.

Every release holds the same five files:

| File | What it is |
| --- | --- |
| `liwip-store-core-<version>.tgz` | Platform-neutral client and domain logic |
| `liwip-react-native-cashfree-<version>.tgz` | Cashfree payment adapter |
| `liwip-react-native-store-<version>.tgz` | The store UI, packed with the partner's layer |
| `liwip-store-install.mjs` | Installer, baked with this release's tag |
| `SHA256SUMS` | Checksums of the four files above |

## Release naming

One release per channel, partner and version.

| | Tag | Title | GitHub flag |
| --- | --- | --- | --- |
| dev | `dev-<partnerId>-v<x.y.z>`, e.g. `dev-pronto-v0.2.0` | `[dev] Pronto · Liwip Store SDK 0.2.0` | Pre-release, never Latest |
| prod | `prod-<partnerId>-v<x.y.z>`, e.g. `prod-pronto-v0.2.0` | `[prod] Pronto · Liwip Store SDK 0.2.0` | Release |

- `partnerId` is the partner's directory in the SDK (`namma-yatri`, `pronto`).
- Versions are plain semver (`0.2.0`) on both channels, with no prerelease
  suffix. The channel is in the tag, not the version.
- A dev number is never reused. If `dev-pronto-v0.2.0` exists, the next dev
  build is `0.2.1` or later.
- Prod only takes a number that dev has tested. `prod-pronto-v0.2.0` needs
  `dev-pronto-v0.2.0` to exist first.
- Published assets are never replaced. A change always gets a new version.

`v0.1.0-dev.1` … `v0.1.0-dev.9` are the old releases from before partners
had their own releases. They stay as they are for existing installs.

## Install a release

From the host React Native project, with the tag your Liwip contact gave you:

```sh
TAG=dev-pronto-v0.2.0
curl -fLO https://github.com/keystonecommerce/liwip-commerce-sdk-releases/releases/download/$TAG/liwip-store-install.mjs
node liwip-store-install.mjs --check-only   # compatibility check, changes nothing
node liwip-store-install.mjs                # install (asks first)
node liwip-store-install.mjs -y             # install without asking
```

Then rebuild the Android app once so the native libraries are linked:

```sh
npx react-native run-android   # Expo: npx expo run:android
```

The installer only downloads from the release it came from. It checks the
host's dependencies, installs any missing peers, verifies the checksums and
checks that shared native libraries resolve from the host app. npm, Yarn
Classic and pnpm all work, with no registry access or Yarn resolutions.

| Option | |
| --- | --- |
| `--project <path>` | Host project (default: current folder) |
| `--package-manager <npm\|yarn\|pnpm>` | Otherwise detected from the lockfile |
| `--check-only` | Check compatibility without installing |
| `-y`, `--yes` | Install without asking |
| `--sdk-dir <path>` | Install from a local folder of release files |
| `--release-base-url <url>` | Download from another URL |
| `--print-release` | Print the tag, partner and version the installer is for |
| `-h`, `--help` | Help |

## How the SDK works

A minimal config:

```ts
const config: LiwipStoreConfig = {
  api: { baseUrl: LIWIP_API_URL, partnerId: LIWIP_PARTNER_ID },
  auth: {
    accountId: user?.id ?? null,                    // null when signed out
    getSession: () => api.post('/liwip/session'),   // your backend, below
  },
  navigation: { onClose },
  // optional: paymentAdapter, analytics, callbacks
};

<LiwipStore config={config} />;
```

- **Sessions.** Your backend calls Liwip's `POST /partner/auth/session` with
  your secret key and returns the response unchanged. The secret key stays
  on your server and never goes in the app. The store keeps the session in
  secure storage and renews it.
- **Admin panel.** Features, the home and branding come from the Liwip admin
  panel, so they change without an app release.
- **Prefetch.** Call `prefetchLiwipStore({ api: config.api, auth: config.auth })`
  once at app launch so the first open shows the store at once.
- **Deprecated.** `features`, `theme` and `copy` still work as local
  overrides for one more release. Set them in the panel instead.

Details: <https://sdk.liwip.com>

## What changed from the old releases

- Partner sessions (`auth`) replace the publishable key, Play Integrity and
  `customer` sign-in flow.
- Features, home layout, branding and words move from local config to the
  admin panel.
- The home is finite: rails and grids show a few products with "See all",
  and it ends with "See all products".
- Every request sends `x-liwip-sdk-version`, and the panel can set a minimum
  version.
- New screens for when the panel closes the store ("The store is closed")
  and for an SDK below the minimum ("Update the app to keep shopping").

## Making a release (maintainers)

In the SDK repository:

```sh
npm run sdk:pack -- --partner pronto --channel dev       # writes dist/sdk/dev-pronto-v0.2.0/
npm run release:plan -- --partner pronto --channel dev   # checks it, prints the command
```

`release:plan` prints the `gh release create` command and does not run it:

```sh
gh release create dev-pronto-v0.2.0 --repo keystonecommerce/liwip-commerce-sdk-releases \
  --title '[dev] Pronto · Liwip Store SDK 0.2.0' --prerelease --latest=false \
  --notes-file dist/sdk/dev-pronto-v0.2.0.notes.md \
  dist/sdk/dev-pronto-v0.2.0/liwip-store-core-0.2.0.tgz \
  dist/sdk/dev-pronto-v0.2.0/liwip-react-native-cashfree-0.2.0.tgz \
  dist/sdk/dev-pronto-v0.2.0/liwip-react-native-store-0.2.0.tgz \
  dist/sdk/dev-pronto-v0.2.0/liwip-store-install.mjs \
  dist/sdk/dev-pronto-v0.2.0/SHA256SUMS
```

Both commands refuse a reused dev number, or a prod release with no matching
dev release. They check with `gh release view`, which only reads. `--force`
skips the check.

Publishing a release runs `.github/workflows/verify-yarn-release.yml`. The
workflow installs the release into a fresh Yarn host. It checks that the
installer's baked tag matches the release and that every package resolves
from the release at the tag's version. It also checks that the store carries
the tag's partner layer. To re-run it: **Actions → verify yarn release → Run
workflow** with the tag.

## Support

- Integration guide: <https://sdk.liwip.com>
- Engineering support: <tech@keystonecommerce.in>
