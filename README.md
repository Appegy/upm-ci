# upm-ci

Reusable GitHub Actions workflows for Appegy UPM packages.

Single source of truth for the signed-release and version-bump pipelines. Each package repo
keeps a thin wrapper that calls these; fixes here apply to every package at once.

## Layout expected in a package repo

- `src/` - the published package (package.json, Runtime/Editor/Tests).
- `Appegy.<Name>.Lab/` - the Unity dev project (auto-detected by the `*.Lab` name).
- `src/**/PackageInfo.cs` - holds `public const string Version` (auto-detected).
- `README.md` / `LICENSE` / `images/` at the repo root - folded into the published package.

## Release

Signs `src/` with the Unity UPM CLI, tags `vX.Y.Z`, publishes a GitHub Release with auto-notes.
Caller `.github/workflows/release.yml`:

```yaml
name: Release
on: workflow_dispatch
jobs:
  release:
    uses: Appegy/upm-ci/.github/workflows/release.yml@v1
    secrets: inherit
```

Requires secrets `UPM_SERVICE_ACCOUNT_KEY_ID`, `UPM_SERVICE_ACCOUNT_KEY_SECRET`, `UPM_ORG_ID`.

## Bump version

Computes the next version (patch/minor/major), syncs it across package.json, PackageInfo.cs,
README and the dev project's bundleVersion, then opens a PR. Caller
`.github/workflows/bump-version.yml`:

```yaml
name: Bump version
on:
  workflow_dispatch:
    inputs:
      type:
        description: "Version bump"
        type: choice
        options: [patch, minor, major]
        default: patch
jobs:
  bump:
    uses: Appegy/upm-ci/.github/workflows/bump-version.yml@v1
    with:
      type: ${{ inputs.type }}
    secrets: inherit
```

## Versioning

Reference by a stable tag (`@v1`) so a change here does not break every package's release at once.
