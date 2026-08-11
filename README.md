# Liwip Store SDK releases

Public distribution repository for versioned Liwip Store SDK artifacts.
Development source, issues and CI are maintained in a private repository.

## Install the development SDK

From the host React Native project:

```sh
curl -fLO https://github.com/keystonecommerce/liwip-commerce-sdk-releases/releases/download/v0.1.0-dev.3/liwip-store-install.mjs
node liwip-store-install.mjs
```

The installer checks host compatibility, verifies release checksums, installs
missing peer dependencies and confirms shared native libraries are resolved
from the host application.

npm, Yarn Classic and pnpm hosts receive all Liwip packages directly from the
same GitHub Release. No private npm-registry access or Yarn resolutions are
required.

## Documentation and support

- Integration guide: <https://sdk.liwip.com>
- Engineering support: <tech@keystonecommerce.in>

Release artifacts are public and inspectable. Every change is published under
a new version; published assets are not replaced in place.
