# WoW Addon Release Workflows

Reusable GitHub Actions workflow for building and publishing World of Warcraft addons.

## Setup in your addon repo

1. Add the caller workflow — copy [`templates/release.yml`](templates/release.yml) to `.github/workflows/release.yml` in your addon repo.
2. (Optional) Copy `.pkgmeta` and `packager-ignore.yml` from `templates/` if you want to customize defaults. If omitted, the reusable workflow generates/reads defaults for these (see [Optional files](#optional-files)).
3. (Optional) Set your `.toc`'s `## Version:` line to the `{current version}` placeholder (as in `templates/addon.toc`) if you want the workflow to auto-fill it from the release tag on each run. Leave it as a literal version to manage it yourself.
4. Define repo variables (`Settings → Secrets and variables → Actions → Variables`):
   - `ADDON_FOLDER`: folder/slug (required)
   - `ADDON_TITLE`: friendly name (optional)
5. Add provider secrets (`Settings → Secrets and variables → Actions → Secrets`)
   - `CURSEFORGE_TOKEN`
   - `WAGO_TOKEN`
   - `WOWI_TOKEN`
6. Commit and push. When you publish a release or run the workflow manually, your addon is packaged, the GitHub release notes are updated, and, unless skipped, uploads go to the providers that have tokens configured.

## Workflow Steps Overview

1. **Checkout** – fetches the addon repo so the workflow can read `.pkgmeta`, the TOC, etc.
2. **Resolve pkgmeta template values** (`resolve-pkgmeta`) – fills `.pkgmeta` (creates from template if missing), uses default ignore template if `packager-ignore.yml` is missing, and writes `zip-ignore.txt`.
3. **Update TOC version** (`update-toc`) – if `<addon_folder>.toc` has `## Version: {current version}`, replaces the placeholder with the version parsed from the triggering ref (the release tag, e.g. `v1.2.3` → `1.2.3`; a leading `v`/`V` is stripped). Skipped otherwise, leaving the `## Version:` line untouched.
4. **Build addon zip** (`build-zip`) – packages the addon folder into `<addon_folder>-<tag>.zip` using the generated exclude list.
5. **Generate release changelog** (`changelog-from-md`) – writes `CHANGELOG_RELEASE.md` from the matching section of `CHANGELOG.md`.
6. **Upload asset to GitHub release** – `softprops/action-gh-release` attaches the generated zip.
7. **Update GitHub release notes** (`update-release-notes`) – syncs the GitHub release body with `CHANGELOG_RELEASE.md` via `gh release edit`.
8. **Determine publish targets** (`determine-publish-targets`) – exposes which provider secrets (CF/Wago/WoWI) are set so the packager step can auto-skip when nothing is configured.
9. **Publish to addon services** – `BigWigsMods/packager@v2` uploads to CurseForge/Wago/WoWI when tokens are available and the release isn’t marked prerelease (or `skip_publish`).

## Optional files

If your repo omits `.pkgmeta`, the workflow creates it from the template. If your repo omits `packager-ignore.yml`, the workflow reads the default template from this repo without copying it into your addon repo. Add the files locally only if you need custom metadata or exclusions.
