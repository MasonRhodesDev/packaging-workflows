# packaging-workflows

Reusable GitHub Actions workflows shared by all MasonRhodesDev projects. One
place to fix CI for every repo.

## Workflows

| Workflow | Purpose | Key inputs |
|---|---|---|
| `rust-ci.yml` | fmt → clippy → test → build in an `archlinux` container | `pacman-deps`, `cargo-flags` |
| `arch-package.yml` | `makepkg` from a HEAD snapshot + namcap; optionally uploads to the tag's GitHub Release | `pkgbuild-dir`, `publish` |
| `rpm-check.yml` | SRPM from HEAD → `rpmbuild --rebuild` (runs `%check`) → gating rpmlint in a `fedora` container | `spec-name` |
| `release.yml` | On tag: Arch package → Release asset, POST COPR webhook, dispatch arch-repo republish | `pkgbuild-dir`; secrets `COPR_WEBHOOK_URL`, `ARCH_REPO_TOKEN` (both optional) |
| `security.yml` | cargo-deny (advisories/licenses/bans/sources) | — |

## Per-repo contract

Every packaged repo provides:

- `packaging/PKGBUILD` — `source=("$pkgname-$pkgver.tar.gz::https://github.com/MasonRhodesDev/$pkgname/archive/v$pkgver.tar.gz")`, `sha256sums=('SKIP')`. CI drops a `git archive` snapshot next to it so makepkg never downloads during PR builds.
- `packaging/<name>.spec` + `packaging/<name>.rpmlintrc` + `packaging/build-srpm.sh` — hyprstate-style vendored-cargo SRPM build with the four-way version gate (spec = Cargo.toml = Cargo.lock = PKGBUILD).
- `dist/` — canonical systemd units and other payload files with **packaged** paths (`/usr/bin`, never `%h/.local/bin`).
- `.copr/Makefile` — `make srpm` entry point so COPR rebuilds on its own GitHub webhook.
- Thin caller workflows in `.github/workflows/` (see below).

## Example callers

```yaml
# .github/workflows/ci.yml
name: CI
on: { push: { branches: [main] }, pull_request: }
jobs:
  ci:
    uses: MasonRhodesDev/packaging-workflows/.github/workflows/rust-ci.yml@main
    with:
      pacman-deps: pipewire alsa-lib   # if needed
```

```yaml
# .github/workflows/release.yml
name: Release
on: { push: { tags: ['v*'] } }
permissions: { contents: write }
jobs:
  release:
    uses: MasonRhodesDev/packaging-workflows/.github/workflows/release.yml@main
    secrets: inherit
```

## Release flow (any repo)

```mermaid
flowchart TD
    tag["tool repo: git push tag vX.Y.Z"]

    subgraph releasewf ["release.yml (workflow_call)"]
        archpkg["job arch-package: calls arch-package.yml — version gate (tag = pkgver = Cargo.toml), git archive HEAD snapshot, makepkg, namcap"]
        coprjob["job copr"]
        update["job update-arch-repo (needs: arch-package)"]
        archpkg --> update
    end

    tag -->|"thin caller: uses packaging-workflows/release.yml@main, secrets: inherit"| releasewf
    archpkg -->|"publish: true — gh release create + upload"| asset["GitHub Release asset (.pkg.tar.zst)"]
    coprjob -->|"POST COPR_WEBHOOK_URL (if set)"| coprbuild["COPR build"]
    update -->|"repository_dispatch: package-released with ARCH_REPO_TOKEN (else 6h cron)"| publish["arch-repo publish.yml"]
    asset -->|"gh release download '*.pkg.tar.zst'"| publish
    publish -->|"repo-add + deploy-pages"| pages["GitHub Pages x86_64 repo"]
    pages -->|"pacman -Syu"| pacuser["user pacman ([mason] repo)"]
```

1. Bump version (Cargo.toml + spec + PKGBUILD `pkgver`) — one commit.
2. `git tag vX.Y.Z && git push --tags`
3. CI builds and attaches the `.pkg.tar.zst`, COPR rebuilds off its webhook,
   and [arch-repo](https://github.com/MasonRhodesDev/arch-repo) republishes the
   pacman database (immediately with `ARCH_REPO_TOKEN`, otherwise on schedule).
