# Zed Query Inheritance Test Extensions

Two dev extensions for end-to-end testing of Zed's tree-sitter query
inheritance (`; inherits:` comments in query files), proposed in
[zed-industries/zed PR: language: Add query inheritance via inherits comments](https://github.com/zed-industries/zed/pull/63194).

- **`mybase/`** provides the language `MyBase` (`.mybase` files) whose
  `highlights.scm` starts with `; inherits: json` — pulling in the built-in
  JSON language's highlights — and adds one highlight of its own: `true` as a
  keyword.
- **`mydep/`** provides the language `MyDep` (`.mydep` files) whose
  `highlights.scm` starts with `; inherits: mybase` and adds one highlight of
  its own: `false` as a keyword.

Together they form the chain `MyDep` → `MyBase` → built-in `JSON`, exercising
transitive inheritance and inheritance from a built-in language. Each level
contributes one unmistakable marker (`false`, `true`, everything else), so a
glance at `sample.mydep` shows exactly which levels are active.

Both languages reuse Zed's built-in JSON grammar, so no grammar compilation is
needed at install time.

## Prerequisites

Build and run Zed from the PR branch:

```sh
git clone https://github.com/JacobWithACapitalJ/zed.git
cd zed
git switch query-inheritance
cargo run
```

On stock Zed the `; inherits:` line is an ordinary query comment, so `MyDep`
only ever highlights `false`. That contrast is itself a useful before/after
check.

## Test 1: inheritance

1. In the dev Zed, run `zed: install dev extension` and pick this repo's
   `mybase/` directory. Repeat for `mydep/`.
2. Open `sample.mydep`.
3. **Expected:** full JSON highlighting — strings, numbers, keys, `null`
   (inherited transitively from the built-in JSON language via `MyBase`) —
   plus `true` styled as a keyword (from `MyBase`) and `false` styled as a
   keyword (from `MyDep` itself). `debug: open syntax tree view` shows the
   captures directly.

## Test 2: missing base degrades gracefully

1. Start with only `mydep/` installed (uninstall `MyBase` if present and
   restart Zed, so `MyDep` loads without its base).
2. Open `sample.mydep`.
3. **Expected:** only `false` is highlighted — the whole inherited chain is
   missing — and the Zed log (`zed: open log`) contains a warning that `MyDep`
   inherits from an unknown language.

## Test 3: live re-expansion on registration change

Continuing from Test 2, with `sample.mydep` still open:

1. Run `zed: install dev extension` and pick `mybase/`.
2. **Expected:** the open buffer's highlighting upgrades without a restart —
   full JSON highlighting appears and `true` becomes a keyword, as `MyDep` is
   evicted and re-expanded against the newly registered base.
3. Uninstall the `MyBase` extension.
4. **Expected:** the buffer downgrades back to `false`-only highlighting.
