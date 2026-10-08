# TW-PKM-Schema

**English** · [Français](README.fr.md)

Source of the [TiddlyWiki](https://tiddlywiki.com) plugin `$:/plugins/nikorion/pkm-schema`, the schema of the *pkm* suite: the fields a tiddler of a pkm wiki can carry, their controlled vocabularies, icons, labels and translations, and the API the other pkm plugins read them through — [TW-PKM-Fields](https://github.com/nikorion/TW-PKM-Fields): the editor, and the columns of [TW-Dynamic-Table](https://github.com/nikorion/TW-Dynamic-Table), which itself knows nothing of the schema. Pure wikitext, no JavaScript, no UI of its own but a reference tab.

This README is for whoever wants to change the schema. What the fields mean for a wiki user, and how to use them in wikitext, is the plugin's own readme (`src/pkm-schema/language/<lang>/readme.tid`). The [online demo](https://nikorion.github.io/TW-PKM-Schema/) calls the API live in its Playground.

## Getting started

```sh
pnpm install
pnpm dev     # dev wiki (wiki/) + hot reload; the URL (random free port) is printed on start
pnpm build   # dist/TW-PKM-Schema-Plugin.json + docs/ (demo wiki, published by CI)
```

`pnpm dev` pushes any edit under `src/pkm-schema` or `wiki/tiddlers` straight into the browser tab already open; only `plugin.info` restarts the server. Do not reload the tab to see a change: it would come back as the server loaded it at boot. Stop with Ctrl+C twice. To see a change through the editor and the tables, run the suite's integration wiki instead (`../PKM`, `pnpm dev` there): it loads every pkm plugin and watches all their sources.

To load the plugin in another Node.js wiki, symlink `src/pkm-schema` as `$TIDDLYWIKI_PLUGIN_PATH/nikorion/pkm-schema` and list `"nikorion/pkm-schema"` in that wiki's `tiddlywiki.info`. Requires TiddlyWiki ≥ 5.3.0.

## Source layout

| Path (under `src/pkm-schema/`) | Role |
|---|---|
| `fields/<field>.tid` | one definition per field, tagged `$:/tags/nikorion/pkm/Field`: `kind`, and for a vocabulary `list`, `groups`/`group-<slug>`, `blank-value`, `tone-<tone>`; `applies-filter` when the field does not apply to every tiddler |
| `tags/Field.tid` | the tag, whose `list` gives the canonical order of the fields |
| `icons.multids` | one emoji per `<field>/<value>` |
| `colours/<field>.tid` | fallback pill colour of a `list` field (`$:/config/nikorion/pkm-schema/<field>/Colour`; text: light palette, `dark` field: dark palette) |
| `api.tid` | the read API: `pkm-all-fields`, `pkm-field-*`, `pkm-applies-to`, `pkm-is-stale`, `pkm-actual-value`, `pkm-vocab-*` (`pkm-vocab-tooltip` included), `pkm-tones` functions, `pkm-vocab-item` and `pkm-pill` procedures |
| `language/<lang>/fields.multids` | `Field/<field>/Label`, `…/Description`, optional `…/Applies` |
| `language/<lang>/vocab.multids` | `Vocab/<field>/<value>` labels, optional `…/Hint`, role `…/Plural`, group names `Vocab/<field>/Group/<slug>` (+ `…/Hint`) |
| `reference.tid`, `reference/*.tid` | the *Fields* tab, generated from the definitions (the field table is also transcluded by the readme) |
| `language/lingo.tid`, `language/<lang>/reference.multids`, `settings.multids` | the plugin's own UI strings |

## How it works

- **Definitions are data, the API is the contract.** Consumers never read `fields/*` or the language strings directly; they call the `pkm-*` functions (through the `function` operator, which binds parameters: a custom function called as a direct operator does not, TW 5.4). Renaming a definition field or a language key is internal; renaming or changing an API function breaks the suite.
- **Only the slug is stored.** Icons and labels are resolved at render time, in the wiki's language, falling back to en-GB, then to the slug itself.
- **Blank value.** A vocabulary's `blank-value` stands for the empty field and is never stored; consumers show it while the field is empty and clear the field when it is chosen.
- **Applicability.** `applies-filter` (evaluated with `currentTiddler`) says where a field means something. Consumers hide the field elsewhere — unless it holds a value, which they keep showing, flagged.
- **Tones.** A vocabulary value may carry a tone (`tone-success: done` lists the values with the `success` tone). It says how a tiddler holding it reads — done, set aside — never how to draw it; `pkm-tones` gives a tiddler's tones, from the vocabulary fields that apply to it only. Consumers map a tone to a style of their own (PKM Fields: a table row class `nk-dyntable-row-<tone>`). Tones in use: `success`, `danger`.
- **Unresolved functions fail silently.** A consumer must check that the schema defines a field (`[[$:/plugins/nikorion/pkm-schema/fields/<field>]get[kind]]`) before calling any `pkm-*` function on it: an undefined function called through `function` returns every tiddler of the wiki.

## Extending

- **A vocabulary value**: its slug in the `list` of `fields/<field>.tid` (for `role`, also in a `group-<slug>`, or it is offered before the first group), its icon in `icons.multids`, its label (and optional hint) in each `language/<lang>/vocab.multids`. A new role also needs its `…/Plural` in each language. Without a label a value shows its slug; without an icon, no icon.
- **A field**: a `fields/<field>.tid` definition (tagged, with its `kind`), its place in the `list` of `tags/Field.tid`, its label and description in each `language/<lang>/fields.multids` (and `…/Applies` with an `applies-filter`), its values as above for a vocabulary, a `colours/<field>.tid` for a list. PKM Fields picks it up with no change, in the editor and in the tables; see its README for the optional touches (editor row, radio control, core field list).
- **A tone**: list its values in a `tone-<tone>` field of the definition. A new tone name also needs a style in each consumer that shows tones.
- **A new `kind`** is a change to the contract: every consumer needs a control/template for it (PKM Fields: an editor control and a table cell).

## On a TiddlyWiki upgrade

`pkm-pill` copies the core's `tag-body-inner` (colour and icon cascades, `contrastcolour`), a procedure local to `$:/core/ui/EditTemplate/tags` and so unreachable from outside: diff it against the new core and resync.

## Installation

**Live demo**: [https://nikorion.github.io/TW-PKM-Schema/](https://nikorion.github.io/TW-PKM-Schema/) — try the plugin before installing it.

**From the nikorion plugin library** (TiddlyWiki then offers each new version as an update):

1. In your wiki, create a tiddler tagged `$:/tags/PluginLibrary`, with a field `url` set to `https://nikorion.github.io/tw-dev/library/index.html` and a `caption` such as `nikorion`.
2. Open *Control Panel → Plugins → Get more plugins*, choose the nikorion library and install **PKM Schema**.

**By hand**: download [`TW-PKM-Schema-Plugin.json`](https://nikorion.github.io/TW-PKM-Schema/TW-PKM-Schema-Plugin.json) and drag it onto your wiki.

Requires TiddlyWiki ≥ 5.3.0.

## License

MIT — see `LICENSE`.
