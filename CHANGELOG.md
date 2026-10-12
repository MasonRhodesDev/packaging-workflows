# Changelog

## 1.3.2 — 2026-10-12

1.3.1 put `DisableSandbox` at the end of `pacman.conf`, which lands inside
whatever repo section is last — `[aur]` in the ARM image — where pacman
ignores it with only a warning, so the Landlock failure persisted. It now
goes into `[options]`, and the step asserts with `pacman-conf DisableSandbox`
so a wrong section fails the step instead of passing a grep of the raw file.

## 1.3.1 — 2026-10-12

Fixes the aarch64 leg added in 1.3.0, which could not run pacman at all:
pacman 7 downloads as the unprivileged `alpm` user behind a Landlock ruleset,
and the arm64 runner's container cannot apply it ("Operation not permitted"),
so database sync failed immediately. The ARM leg now sets `DisableSandbox` in
`pacman.conf` — not `--disable-sandbox`, because `makepkg -s` calls pacman
itself and would not inherit a flag. The x86_64 leg is untouched and keeps
the sandbox.

## 1.3.0 — 2026-10-12

arch-package.yml builds aarch64 alongside x86_64. A `plan` job intersects the
new `architectures` input (default x86_64 + aarch64) with the PKGBUILD's own
`arch=()`, so a project opts in by editing that one line and a project still
on `arch=('x86_64')` keeps building exactly one leg — its release gate is
unaffected. `arch=('any')` builds once, because arch-repo copies arch=any
packages into every architecture's database.

The ARM leg runs natively on `ubuntu-24.04-arm` (free for public
repositories) in `menci/archlinuxarm:base-devel`, pinned by digest; Arch
publishes no official aarch64 image. Artifacts are now named
`arch-package-<arch>` and release.yml collects them with a pattern.

Needed because the [mason] repo now serves a per-architecture directory
(arch-repo#2) and an M1 MacBook Air on Asahi Alarm wants the same desktop
packages the x86 machines get.

## 1.2.0 — 2026-08-24

`copr-chroots` input on release.yml: declares the COPR build-target set;
used at project creation and enforced (copr-cli modify) on every release.
Empty keeps prior behavior. First consumer: waybar-workspace-buttons,
mirroring nett00n/hyprland's targets (f43/f44/f45/rawhide, x86_64+aarch64).

## 1.1.0 — 2026-08-24

`extra-copr-repos` input on release.yml / rpm-check.yml: whitespace-separated
owner/project COPR repos enabled in the rpm-check container (so BuildRequires
outside Fedora proper resolve) and attached to the COPR project as external
repos on submit. First consumer: waybar-workspace-buttons, whose
hyprland-workspace-zones subpackage BuildRequires pkgconfig(hyprland) from
nett00n/hyprland.

## 1.0.0 — 2026-08-16

Initial versioned contract. Callers pin `@v1` (major) or `@v1.0` (minor),
not a commit SHA.

Includes: Arch HEAD snapshot + `updpkgsums` on tag builds, `pacman-key --init`
before lsigning extra repos, COPR project-name fallback, fail-closed COPR and
`[mason]` publish after a successful GitHub Release.
