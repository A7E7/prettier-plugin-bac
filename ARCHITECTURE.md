# Architecture — prettier-plugin-bac

This repo is a **thin distribution shell**. It contains no formatter logic
of its own — the formatter lives in [vscode-bac][vscode-bac]
(`lsp/src/format/`) and is published here so non-VS-Code consumers can use
it through the standard Prettier plugin protocol.

[vscode-bac]: https://github.com/A7E7/vscode-bac

## Layout

```
prettier-plugin-bac/
├── package.json              # Published metadata + peer dep on prettier
├── README.md                 # Install + usage docs
├── ARCHITECTURE.md           # This file
├── CLAUDE.md                 # Doc-update + commit/push rules for Claude
├── scripts/build.js          # Compiles + bundles vscode-bac/lsp into dist/
└── dist/                     # Generated; published as the npm package contents
    ├── index.js              # Re-exports bacPrettierPlugin + formatBacText
    ├── index.d.ts            # Types
    ├── format/               # Copied from vscode-bac/lsp/out/format/
    └── script/               # Copied from vscode-bac/lsp/out/script/
```

## Build

```sh
node scripts/build.js
```

The script:
1. Runs `npm run build` in `../vscode-bac/lsp/` (or
   `$BAC_VSCODE_REPO/lsp/` if the env var is set), compiling the TS port
   of the BAC parser/AST and the formatter.
2. Copies `lsp/out/{script,format}/` into `dist/`.
3. Generates `dist/index.js` and `dist/index.d.ts` that re-export
   `bacPrettierPlugin` and `formatBacText`.

The script bails with a clear message if `vscode-bac` isn't a sibling and
`BAC_VSCODE_REPO` isn't set.

## Why a separate repo

Keeping this as its own repo (rather than a subdirectory of vscode-bac):

- Its release cadence is independent — formatter style changes in vscode-bac
  don't always need a new npm package release.
- Consumers `npm install prettier-plugin-bac` without cloning the much
  larger vscode-bac repo (which carries the parser, validators, navigation
  providers, VS Code extension, etc.).
- npm's "github URL" install form works for either repo independently.

Trade-off: keeping the formatter source in vscode-bac means a build-time
copy. We accept that — the alternative (extracting a third internal package
like `bac-script-core`) is more complexity than it's worth right now.

## Drift prevention

Every time vscode-bac's formatter changes, this repo must rebuild and ship
a new `dist/`. The protocol for doing so is in [CLAUDE.md](CLAUDE.md). The
build script is the single source of truth for what gets shipped — there's
no hand-curated wrapper code that could drift.
