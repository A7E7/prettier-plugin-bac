# Claude instructions — prettier-plugin-bac

This repo is a thin distribution shell around the formatter that lives in
[vscode-bac][vscode-bac]. There's no formatter logic here — only packaging,
docs, and a build script. Don't edit the formatter from here; edit it in
vscode-bac and rebuild.

[vscode-bac]: https://github.com/A7E7/vscode-bac

## Shared-state map (this repo's only sibling)

| Touched in `vscode-bac`                    | Effect here                                                                  |
|--------------------------------------------|------------------------------------------------------------------------------|
| `lsp/src/script/**` (lexer/parser/AST/diagnostics) | `dist/` drifts — rerun `node scripts/build.js`                       |
| `lsp/src/format/**` (formatter)            | `dist/` drifts — rerun `node scripts/build.js`                              |
| Formatter output style (anything observable in formatted text) | `package.json` `version` bump (semver: patch=fix, minor=new style choice, major=breaking shape) + `CHANGELOG.md` (start one on first non-trivial change) |
| `lsp/package.json` deps that affect the formatter at runtime | mirror in this repo's `peerDependencies` if it's a runtime dep        |

Anything else lives in `vscode-bac` and doesn't touch this repo.

## Documentation update protocol

| Change                                          | Update                                                              |
|-------------------------------------------------|---------------------------------------------------------------------|
| Bump version (formatter style change in vscode-bac) | `package.json` `version` + add a `CHANGELOG.md` (don't have one yet — start it on first release) |
| Change `scripts/build.js` (e.g. new dep)        | `ARCHITECTURE.md` *Build* + `README.md` *Install* if it affects users |
| Change README install/usage instructions        | `README.md` only — keep it terse; deep notes go in `ARCHITECTURE.md`  |

For everything else, the change probably belongs in vscode-bac, not here.

## Build & ship cycle

```sh
# Bump vscode-bac formatter, then in this repo:
node scripts/build.js                # rebuild dist/ from sibling vscode-bac
node -e "const p=require('prettier');const x=require('./dist');console.log(typeof p.format)"   # smoke-test
# bump package.json "version"
# update CHANGELOG.md (when it exists)
git add -A && git commit -m "release X.Y.Z: <one-line>"
git push
npm publish --access public          # (when the user is ready to publish)
```

## Commit / push protocol

- **Auto-commit** after every shippable change (build clean + smoke-tested).
  One slice = one commit. Use a deliberate message — readers should
  understand the WHY without diffing.
- **Auto-push** to `main` after committing. This repo is public, solo dev,
  no review gate; pushing makes work visible on GitHub for review.
- **Don't run `npm publish` autonomously.** Publishing is an explicit user
  action — it goes to the npm registry where releases are immutable.

## Cross-repo coordination

When a vscode-bac formatter change ships:

1. Land + push the change in vscode-bac.
2. In this repo: re-run `node scripts/build.js` so `dist/` reflects the new
   formatter behaviour.
3. Bump `package.json` version (semver: patch for bug fixes, minor for new
   format choices, major for any breaking output change — the canonical
   formatted output is wire-stable).
4. Commit + push.
5. Tell the user the package is ready to publish.

The build script (`scripts/build.js`) checks for the sibling vscode-bac repo
and bails if it's missing — no silent fallback to a stale dist.
