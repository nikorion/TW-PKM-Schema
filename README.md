# TW-KMS-Ontology

Source of the [TiddlyWiki](https://tiddlywiki.com) plugin `$:/plugins/nikorion/kms-ontology`, the ontology of the *kms* suite: the fields a tiddler of a kms wiki can carry, their controlled vocabularies, icons, labels and translations, and the API the other kms plugins read them through — [TW-Base-Fields](https://github.com/nikorion/TW-Base-Fields) (editor) and [TW-Dynamic-Table](https://github.com/nikorion/TW-Dynamic-Table) (table columns). Pure wikitext, no JavaScript, no UI of its own but a reference tab.

This README is for whoever wants to change the ontology. What the fields mean for a wiki user, and how to use them in wikitext, is the plugin's own readme (`src/kms-ontology/language/<lang>/readme.tid`). The demo wiki `docs/TW-KMS-Ontology-Wiki.html` calls the API live in its Playground.

## Getting started

```sh
pnpm install
pnpm dev     # dev wiki (wiki/) + hot reload; the URL (random free port) is printed on start
pnpm build   # dist/TW-KMS-Ontology-Plugin.json + docs/TW-KMS-Ontology-Wiki.html
```

`pnpm dev` pushes any edit under `src/kms-ontology` or `wiki/tiddlers` straight into the browser tab already open; only `plugin.info` restarts the server. Do not reload the tab to see a change: it would come back as the server loaded it at boot. Stop with Ctrl+C twice. To see a change through the editor and the tables, run the suite's integration wiki instead (`../KMS`, `pnpm dev` there): it loads every kms plugin and watches all their sources.

To load the plugin in another Node.js wiki, symlink `src/kms-ontology` as `$TIDDLYWIKI_PLUGIN_PATH/nikorion/kms-ontology` and list `"nikorion/kms-ontology"` in that wiki's `tiddlywiki.info`. Requires TiddlyWiki ≥ 5.3.0.

## Source layout

| Path (under `src/kms-ontology/`) | Role |
|---|---|
| `fields/<field>.tid` | one definition per field, tagged `$:/tags/nikorion/kms/Field`: `kind`, and for a vocabulary `list`, `groups`/`group-<slug>`, `blank-value`; `applies-filter` when the field does not apply to every tiddler |
| `tags/Field.tid` | the tag, whose `list` gives the canonical order of the fields |
| `icons.multids` | one emoji per `<field>/<value>` |
| `colours/<field>.tid` | fallback pill colour of a `list` field (`$:/config/nikorion/kms-ontology/<field>/Colour`; text: light palette, `dark` field: dark palette) |
| `api.tid` | the read API: `kms-fields`, `kms-field-*`, `kms-vocab-*` functions, `kms-vocab-item` and `kms-pill` procedures |
| `language/<lang>/fields.multids` | `Field/<field>/Label`, `…/Description`, optional `…/Applies` |
| `language/<lang>/vocab.multids` | `Vocab/<field>/<value>` labels, optional `…/Hint`, role `…/Plural`, group names `Vocab/<field>/Group/<slug>` (+ `…/Hint`) |
| `reference.tid`, `reference/*.tid` | the *Fields* tab, generated from the definitions (the field table is also transcluded by the readme) |
| `language/lingo.tid`, `language/<lang>/reference.multids`, `settings.multids` | the plugin's own UI strings |

## How it works

- **Definitions are data, the API is the contract.** Consumers never read `fields/*` or the language strings directly; they call the `kms-*` functions (through the `function` operator, which binds parameters: a custom function called as a direct operator does not, TW 5.4). Renaming a definition field or a language key is internal; renaming or changing an API function breaks the suite.
- **Only the slug is stored.** Icons and labels are resolved at render time, in the wiki's language, falling back to en-GB, then to the slug itself.
- **Blank value.** A vocabulary's `blank-value` stands for the empty field and is never stored; consumers show it while the field is empty and clear the field when it is chosen.
- **Applicability.** `applies-filter` (evaluated with `currentTiddler`) says where a field means something. Consumers hide the field elsewhere — unless it holds a value, which they keep showing, flagged.
- **Unresolved functions fail silently.** A consumer must check that the ontology defines a field (`[[$:/plugins/nikorion/kms-ontology/fields/<field>]get[kind]]`) before calling any `kms-*` function on it: an undefined function called through `function` returns every tiddler of the wiki.

## Extending

- **A vocabulary value**: its slug in the `list` of `fields/<field>.tid` (for `role`, also in a `group-<slug>`, or it is offered before the first group), its icon in `icons.multids`, its label (and optional hint) in each `language/<lang>/vocab.multids`. A new role also needs its `…/Plural` in each language. Without a label a value shows its slug; without an icon, no icon.
- **A field**: a `fields/<field>.tid` definition (tagged, with its `kind`), its place in the `list` of `tags/Field.tid`, its label and description in each `language/<lang>/fields.multids` (and `…/Applies` with an `applies-filter`), its values as above for a vocabulary, a `colours/<field>.tid` for a list. Base Fields and Dynamic Table pick it up with no change; see their READMEs for the optional touches (editor row, core field list).
- **A new `kind`** is a change to the contract: every consumer needs a control/template for it.

## On a TiddlyWiki upgrade

`kms-pill` copies the core's `tag-body-inner` (colour and icon cascades, `contrastcolour`), a procedure local to `$:/core/ui/EditTemplate/tags` and so unreachable from outside: diff it against the new core and resync.

## License

MIT — see `LICENSE`.
