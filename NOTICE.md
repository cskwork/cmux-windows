# NOTICE

This repository, `cskwork/cmux-windows`, is a public maintenance fork of `mkurman/cmux-windows`.

## Provenance

- **Original concept** — `manaflow-ai/cmux`. A macOS-native terminal for AI coding agents (Swift + libghostty, GPL-3.0-or-later). https://github.com/manaflow-ai/cmux
- **Upstream Windows-native reimplementation** — `mkurman/cmux-windows`. C# WPF + ConPTY, MIT. https://github.com/mkurman/cmux-windows
- **This fork** — `cskwork/cmux-windows`. Public maintenance mirror by cskwork.
- **Fork point** — commit `974b718` (Merge PR #6 from Nolyo/main, taken from upstream `main`).
- **Fork date** — 2026-05-19.

## License

The upstream MIT LICENSE is preserved verbatim. `Copyright (c) 2026 Mariusz Kurman` remains the canonical upstream copyright line and is not modified by this fork.

New contributions made in this fork are licensed under MIT alongside the upstream, and may add a line of the form:

```
Copyright (c) 2026 cskwork contributors
```

added next to — never replacing — the upstream copyright.

## Why this fork exists

- Maintain a stable, publicly hosted, cskwork-branded mirror of `cmux-windows`.
- Enable later optional work on cmux V1 + V2 socket-protocol parity (post-fork roadmap, not promised).
- Provide an MIT-licensed, native-Windows alternative to AGPL-licensed cmux ports (e.g., `amirlehmam/wmux`), without contaminating downstream code with copyleft network-use clauses.

## Non-claims

- "cmux" is the name of the original project by `manaflow-ai`. Use of the name here is descriptive of the lineage and follows upstream conventions. This fork makes no trademark claim over "cmux".
- This fork is not endorsed by `manaflow-ai` or `mkurman`.

## Syncing from upstream

Pull upstream changes via:

```sh
git remote add upstream https://github.com/mkurman/cmux-windows.git  # if not already set
git fetch upstream
git merge upstream/main
```

GitHub's web UI "Sync fork" button is the supported low-friction path.
