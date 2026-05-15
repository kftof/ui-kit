---
name: ui-kit-prd-sync
description: Enrichit le PRD d'un projet avec le détail des pages et fonctionnalités présentes dans le UI kit mais non documentées dans le PRD initial. À invoquer après avoir ajouté/modifié des pages dans le kit via `ui-kit-creator` ou `ui-kit-editor`, pour que la pipeline de code-gen (Mirror Code, agent codeur custom) dispose dans le PRD du détail page-par-page (but, états, interactions, data, navigation, a11y) — pas seulement de la vue d'ensemble produit. Détecte les écarts kit ↔ PRD, propose un diff, et écrit dans une section dédiée `## Pages détaillées (synchronisées depuis le kit)` du PRD pour ne pas piétiner les sections rédigées à la main. Lecture seule sur le kit, écriture exclusivement dans le PRD.
---

# UI Kit PRD Sync — enrichir le PRD avec le détail des pages du kit

Le PRD initial décrit la vision produit, la cible, les flows à haut niveau. Mais quand on ajoute des pages au kit via `ui-kit-creator` / `ui-kit-editor` (Settings, Legal, About, Suggestions, mais aussi des pages métier qui émergent au design), le PRD est dépassé. La pipeline de code-gen (Mirror Code, agent codeur) lit le PRD pour comprendre l'intention de chaque page → si le PRD ne mentionne pas la page, le code généré est approximatif.

Ce skill **rapproche le PRD du kit** : il scanne `flows/`, extrait les conventions sémantiques de chaque page, et produit pour chaque page une **fiche structurée** consommable mécaniquement. Aucune modification du kit — uniquement écriture dans le PRD, dans une section dédiée qui ne piétine pas les sections rédigées à la main.

## 🎯 Quand utiliser ce skill

- Après avoir ajouté un flow / une page au kit qui n'était pas dans le PRD initial
- Avant de lancer Mirror Code ou un agent codeur sur le projet — pour s'assurer que le PRD reflète l'état réel du kit
- Audit kit ↔ PRD : "Quelles pages existent dans le kit mais pas dans le PRD ?"
- Onboarding d'un nouveau contributeur : générer la doc page-par-page automatiquement
- Migration d'un kit hérité : produire un PRD page-par-page rétroactivement

## 🚫 Quand NE PAS utiliser

- Le PRD n'existe pas du tout → c'est un cas pour `ui-kit-creator` qui pose les bonnes questions, pas ce skill
- Tu veux modifier le kit pour refléter le PRD → c'est l'inverse, utilise `ui-kit-editor`
- Tu veux auditer la qualité stratégique du kit → c'est `ui-kit-audit`

## 📋 Inputs requis

**Obligatoire** :
1. **Path du kit** — racine d'un kit avec `flows/`, `UI Kit.html` et un `PRD.md` (ou variante) déjà présent.

**Optionnel** :
2. **Path du PRD** — si non-standard. Par défaut le skill cherche dans l'ordre :
   - `<kit>/PRD.md`
   - `<kit>/PRD_*.md`
   - `<kit>/docs/PRD.md`
   - `<kit>/../PRD.md` (parent — typique quand le kit vit dans un sous-dossier `mockup/` du repo produit)
3. **Mode** :
   - `dry-run` (défaut) : produit un fichier `PRD_SYNC_PREVIEW.md` à côté du PRD avec le diff proposé, ne touche pas au PRD
   - `apply` : applique directement les ajouts au PRD (après dry-run validé par l'utilisateur)

## 🧭 Workflow

### Phase 1 — Reconnaissance

1. **Localiser le PRD** (ordre ci-dessus). Si rien trouvé → demander à l'utilisateur de pointer le path OU de confirmer qu'aucun PRD n'existe (auquel cas, abort avec message : "Pas de PRD à enrichir. Utilise `ui-kit-creator` pour générer un projet complet avec PRD initial.").

2. **Lire le PRD intégralement**. Identifier :
   - Les sections existantes (`## ...`)
   - Si une section `## Pages détaillées (synchronisées depuis le kit)` existe déjà → la **remplacer** au prochain apply (idempotent). Sinon → l'ajouter à la fin du PRD.
   - Les pages déjà mentionnées textuellement dans le PRD (grep des noms `flow/page` ou des titres de sections de page) — pour les diff.

3. **Inventorier le kit** :
   ```bash
   # Liste exhaustive flow_id × page_id
   find flows -type f -name "*.html" | sort
   ```
   Pour chaque fichier `flows/<NN>-<flow_id>/<page_id>.html` :
   - Extraire `flow_id` (strip `^\d+-`)
   - Extraire `page_id` (basename sans `.html`)

4. **Diff** : liste les pages présentes dans le kit ET absentes du PRD (par grep du nom dans le PRD existant). Ces pages sont les candidates à enrichir.

### Phase 2 — Extraction par page

Pour chaque page candidate, ouvrir le fichier HTML et extraire :

| Champ PRD | Source dans le HTML |
|---|---|
| `id_stable` | Le `data-screen-label` racine (`"NN <Page> default"`) → `flow_id/page_id` |
| `titre_visible` | Premier `<h1>` / `<h2>` / texte du `app-bar` / `header` |
| `but` | Inférence LLM à partir du titre + des CTAs + du contexte (titre `Réglages` → "Page de configuration utilisateur") |
| `nav_entrants` | Grep `data-nav-target="<flow_id>/<page_id>"` dans les autres fichiers HTML → liste des écrans qui mènent ici |
| `nav_sortants` | `data-nav-target` posés sur les éléments de cette page |
| `etats` | Tous les `data-screen-label` dans le fichier → list des states (default, loading, empty, error, success, etc.) |
| `cta` | `<button>` / `.btn` avec leur texte + leur `data-nav-target` ou `data-api-call` |
| `forms` | `<input>` / `<select>` / `<textarea>` / `<switch>` avec leur label associé et leur type |
| `api_endpoints` | Union de tous les `data-api-call` du fichier (héritage parent + set union enfants) |
| `runtime` | Présence de `data-runtime` → source (user-generated / server-driven / algorithm / local-storage) + shape |
| `patterns_ui` | Tous les `data-uses="ui:..."` distincts |
| `fonctions_natives` | Tous les `data-uses="native:..."` distincts |
| `a11y` | Présence de `data-a11y-label`, `data-a11y-live`, `data-a11y-hidden` → annoter "screen-reader-ready" |
| `hints_metier` | Tous les `data-hint` non vides |
| `os_chrome` | Présence de `data-os-chrome` → annoter "respecte SafeArea" |

**Règle critique** : ne pas inventer. Si un champ n'est pas inférable depuis le HTML, le laisser vide avec mention `_(non déterminable depuis le kit — à compléter manuellement)_` dans la fiche. Mirror Code préfère un trou explicite à une valeur hallucinée.

### Phase 3 — Génération des fiches

Pour chaque page candidate, produire une fiche au format suivant (consommable mécaniquement par Mirror Code et tout outil similaire) :

```markdown
### `<flow_id>/<page_id>` — <Titre humain de la page>

> _Fiche synchronisée depuis `flows/<NN>-<flow_id>/<page_id>.html` le YYYY-MM-DD._

**But** : <1 phrase>

**Navigation** :
- Arrivée depuis : `<flow>/<page>`, `<flow>/<page>` (ou `_première vue / pas d'écran source_`)
- Sorties vers : `<flow>/<page>`, `<flow>/<page>` (ou `_aucune navigation sortante_`)

**États** (cells) :
| State | Description |
|---|---|
| `default` | <inférence courte> |
| `loading` | <inférence ou "skeleton sur la liste principale"> |
| `empty` | <inférence ou "CTA d'amorce visible"> |
| `error` | <inférence ou "retry visible"> |
| `success` | <inférence ou "_(non présent)_"> |

**Contenu visible** :
- Titre : "<texte>"
- Sections : <liste>
- Composants clés : <classes widgets DS utilisées : `.btn-primary`, `.task-row`, etc.>

**Interactions** :
- CTA "<texte>" → `<flow>/<page>` (navigation)
- CTA "<texte>" → `POST /api/...` (action backend)
- Form fields : `<name> (<type>)`, `<name> (<type>)`
- Patterns UI : `ui:carousel`, `ui:bottom-sheet`, ...

**Données** :
- Endpoints consommés : `GET /...`, `POST /...` _(héritage parent + set union des enfants)_
- Runtime : `<source>` (`<shape>`) _(si applicable — ex: `server-driven (post)` pour un feed)_
- Empty state si vide : `<flow>/<page>:empty` _(si pointé via `data-runtime-empty`)_

**Fonctions natives** : `native:image-picker`, `native:camera`, ... _(ou `_aucune_`)_

**Accessibilité** :
- Labels screen reader posés sur : <inventaire>
- Annonces live (toasts, alerts) : <inventaire>
- Éléments décoratifs ignorés : <inventaire>

**Hints métier** : <reprise verbatim des `data-hint` non triviaux>

**Notes pour le code-gen** :
- <inférence sur ce qui doit être généré : "List dynamique server-driven, brancher sur cubit/repo via `_postRepo.list()`">
- <warnings si ambiguïté : "_le `data-runtime-source` pointe vers `api:GET /feed` mais aucun `data-api-call` n'est posé sur le conteneur — à clarifier_">
```

### Phase 4 — Production du livrable

**Mode `dry-run` (défaut)** :
1. Écrire `PRD_SYNC_PREVIEW.md` à côté du PRD avec :
   - **Préambule** : date, kit scanné, PRD source, liste des pages déjà documentées vs candidates
   - **Section générée complète** prête à être insérée dans le PRD
   - **Patch proposé** : "Insère cette section après `## <dernière section>` du PRD" OU "Remplace la section `## Pages détaillées (synchronisées depuis le kit)` existante"
2. Ne PAS toucher au PRD lui-même.
3. Demander à l'utilisateur de relire et lancer le mode `apply` si OK.

**Mode `apply`** :
1. Lire le PRD existant.
2. Si une section `## Pages détaillées (synchronisées depuis le kit)` existe → la remplacer intégralement (idempotent).
3. Sinon → l'ajouter à la fin du PRD, séparée par `---`.
4. Préserver toutes les autres sections du PRD inchangées (diff git doit montrer **uniquement** l'ajout/remplacement de cette section).
5. Supprimer `PRD_SYNC_PREVIEW.md` après apply réussi (le diff vit dans git).

## 📋 Section ajoutée au PRD — structure complète

La section ajoutée au PRD a toujours cette structure :

```markdown
---

## Pages détaillées (synchronisées depuis le kit)

> _Section auto-générée par le skill `ui-kit-prd-sync` le YYYY-MM-DD.
> Ne pas éditer manuellement — relancer le skill après modif du kit.
> Pour ajouter du contexte non-déductible du kit, éditer la sous-section
> "**Notes complémentaires (manuelles)**" en fin de chaque fiche._

Kit source : `<path/to/kit>`
Pages documentées : N
Dernière synchronisation : YYYY-MM-DD HH:MM

### `flow_id_1/page_id_1` — <Titre>
<fiche complète comme spécifié Phase 3>

#### Notes complémentaires (manuelles)
<bloc vide réservé à l'utilisateur — préservé entre runs>

### `flow_id_1/page_id_2` — <Titre>
<fiche complète…>

#### Notes complémentaires (manuelles)
<bloc préservé>

### `flow_id_2/page_id_1` — <Titre>
…
```

**Règle de préservation** : le bloc `#### Notes complémentaires (manuelles)` est **réécrit verbatim** d'un run à l'autre — c'est l'espace où l'utilisateur ajoute des nuances métier que le kit ne peut pas exprimer (raisons stratégiques, contraintes business, edge cases connus). Le skill ne touche jamais à son contenu, uniquement à la position (sous la fiche correspondante). Si une page disparaît du kit, son bloc Notes est conservé dans la section `## Pages obsolètes` à la fin (et l'utilisateur peut décider de purger).

## 🔍 Cas particuliers

### Page présente dans le PRD ET dans le kit
Si la page est déjà mentionnée dans une section manuelle du PRD (`## Onboarding`, `### Écran Welcome`, etc.) — **ne pas la dupliquer** dans la section auto-générée. Détection : grep du `flow_id` + `page_id` (ou titre humain proche) dans le PRD hors section auto. Si match → mentionner dans la fiche "Référence manuelle dans le PRD : `<section>` — synchronisé pour cohérence avec Mirror Code" et générer une fiche **abrégée** (juste l'inventaire des attributs sémantiques, pas la description complète déjà rédigée à la main).

### Page existante dans le PRD mais pas dans le kit
Out of scope de ce skill (il enrichit le PRD depuis le kit, pas l'inverse). Mentionner dans le préambule du dry-run : "_N pages mentionnées dans le PRD mais absentes du kit — voir liste en annexe pour invoquer `ui-kit-editor` et créer les écrans manquants._".

### Plusieurs PRD (PRD.md + PRD_v2.md)
Demander à l'utilisateur quel PRD est canonique. Ne pas enrichir les deux automatiquement.

### Kit sans `data-screen-label` (kit hérité non migré)
La détection des états par `data-screen-label` ne marche pas. Fallback : inférer un seul état `default` par fichier, et warner l'utilisateur : "_Kit hérité détecté — les états (loading, empty, error) ne sont pas exprimés par `data-screen-label`. Fiches générées avec un seul état `default`. Invoque `ui-kit-editor` recette 'Migrer un kit aux conventions sémantiques' pour exposer les états._".

### Pages avec `data-runtime` sans `data-runtime-mock` cohérent
Warner dans la fiche : "_Conteneur `data-runtime` détecté sans enfant `data-runtime-mock="true"` — le code-gen ne saura pas quel élément traiter comme placeholder. Cf. check ST-5 de `ui-kit-audit`._".

## 🚫 Anti-patterns

1. **Modifier le kit** — ce skill est strictement PRD-write. Toute modif du kit doit passer par `ui-kit-editor`.
2. **Halluciner des fonctionnalités absentes du kit** — si un champ n'est pas extractible, le laisser explicitement vide. Mirror Code préfère un `_(à compléter)_` à une valeur inventée qui passera silencieusement en prod.
3. **Réécrire les sections manuelles du PRD** — le skill écrit UNIQUEMENT dans `## Pages détaillées (synchronisées depuis le kit)`. Diff git après apply doit le confirmer.
4. **Toucher au bloc "Notes complémentaires (manuelles)"** — c'est l'espace utilisateur, préservé verbatim.
5. **Lancer en `apply` direct sans `dry-run`** — toujours passer par le preview pour permettre la validation humaine, sauf si l'utilisateur le demande explicitement (CI/CD, batch).

## ✅ Validation finale

Avant de marquer le skill comme terminé :
- [ ] Le PRD source est inchangé hors de la section `## Pages détaillées (synchronisées depuis le kit)` (vérifier via `git diff <PRD>` après apply)
- [ ] Chaque page de `flows/` a sa fiche dans la section, OU est documentée dans une section manuelle référencée
- [ ] Les blocs `#### Notes complémentaires (manuelles)` pré-existants sont préservés verbatim
- [ ] Aucune fiche ne contient de champ halluciné — les trous sont explicites
- [ ] Le préambule mentionne la date + le path du kit + le nombre de pages documentées
- [ ] Si mode `apply` : `PRD_SYNC_PREVIEW.md` a été supprimé après écriture

## 📤 Livrable

**Mode `dry-run`** : `PRD_SYNC_PREVIEW.md` à côté du PRD (préambule + section générée + patch proposé).

**Mode `apply`** : PRD enrichi avec la section `## Pages détaillées (synchronisées depuis le kit)`, idempotent au prochain run.

## 🔄 Re-invocation idempotente

Re-invoquer le skill après une modif du kit régénère la section avec les changements. Les blocs `#### Notes complémentaires (manuelles)` sont rapatriés depuis l'ancienne version pour chaque page existante. Les pages supprimées du kit voient leur fiche déplacée vers `## Pages obsolètes` (avec leur bloc Notes) — l'utilisateur décide de purger ou non.

## 🎯 Workflow utilisateur typique

```
1. ToF a créé un kit via ui-kit-creator avec un PRD initial vue d'ensemble
2. ToF ajoute via ui-kit-editor : flow Settings (settings + about + suggestions),
   flow Legal (cgu + privacy), 3 pages métier non détaillées dans le PRD
3. ToF invoque ui-kit-prd-sync sur le kit
4. Skill scanne flows/ → trouve 8 pages, 6 absentes du PRD
5. Skill génère PRD_SYNC_PREVIEW.md avec les 6 fiches structurées
6. ToF relit le preview, ajuste 2 trous "_(non déterminable)_" manuellement
7. ToF invoque le skill en mode apply
8. PRD enrichi, PRD_SYNC_PREVIEW.md supprimé, diff git propre
9. Mirror Code lit le PRD enrichi → code-gen pertinent par page
```

Le PRD devient la **source de vérité textuelle synchronisée avec le kit**, consommable par toute pipeline de code-gen sans deviner les états / endpoints / interactions de chaque page.
