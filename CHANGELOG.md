# Changelog

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
