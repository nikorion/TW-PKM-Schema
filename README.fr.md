# TW-PKM-Schema

[English](README.md) · **Français**

Sources du plugin [TiddlyWiki](https://tiddlywiki.com) `$:/plugins/nikorion/pkm-schema`, le schéma de la suite *pkm* : les champs que peut porter un tiddler d'un wiki pkm, leurs vocabulaires contrôlés, icônes, libellés et traductions, et l'API par laquelle les autres plugins pkm les lisent — [TW-PKM-Fields](https://github.com/nikorion/TW-PKM-Fields) : l'éditeur, et les colonnes de [TW-Dynamic-Table](https://github.com/nikorion/TW-Dynamic-Table), qui lui-même ignore tout du schéma. Du wikitext pur, sans JavaScript, sans autre interface qu'un onglet de référence.

Ce README s'adresse à qui veut modifier le schéma. Ce que signifient les champs pour un utilisateur du wiki, et comment s'en servir en wikitext, relève du readme du plugin lui-même (`src/pkm-schema/language/<lang>/readme.tid`). Le wiki de démo `docs/TW-PKM-Schema-Wiki.html` appelle l'API en direct dans son Playground.

## Sommaire

- [Prise en main](#prise-en-main)
- [Organisation des sources](#organisation-des-sources)
- [Fonctionnement](#fonctionnement)
- [Extension](#extension)
- [Lors d'une mise à jour de TiddlyWiki](#lors-dune-mise-à-jour-de-tiddlywiki)
- [Licence](#licence)

## Prise en main

```sh
pnpm install
pnpm dev     # wiki de dev (wiki/) + rechargement à chaud ; l'URL (port libre aléatoire) s'affiche au démarrage
pnpm build   # dist/TW-PKM-Schema-Plugin.json + docs/TW-PKM-Schema-Wiki.html
```

`pnpm dev` pousse toute modification sous `src/pkm-schema` ou `wiki/tiddlers` directement dans l'onglet de navigateur déjà ouvert ; seul `plugin.info` redémarre le serveur. Ne pas recharger l'onglet pour voir une modification : il reviendrait tel que le serveur l'a chargé au démarrage. Arrêter avec deux Ctrl+C. Pour voir une modification à travers l'éditeur et les tableaux, lancer plutôt le wiki d'intégration de la suite (`../PKM`, `pnpm dev` là-bas) : il charge tous les plugins pkm et surveille toutes leurs sources.

Pour charger le plugin dans un autre wiki Node.js, créer un lien symbolique de `src/pkm-schema` vers `$TIDDLYWIKI_PLUGIN_PATH/nikorion/pkm-schema` et ajouter `"nikorion/pkm-schema"` au `tiddlywiki.info` de ce wiki. Requiert TiddlyWiki ≥ 5.3.0.

[↑ Retour au sommaire](#sommaire)

## Organisation des sources

| Chemin (sous `src/pkm-schema/`) | Rôle |
|---|---|
| `fields/<field>.tid` | une définition par champ, taguée `$:/tags/nikorion/pkm/Field` : `kind`, et pour un vocabulaire `list`, `groups`/`group-<slug>`, `blank-value`, `tone-<tone>` ; `applies-filter` quand le champ ne s'applique pas à tous les tiddlers |
| `tags/Field.tid` | le tag, dont le `list` donne l'ordre canonique des champs |
| `icons.multids` | un emoji par `<field>/<value>` |
| `colours/<field>.tid` | couleur de pastille par défaut d'un champ `list` (`$:/config/nikorion/pkm-schema/<field>/Colour` ; texte : palette claire, champ `dark` : palette sombre) |
| `api.tid` | l'API de lecture : fonctions `pkm-all-fields`, `pkm-field-*`, `pkm-applies-to`, `pkm-is-stale`, `pkm-actual-value`, `pkm-vocab-*` (`pkm-vocab-tooltip` comprise), `pkm-tones`, procédures `pkm-vocab-item` et `pkm-pill` |
| `language/<lang>/fields.multids` | `Field/<field>/Label`, `…/Description`, `…/Applies` facultatif |
| `language/<lang>/vocab.multids` | libellés `Vocab/<field>/<value>`, `…/Hint` facultatif, `…/Plural` pour les rôles, noms de groupe `Vocab/<field>/Group/<slug>` (+ `…/Hint`) |
| `reference.tid`, `reference/*.tid` | l'onglet *Fields*, généré à partir des définitions (le tableau des champs est aussi transclus par le readme) |
| `language/lingo.tid`, `language/<lang>/reference.multids`, `settings.multids` | les chaînes d'interface propres au plugin |

[↑ Retour au sommaire](#sommaire)

## Fonctionnement

- **Les définitions sont des données, l'API est le contrat.** Les consommateurs ne lisent jamais `fields/*` ni les chaînes de langue directement ; ils appellent les fonctions `pkm-*` (via l'opérateur `function`, qui lie les paramètres : une fonction personnalisée appelée comme opérateur direct ne le fait pas, TW 5.4). Renommer un champ de définition ou une clé de langue est une affaire interne ; renommer ou modifier une fonction de l'API casse la suite.
- **Seul le slug est stocké.** Icônes et libellés sont résolus au rendu, dans la langue du wiki, avec repli sur en-GB, puis sur le slug lui-même.
- **Valeur vide.** Le `blank-value` d'un vocabulaire représente le champ vide et n'est jamais stocké ; les consommateurs l'affichent tant que le champ est vide et vident le champ quand on le choisit.
- **Applicabilité.** `applies-filter` (évalué avec `currentTiddler`) indique où un champ a un sens. Les consommateurs masquent le champ ailleurs — sauf s'il contient une valeur, qu'ils continuent d'afficher, signalée.
- **Tonalités.** Une valeur de vocabulaire peut porter une tonalité (`tone-success: done` liste les valeurs de tonalité `success`). Elle dit comment se lit un tiddler qui la porte — terminé, mis de côté —, jamais comment le dessiner ; `pkm-tones` donne les tonalités d'un tiddler, à partir des seuls champs de vocabulaire qui s'y appliquent. Chaque consommateur associe une tonalité à un style qui lui est propre (PKM Fields : une classe de ligne de tableau `nk-dyntable-row-<tone>`). Tonalités utilisées : `success`, `danger`.
- **Les fonctions non résolues échouent en silence.** Un consommateur doit vérifier que le schéma définit un champ (`[[$:/plugins/nikorion/pkm-schema/fields/<field>]get[kind]]`) avant d'appeler une fonction `pkm-*` dessus : une fonction indéfinie appelée via `function` renvoie tous les tiddlers du wiki.

[↑ Retour au sommaire](#sommaire)

## Extension

- **Une valeur de vocabulaire** : son slug dans le `list` de `fields/<field>.tid` (pour `role`, aussi dans un `group-<slug>`, sinon elle est proposée avant le premier groupe), son icône dans `icons.multids`, son libellé (et son indication facultative) dans chaque `language/<lang>/vocab.multids`. Un nouveau rôle exige aussi son `…/Plural` dans chaque langue. Sans libellé, une valeur affiche son slug ; sans icône, pas d'icône.
- **Un champ** : une définition `fields/<field>.tid` (taguée, avec son `kind`), sa place dans le `list` de `tags/Field.tid`, son libellé et sa description dans chaque `language/<lang>/fields.multids` (et `…/Applies` avec un `applies-filter`), ses valeurs comme ci-dessus pour un vocabulaire, un `colours/<field>.tid` pour une liste. PKM Fields le prend en compte sans modification, dans l'éditeur comme dans les tableaux ; voir son README pour les retouches facultatives (rangée de l'éditeur, contrôle radio, liste des champs du core).
- **Une tonalité** : lister ses valeurs dans un champ `tone-<tone>` de la définition. Un nouveau nom de tonalité exige aussi un style dans chaque consommateur qui affiche les tonalités.
- **Un nouveau `kind`** est une modification du contrat : chaque consommateur a besoin d'un contrôle/modèle pour lui (PKM Fields : un contrôle d'éditeur et une cellule de tableau).

[↑ Retour au sommaire](#sommaire)

## Lors d'une mise à jour de TiddlyWiki

`pkm-pill` recopie le `tag-body-inner` du core (cascades de couleur et d'icône, `contrastcolour`), une procédure locale à `$:/core/ui/EditTemplate/tags` et donc inaccessible de l'extérieur : la comparer au nouveau core et resynchroniser.

[↑ Retour au sommaire](#sommaire)

## Licence

MIT — voir `LICENSE`.

[↑ Retour au sommaire](#sommaire)
