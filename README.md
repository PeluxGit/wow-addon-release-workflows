# WoW Addon Release Workflows

Reusable GitHub Actions workflow for building and publishing World of Warcraft addons.

## Setup in your addon repo

1. Copy the template files:
   - `.pkgmeta` and `packager-ignore.yml` from `templates/`
   - `.github/workflows/release.yml` from `templates/release.yml`
2. Define repo variables (`Settings → Secrets and variables → Actions → Variables`):
   - `ADDON_FOLDER`: folder/slug (required)
   - `ADDON_TITLE`: friendly name (optional)
3. Add provider secrets (`Settings → Secrets and variables → Actions → Secrets`)
   - `CURSEFORGE_TOKEN`
   - `WAGO_TOKEN`
   - `WOWI_TOKEN`
4. Commit and push. When you publish a release or run the workflow manually, your addon is packaged, the GitHub release notes are updated, and, unless skipped, uploads go to the providers that have tokens configured.

## Workflow Steps Overview

1. **Checkout** – fetches the addon repo so the workflow can read `.pkgmeta`, the TOC, etc.
2. **Resolve pkgmeta template values** – custom action that fills `.pkgmeta`, ensures ignore lists, and writes `zip-ignore.txt`.
3. **Update TOC version** – custom action that patches the `## Version` line in the `.toc`.
4. **Build addon zip** – custom action that rsyncs files, applies the ignore list, and zips the addon folder.
5. **Generate release changelog** – custom action that extracts the version section from `CHANGELOG.md`.
6. **Upload asset to GitHub release** – `softprops/action-gh-release` attaches the generated zip.
7. **Update GitHub release notes** – custom action running `gh release edit` with the changelog.
8. **Determine publish targets** – custom action that checks which provider tokens exist.
9. **Publish to addon services** – `BigWigsMods/packager@v2` uploads to CurseForge/Wago/WoWI when tokens are available and the release isn’t marked prerelease (or `skip_publish`).

## Included Custom Actions

- `resolve-pkgmeta` – fills `.pkgmeta`, copies default templates if missing, and writes `zip-ignore.txt` from `packager-ignore.yml`.
- `update-toc` – updates the `## Version` line in the `.toc`.
- `build-zip` – packages the addon folder into `<addon_folder>-<tag>.zip` using the generated exclude list.
- `changelog-from-md` – writes `CHANGELOG_RELEASE.md` from the matching section of `CHANGELOG.md`.
- `update-release-notes` – syncs the GitHub release body with `CHANGELOG_RELEASE.md` via `gh release edit`.
- `determine-publish-targets` – exposes which provider secrets (CF/Wago/WoWI) are set so the packager step can auto-skip when nothing is configured.

## Template wrapper workflow

```yaml
name: Release Addon
on:
  release:
    types: [published]
  workflow_dispatch:
    inputs:
      skip_publish:
        description: "Only create zip, don't push to addon services"
        required: false
        type: boolean
        default: false

jobs:
  release:
    uses: PeluxGit/wow-addon-release-workflows/.github/workflows/addon-release.yml@v1
    with:
      addon_folder: ${{ vars.ADDON_FOLDER || github.event.repository.name }}
      addon_title: ${{ vars.ADDON_TITLE || vars.ADDON_FOLDER || github.event.repository.name }}
      skip_publish: ${{ inputs.skip_publish }}
    secrets: inherit
```

## Optional files

If your repo omits `.pkgmeta` or `packager-ignore.yml`, the workflow copies the default templates from this repo. Edit them in your addon repo if you need custom metadata or exclusions.
