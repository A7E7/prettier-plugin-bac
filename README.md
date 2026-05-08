# prettier-plugin-bac

Prettier plugin for **Blueprint as Code** (`.bac`) files. Opinionated style,
no config knobs.

`.bac` is the TypeScript-flavoured text representation of Unreal Engine
Blueprints used by the [BlueprintAsCode][plugin] plugin. See
[vscode-bac](https://github.com/A7E7/vscode-bac) for the language tooling
and [BlueprintAsCode][plugin] for the round-trip plugin itself.

[plugin]: https://github.com/A7E7/BlueprintAsCode

## Install

```sh
npm install --save-dev prettier prettier-plugin-bac
```

## Use

```sh
# Format one or more files in place
npx prettier --write "**/*.bac"

# Check without modifying
npx prettier --check "**/*.bac"
```

Wire it into your project's `.prettierrc`:

```json
{
  "plugins": ["prettier-plugin-bac"]
}
```

Programmatic:

```js
const prettier = require('prettier');

const formatted = await prettier.format(source, {
  parser:  'bac',
  plugins: [require('prettier-plugin-bac')],
});
```

Or skip Prettier's plugin layer and call the formatter directly:

```js
const { formatBacText } = require('prettier-plugin-bac');
const formatted = formatBacText(source);
```

## Style decisions (locked, no config)

- 2-space indent, 80-char print width
- No semicolons (BAC has none)
- Decorators on their own line above the member they decorate
- One blank line between top-level members
- Imports grouped at top, source-path alphabetical within a group
- Trailing commas on multi-line argument and parameter lists
- `if/else if/else` chains stay on the same line as the closing `}`
- Calls / generic calls / binary chains break only when they don't fit on
  one line; on break, one piece per line with trailing comma

The formatter is **idempotent**: `format(format(x)) === format(x)`.

## Editor integration

If you're using VS Code, install the
[Blueprint as Code (BAC)](https://github.com/A7E7/vscode-bac) extension
instead — it bundles the LSP (which includes this formatter) and gives you
format-on-save with zero config. This npm package is for CI, pre-commit
hooks, AI agent loops, and any other non-VS-Code use.

## Related

- [vscode-bac](https://github.com/A7E7/vscode-bac) — VS Code extension + LSP
- [BlueprintAsCode][plugin] — the UE plugin (paid, marketplace)

## License

MIT
