# Changelog

## 1.0.0 — 2026-08-16

Initial versioned contract. Callers pin `@v1` (major) or `@v1.0` (minor),
not a commit SHA.

Includes: Arch HEAD snapshot + `updpkgsums` on tag builds, `pacman-key --init`
before lsigning extra repos, COPR project-name fallback, fail-closed COPR and
`[mason]` publish after a successful GitHub Release.
