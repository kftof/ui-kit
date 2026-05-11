---
name: ui-kit-editor
description: Modifie, étend ou audite un UI kit / design system HTML existant (peu importe le produit) en préservant la cohérence. À invoquer pour ajouter un écran, modifier un composant, auditer un kit, créer une variante, ou refactorer un package design existant.
---

# UI Kit Editor — Modifier un kit existant

Intervient sur un package design HTML déjà existant sans casser sa cohérence visuelle. Agnostique du produit : fonctionne sur n'importe quel kit tant qu'il a une structure tokens + composants.

## 📋 Contexte à charger

Un kit "complet" = **tokens + composants partagés + iconographie** réifiés. Avant toute modif, charger ces trois piliers.

**Obligatoire** :
1. **Tokens** — `ds/tokens.css` (ou `design-system.css`, `variables.css`…). Variables couleurs/spacing/type/radii/shadows.
2. **Catalogue de composants** — `ds/components.css`. Sinon : repérer où vivent les classes widgets (souvent dans `tokens.css` ou inline dans `UI Kit.html`).
3. **Librairie d'icônes** — `ds/assets/icons/<name>.svg` (un fichier par icône) + `ds/assets/icons/icons.svg` (sprite consolidé source). Dans les HTML, **chaque icône est inlinée en plein SVG** avec un attribut `data-asset="ds/assets/icons/<name>.svg"` qui pointe vers le fichier source — c'est la clé de matching pour les outils de code-gen. Sinon : si SVG inline sans `data-asset`, ou sprite externe `<use href="ds/...">`, c'est un kit obsolète à consolider.
4. **Assets non-icônes** — vérifier la présence et le contenu de :
   - `ds/assets/brand/` (logos, app-icon, favicon, splash)
   - `ds/assets/illustrations/` (scènes vectorielles : empty states, onboarding, decorative)
   - `ds/assets/images/` (photos, placeholders, hero, textures raster)
   - `ds/assets/fonts/` (fonts custom)
   Si un de ces dossiers manque mais qu'il devrait exister (ex: illustration d'empty state ad-hoc dans un Flow), le créer + migrer.
5. **Page UI Kit unique** — `UI Kit.html` (tokens + brand + iconographie + illustrations + images + composants en rendu visuel).
6. Le(s) fichier(s) HTML à modifier.

**Recommandé** :
6. `GUIDELINES.md` ou équivalent du projet
7. Le fichier HTML d'un écran existant similaire à ce qu'on veut faire

### Convention de nommage flows / pages / cells (à respecter)

Format machine-readable utilisé par les outils de code-gen :

| Niveau | Format | Exemple | ID dérivé |
|---|---|---|---|
| Flow | `flows/NN-<id>/` | `flows/01-tasks/` | `flow_id = "tasks"` (strip `/^\d+-/`) |
| Page | `<id>.html` ou `NN-<id>.html` (si ordre voulu) | `list.html` | `page_id = "list"` |
| État (cell) | `data-screen-label="NN <Page> <state>"` | `"01 List default"` | `state_id = "default"` (dernier mot lower) |
| Screen global | `<flow_id>/<page_id>[:<state_id>]` | `tasks/list:loading` | identifiant stable |

Toute modif (ajout d'un flow, d'une page, d'une cell d'état) doit respecter ces formats. Préfixe `NN-` obligatoire sur les dossiers flow, optionnel sur les pages (uniquement si ordre voulu : onboarding séquentiel par exemple).

### Conventions sémantiques machine-readable

En plus de `data-asset` et `data-screen-label`, **plusieurs attributs sémantiques** doivent être respectés / posés à chaque modif touchant les Flows :

| Attribut | Quoi | Vocabulaire |
|---|---|---|
| `data-os-chrome="<type>"` | Élément rendu par l'OS, à skip totalement (héritage parent → tous descendants skipés) | `device-frame · dynamic-island · notch · camera-cutout · status-bar · home-indicator · gesture-bar · keyboard · keypad` |
| `data-uses="native:<feature>"` | Fonction native du device (image-picker, camera, biometric…) | cf. UI_KIT_CONTRACT du contrat upstream — vocabulaire vivant |
| `data-uses="ui:<pattern>"` | Pattern UI au comportement non-évident (carousel, bottom-sheet, dialog, dot-indicator…) | idem |
| `data-uses-context="..."` | Texte libre paramétrant `data-uses` (multi-select, max items, options) | — |
| `data-hint="..."` | Hint sémantique métier libre (galerie BDD vs native, swipe behavior, etc.) | — |
| `data-nav-target="<flow>/<page>[:<state>]"` | CTA / row / lien qui navigue vers un autre écran. `+ data-nav-back="true"` pour le retour. | format `<flow_id>/<page_id>` ou `<flow_id>/<page_id>:<state_id>` ou `back` |
| `data-api-call="<METHOD>:/path[;...]"` | Élément / conteneur qui consomme un endpoint backend. Héritage parent + set union enfants. Skip sous `data-os-chrome`. | `METHOD` ∈ `GET\|POST\|PUT\|PATCH\|DELETE`, path identique à OpenAPI, path params en `{name}`, multiples séparés par `;` — query params exclus (vivent dans le state du cubit) |

**Règles d'or** :
- Les SVG du chrome système (signal, wifi, battery, time) sont skipés par héritage parent `data-os-chrome="status-bar"` — ils **n'ont jamais** `data-asset`.
- Les patterns UI dont le comportement échappe au visuel statique (carousel, dialog, bottom-sheet, etc.) **doivent** porter `data-uses="ui:..."`.
- Les fonctions natives **doivent** porter `data-uses="native:..."` — l'agent codeur cherchera alors d'abord un widget partagé du projet, puis tombera sur l'API native si rien d'existant.
- Tout CTA / row / lien qui **navigue entre écrans** doit porter `data-nav-target` — le skill `ui-kit-prototype` consomme cet attribut pour générer un prototype interactif, et les outils de code-gen génèrent les routes Flutter directement à partir de l'inventaire.

### Si le kit a une organisation obsolète

Symptômes typiques sur kits anciens :
- composants redéfinis localement par Flow (au lieu de `ds/components.css`)
- assets directement sous `ds/icons/`, `ds/brand/` au lieu de `ds/assets/`
- icônes SVG inline **sans** `data-asset` (perte de traçabilité code-gen)
- sprite référencé en externe (`<use href="ds/assets/icons/icons.svg#…"/>`) qui casse en `file://`
- pages flow à la racine (`Flow 01 - Tasks.html`) au lieu de `flows/<flow>/<page>.html`
- pages doc séparées (`Components.html`, `Icons.html`) au lieu d'une seule `UI Kit.html`
- **éléments OS-chrome** (status-bar, dynamic-island, home-indicator, keyboard) sans `data-os-chrome` → l'agent codeur va générer des widgets pour eux (faux)
- **SVG du chrome** (signal, wifi, battery, time) qui portent `data-asset` → ils se retrouvent dans `contract.assets.icons[]` et polluent les imports de l'app
- **patterns UI à comportement non-évident** (carousel, dialog, bottom-sheet, etc.) sans `data-uses="ui:..."` → l'agent recode en custom au lieu d'utiliser le widget partagé / framework primitive
- **fonctions natives** (image-picker, camera, biometric) sans `data-uses="native:..."` → l'agent recode une UI custom
- **CTA / rows / liens qui naviguent** entre écrans sans `data-nav-target` → `ui-kit-prototype` ne peut pas créer de prototype interactif fonctionnel, et l'agent codeur devine au lieu de générer la bonne route
- **Pages data-driven (liste, détail, feed, form) sans `data-api-call`** → le code-gen génère un cubit `emit(initial)` placeholder ou infère le mauvais endpoint. Poser sur le conteneur le plus haut (`<main>` ou `.phone__screen`) qui consomme l'endpoint. Format strict `<METHOD>:/path`, path params en `{name}`, multiples séparés par `;` — query params **interdits** dans l'attribut
- **galeries BDD app vs native** ambiguës sans `data-hint` métier

**Ne pas faire semblant que tout est OK** — signaler à l'utilisateur, proposer une consolidation (cf. recette "Consolider un kit non-DRY") avant ou en parallèle de la modif demandée.

### Si le kit a des pages séparées Components.html / Icons.html

C'est une organisation obsolète. Le UI Kit doit être une **page unique** qui regroupe tokens + composants + icônes. Si tu rencontres ce split, proposer la consolidation en page unique (cf. recette dédiée).

## 🧭 Workflow en 4 phases

### Phase 1 — Reconnaissance (obligatoire)

Avant toute modif, **lire le kit** pour extraire :

1. **Tokens dispos** : lister les variables CSS existantes (couleurs, spacing, radii, shadows, type)
2. **Composants partagés** : ouvrir `ds/components.css` ; lister les classes (`.btn`, `.card`, `.task-row`, `.dialog`, `.sheet`, `.tile`, `.banner`, `.empty-state`…) et leurs modifiers.
3. **Icônes** : lister les fichiers dans `ds/assets/icons/` (un par icône, contenu Lucide canonique) + ouvrir `ds/assets/icons/icons.svg` (sprite source consolidé). Vérifier comment les HTML les utilisent : SVG inline avec `data-asset` (recommandé) OU sprite externe `<use href="ds/...">` (cassé en file://, à corriger).
4. **Convention `data-asset`** : grep `data-asset="ds/assets/` dans les HTML existants pour confirmer que le kit utilise bien cette convention. Si absente : c'est un kit obsolète, voir consolidation.
5. **Conventions sémantiques** : grep `data-os-chrome=`, `data-uses=`, `data-hint=`, `data-nav-target=` pour confirmer leur présence. Si absentes alors que des éléments concernés existent (status bar, carousel, dialog, image-picker, CTA navigants…), c'est un kit obsolète → proposer la recette "Migrer un kit aux conventions sémantiques".
6. **Graphe de navigation** (si `data-nav-target` présents) : pour chaque attribut, vérifier que la cible (`<flow_id>/<page_id>[:<state_id>]`) existe bien dans l'inventaire des écrans. Liens cassés → warning, à fixer en même temps que la modif.
7. **Doc UI Kit** : ouvrir `UI Kit.html` et vérifier que c'est bien une page unique (tokens + composants + iconographie). Si Components.html ou Icons.html existent en plus, proposer consolidation.
8. **Conventions de nommage** : BEM ? utility classes ? préfixes ?
9. **Structure de fichier** : comment les écrans sont organisés (cellules, canvas, sections…)
10. **Règles implicites** : lire GUIDELINES si présent, sinon inférer en observant
11. **`Prototype.html` éventuellement présent** à la racine : c'est un fichier généré par le skill `ui-kit-prototype`. Ne pas l'éditer à la main. Après la modif courante, signaler à l'utilisateur de re-générer (`ui-kit-prototype` à invoquer) si la modif a touché un Flow.

Ne jamais sauter cette phase. Les erreurs les plus courantes viennent de suppositions sur ce qui existe déjà dans `ds/`.

### Phase 2 — Plan

Avant d'écrire du HTML, produire un plan court :

```
## Plan : [résumé modif]
- Fichier(s) cible : [...]
- Cellule/section touchée : [...]
- Tokens utilisés : [...]
- Widgets réutilisés (depuis components.css) : [...]
- Widgets nouveaux à ajouter au catalogue : [...]
- Icônes utilisées (depuis icons.svg) : [...]
- Icônes nouvelles à ajouter (`ds/assets/icons/`) : [...]
- Impact DS : [aucun / extension components.css / extension icons.svg / extension de tokens / refonte]
```

Valider avec l'utilisateur si scope important. Pour petit fix, enchaîner direct.

### Phase 3 — Exécution

Appliquer la modif en respectant ces garde-fous :

#### Tokens uniquement
Toute valeur visuelle via variable CSS. Pas de hex/px hardcodé en dehors de la gamme DS.

#### Widgets depuis le catalogue uniquement
Aucun widget redéfini en local dans un Flow. Le markup d'un Flow ne contient que des classes du DS (`ds/components.css`) ou de la grille canvas/frame. Si quelque chose manque : ajouter au catalogue avant de l'utiliser, jamais après.

#### Icônes inlinées avec `data-asset` obligatoire
Toute icône utilisée dans un Flow ou dans `UI Kit.html` est inlinée en plein SVG avec `data-asset="ds/assets/icons/<name>.svg"` qui pointe vers le fichier source. Pattern :
```html
<svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor"
     stroke-width="2" stroke-linecap="round" stroke-linejoin="round"
     data-asset="ds/assets/icons/plus.svg">
  <path d="M5 12h14"/><path d="M12 5v14"/>
</svg>
```
Pas de `<use href>`, pas de SVG inline anonyme. Icône absente de `ds/assets/icons/` → la fetch depuis la source officielle, créer le fichier, **puis** l'inliner dans le HTML avec son `data-asset`.

#### Pas de duplication
Avant de créer un composant ou une icône, vérifier qu'ils n'existent pas déjà (même sous un autre nom). Grep agressivement `ds/components.css` et `ds/assets/icons/`.

#### Cohérence de nommage
Suivre la convention existante du projet. Pas de mix BEM/utility.

#### Ne pas casser l'existant
Si modif dans un fichier multi-écran, restreindre au périmètre nécessaire. Si CSS partagé touché, auditer tous les consommateurs.

#### Numérotation / ordre logique
Respecter la numérotation existante (ex: `data-screen-label="NN"`, sections ordonnées, etc.).

### Phase 4 — Vérification (visuelle obligatoire)

Avant de livrer, **screenshot via chrome-devtools MCP** + audit grep. Pas de "c'est livré" sans avoir VU le rendu.

#### Visuel via chrome-devtools MCP — viewport-by-viewport (PAS fullPage)

Les pages UI Kit denses font 8 000-15 000 px de haut. Un screenshot `fullPage: true` à cette taille (×2 sur Retina = 30 000 px) dépasse les capacités de lecture du modèle. Méthode :

```
1. mcp__chrome-devtools__new_page  url: file:///path/to/<fichier-modifié>.html

2. mcp__chrome-devtools__list_console_messages  types: ["error", "warn"]
   → DOIT être vide.

3. mcp__chrome-devtools__evaluate_script
   () => {
     const sections = Array.from(document.querySelectorAll('section.doc-section, .cell, .showcase'))
       .map(el => ({ id: el.id || el.dataset.screenLabel
                       || el.querySelector('.showcase__title, .cell-head .name')?.textContent,
                     top: el.offsetTop, height: el.offsetHeight }));
     return { sections };
   }

4. POUR CHAQUE SECTION (couvrir TOUTES, pas un échantillon) :
   a. evaluate_script: window.scrollTo({top: <SECTION_TOP - 20>, behavior: 'instant'});
      ⚠️ behavior: 'instant' OBLIGATOIRE (sinon scroll smooth → mauvaise zone capturée)
   b. take_screenshot  filePath: /tmp/<page>-<section>.png  (PAS fullPage)
   c. Read /tmp/<page>-<section>.png  → vérifier visuellement

5. Si dark mode supporté : basculer `data-theme="dark"` + re-screenshoter sections clés.
```

#### Cas spécial — Flow avec phones côte-à-côte (canvas 3-colonne)

Si la modif touche un Flow où le canvas affiche 3-5 phones en grille, le screenshot viewport global est insuffisant : chaque phone n'occupe que ~250-300 px dans le PNG → détails fins invisibles. **Cropper la grille en colonnes via PIL** :

```python
from PIL import Image
img = Image.open('/tmp/flow-fullpage.png')
w, h = img.size
n_cells = 3
col_w = w // n_cells
for i, name in enumerate(['default', 'loading', 'empty']):
    img.crop((i * col_w, 0, (i+1) * col_w, h)).save(f'/tmp/cell-{i+1}-{name}.png')
```

Lire chaque `cell-N-NAME.png` séparément. À ~1100 px par phone, scanner activement : centrage des éléments, overflow texte sur boutons, z-index sandwich, auto-scroll parasite, variables CSS undefined.

#### Validation XML des SVG modifiés/ajoutés

Avant le screenshot, valider tout SVG modifié ou ajouté :
```bash
xmllint --noout ds/assets/<famille>/<file>.svg
```
Tout échec (`& brut` au lieu de `&amp;`, `<` brut, attribut sans guillemets) doit être corrigé. Le browser refuse silencieusement les SVG mal formés en chargement externe via `<img>` → broken icon au rendu.

Si chrome-devtools indispo : kill du chrome existant + suppression `SingletonLock` puis retry. En dernier recours : `open` + demander confirmation au user, ne jamais livrer sans confirmation visuelle.

#### Checklist post-screenshot

- [ ] **Console clean** : zéro error/warning
- [ ] **Icônes visibles** dans la galerie iconographie ET dans les Flows (si une icône est vide → SVG inline incomplet ou paths cassés, vérifier que le `<svg>` contient bien tous les `<path>` du fichier source)
- [ ] **Pas de débordement visuel**
- [ ] **Responsive** aux breakpoints du projet
- [ ] **Cohérence avec les écrans voisins**
- [ ] **Tokens uniquement** (re-grep les hex dans ma modif → 0)
- [ ] **Widgets uniquement depuis le catalogue** (re-grep `<style>` local dans le Flow modifié → vide ou limité à canvas/frame)
- [ ] **Icônes inlinées avec `data-asset`** : tout `<svg>` app qui contient des `<path>` doit porter `data-asset="ds/assets/icons/<name>.svg"`. Grep `<svg(?:(?!data-asset).)*?<path` → 0 SAUF descendants d'un `data-os-chrome` (système, à skipper)
- [ ] **Aucun `<use href="ds/`** dans le markup (sprite externe banni)
- [ ] **OS-chrome marqué** : status-bar, dynamic-island, home-indicator, keyboard, gesture-bar (si présents) portent tous `data-os-chrome="<type>"`. Grep `class="status-bar"` → confirmer parent `data-os-chrome="status-bar"`. Idem dynamic-island, home-indicator. **Aucun `data-asset` sur les SVG enfants** de ces conteneurs (signal/wifi/battery sont du chrome système, pas des assets app).
- [ ] **`data-uses="ui:..."` posé** sur tous les patterns à comportement non-évident touchés par la modif : carousel, dialog, alert-dialog, bottom-sheet, action-sheet, app-bar, bottom-nav, tab-bar, dot-indicator, segmented-control, snackbar, tooltip, pull-to-refresh, swipe-actions, expansion-panel, etc.
- [ ] **`data-nav-target` posé** sur tous les CTA / rows / liens qui naviguent (boutons submit, task-row tappable, nav-action retour, link, etc.). Pour chaque target déclaré, vérifier que l'écran ciblé existe dans le kit (grep `data-screen-label` du fichier flow correspondant).
- [ ] **Si la modif touche un Flow** : signaler à l'utilisateur de re-générer le `Prototype.html` via `ui-kit-prototype` pour que le prototype reflète la modif.
- [ ] **`data-uses="native:..."` posé** sur toute fonction native sollicitée : image-picker, camera, file-picker, date-picker, map, share-sheet, biometric-auth, scanner, web-view, etc.
- [ ] **`data-uses-context`** présent quand le paramétrage est non-trivial (multi-select, max items, options spécifiques)
- [ ] **`data-hint` posé** sur les éléments dont la sémantique métier n'est pas évidente (galerie BDD vs native, swipe behavior, etc.)
- [ ] **Si nouveau widget** : ajouté à `components.css` ET documenté dans la section Composants de `UI Kit.html`
- [ ] **Si nouvelle icône** : fetched depuis source officielle (Lucide canonical), ajoutée à `ds/assets/icons/<name>.svg`, ajoutée au sprite consolidé `ds/assets/icons/icons.svg`, **et inlinée en plein SVG avec `data-asset`** dans les HTML qui l'utilisent (UI Kit + Flows concernés)
- [ ] **Pages flow sous `flows/<flow>/<page>.html`** — pas de fichier `Flow NN.html` à la racine
- [ ] **Pas de `<pre><code>` ajouté** dans la doc UI Kit
- [ ] **Copy cohérent** avec le ton existant (formel/familier)
- [ ] **Aucun terme technique dans la copy** ajoutée/modifiée : pas de nom de modèle IA (Sonnet/Opus/GPT…), pas de provider (Groq/RevenueCat/Anthropic…), pas d'acronyme infra (OCR/ASR/LLM/SDK/JWT…). Préférer les termes métier ("scans photo" pas "scans IA Sonnet 4.6", "synchro" pas "WebSocket sync").
- [ ] **Accessibilité** : contrastes, tailles, hit zones

## 🔧 Recettes courantes

### Ajouter un écran/cellule

1. Identifier le fichier du flow concerné
2. Grep le dernier identifiant utilisé (`data-screen-label`, numéro de section…)
3. Copier la cellule la plus proche du besoin
4. Incrémenter l'identifiant
5. Adapter le contenu via composants existants
6. Remplir les métadonnées (description, tag, etc.)

### Modifier un écran sans casser les voisins

1. Localiser la cellule précise
2. Scope les changements CSS avec préfixes spécifiques (`.foo-v2`)
3. Éviter de toucher aux classes partagées
4. Si inévitable : auditer tous les consommateurs

### Créer une variante (A/B)

1. **Ne pas remplacer l'existant** — toujours ajouter à côté
2. Suffixer l'identifiant : `13b`, `13c`, `21-v2`
3. Documenter dans `cell-desc` : "Variante explorant [axe]"
4. Permettre comparaison directe

### Ajouter un widget partagé au catalogue

1. Vérifier 2 fois qu'il n'existe pas déjà (grep `components.css` + section Composants de `UI Kit.html`, et tester si une combinaison de modifiers d'un widget existant suffit)
2. Choisir le bon niveau (atome / molécule / organisme)
3. Définir l'API : classes, modifiers, variantes, états (ex. `.card`, `.card--elevated`, `.card--interactive`)
4. **Ajouter le CSS dans `ds/components.css`** (utiliser uniquement des tokens, pas de valeurs en dur)
5. Ajouter les tokens nécessaires si manquants (impact DS — cf. recette dédiée)
6. **Documenter dans la section Composants de `UI Kit.html`** : preview visuelle des variantes/états + libellé monospace `.classname` discret. **Pas de `<pre><code>` avec markup HTML.**
7. Si pertinent, propager dans les flows qui consomment encore une version locale
8. **Screenshot post-modif** via chrome-devtools pour valider visuellement

### Ajouter un asset brand / illustration / image

Distinguer la famille avant tout :

| Famille | Quand | Dossier cible |
|---|---|---|
| **brand/** | logo, app-icon, favicon, splash, marque | `ds/assets/brand/` |
| **illustrations/** | scène vectorielle (empty state, onboarding, decoration) | `ds/assets/illustrations/` |
| **images/** | photo, placeholder, hero, texture raster | `ds/assets/images/` |

Workflow :

1. **Vérifier qu'il n'existe pas déjà** (grep dans tous les Flows pour l'usage attendu, lister `ds/assets/brand/`, `ds/assets/illustrations/`, `ds/assets/images/`)
2. **Récupérer le fichier** :
   - Si fourni par le user / designer : sauvegarder verbatim
   - Si à créer : SVG si vectoriel possible (préféré pour brand + illustrations), PNG/JPG sinon
3. **Nommer en kebab-case descriptif** : `empty-tasks.svg`, `logo-monochrome.svg`, `hero-landing.jpg`, pas `illustration1.svg`
4. **Sauvegarder dans le bon sous-dossier** :
   - `ds/assets/brand/` pour identité
   - `ds/assets/brand/splash/` spécifiquement pour splash screen
   - `ds/assets/illustrations/` pour scènes vectorielles
   - `ds/assets/images/` pour raster
5. **Référencer dans le markup** :
   - `<img src="ds/assets/brand/logo.svg" alt="Logo TaskMate">` dans une nav-bar par exemple
   - `<img src="ds/assets/illustrations/empty-tasks.svg" alt="">` dans un empty state (alt vide si décoratif)
   - `background-image: url('ds/assets/images/hero.jpg')` ou `<img>` selon le besoin
6. **Documenter dans `UI Kit.html`** : ajouter une cellule de galerie dans la section Brand / Illustrations / Images avec rendu de l'asset + nom de fichier en monospace
7. **Screenshot post-ajout** via chrome-devtools pour valider rendu (l'asset doit s'afficher correctement, pas un broken image icon)

### Ajouter une icône (pattern data-asset)

1. Vérifier qu'elle n'existe pas déjà (`ls ds/assets/icons/` + grep `data-asset="ds/assets/icons/<name>.svg"` dans les HTML)
2. Choisir le nom (kebab-case, **canonique** de la famille du kit — pour Lucide, vérifier sur lucide.dev avant de nommer : `ellipsis-vertical` pas `more-vertical`, `trash-2` pas `trash`, `triangle-alert` pas `alert-triangle`)
3. **Fetch le SVG depuis la source officielle** :
   - Lucide : `https://raw.githubusercontent.com/lucide-icons/lucide/main/icons/<name>.svg`
   - Phosphor : `https://raw.githubusercontent.com/phosphor-icons/core/main/raw/regular/<name>.svg`
   - Heroicons, Material : équivalents respectifs
   - **Ne jamais ré-écrire les paths à la main**
4. Sauvegarder verbatim dans `ds/assets/icons/<name>.svg` (asset individuel — fichier source de vérité)
5. **Ajouter aussi le SVG dans le sprite consolidé** `ds/assets/icons/icons.svg` (référence agrégée — pas inliné dans les HTML, juste pour la doc)
6. **Inliner l'icône dans le HTML qui en a besoin** : copier le contenu du fichier `ds/assets/icons/<name>.svg`, ajouter l'attribut `data-asset="ds/assets/icons/<name>.svg"` sur le `<svg>` racine, ajuster `width`/`height` au contexte d'usage. Exemple :
   ```html
   <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor"
        stroke-width="2" stroke-linecap="round" stroke-linejoin="round"
        data-asset="ds/assets/icons/<name>.svg">
     <path d="..."/>
   </svg>
   ```
7. **Documenter dans la section Iconographie de `UI Kit.html`** : nouvelle cellule de galerie avec le SVG inliné + son `data-asset` + nom de fichier sous l'icône
8. **Screenshot post-ajout** via chrome-devtools, vérifier que la nouvelle icône rend bien dans la galerie ET dans le Flow où elle est consommée

### Refaire le logo (et les 7 fichiers brand)

Le logo posé par `ui-kit-creator` est un placeholder simple (lettermark générique). Quand l'utilisateur veut itérer dessus ou repartir d'une exploration multi-direction, **invoquer le skill dédié `ui-kit-brand-explorer`** qui :

1. Lit `ds/tokens.css` (palette + typo) et le brief court depuis PRD/README/description
2. Propose 6 directions logo (lettermark, wordmark, combinaison, géométrique, métier, monogramme)
3. Génère un `Brand explorer.html` temporaire avec 6 × 4 variantes côte-à-côte
4. Screenshot via chrome-devtools, l'utilisateur choisit
5. Itère sur la direction choisie si demandé
6. Génère les 7 fichiers brand cohérents (`logo.svg`, `logo-mark.svg`, `logo-monochrome.svg`, `logo-dark.svg`, `app-icon.svg`, `splash.svg`, `favicon.svg`)
7. Met à jour la section Brand de `UI Kit.html`
8. Cleanup `Brand explorer.html`

**Quand ne PAS utiliser `ui-kit-brand-explorer`** :
- Modification mineure d'un seul fichier (ex: dark-mode du logo manquant) → recette "Ajouter un asset brand" suffit
- L'utilisateur fournit déjà un logo finalisé (SVG depuis Figma/Illustrator) → juste copier les 7 fichiers à la main

**Quand l'utiliser** :
- Le placeholder du creator ne convient pas
- Refonte complète de l'identité
- Exploration multi-directions avant de figer
- L'utilisateur n'est pas designer et a besoin de propositions visuelles

Voir `ui-kit-brand-explorer/SKILL.md` pour le détail complet du workflow.

### Localiser les fonts d'un kit existant (passer de @import CDN → fichiers locaux)

Cas typique : `tokens.css` commence par `@import url('https://fonts.googleapis.com/css2?...')`. Ça marche pour le mockup web mais casse côté Flutter (qui ne lit pas le woff2 servi par la CDN) et pour le offline.

1. **Identifier les fonts** utilisées : grep `font-family:` dans `tokens.css` et `components.css`, lister chaque famille distincte avec ses graisses.
2. **Créer le dossier** : `mkdir -p ds/assets/fonts/<font-name>/` pour chaque famille.
3. **Télécharger les deux formats** :
   ```bash
   # WOFF2 (web)
   curl -L -o ds/assets/fonts/<font>/<font>-variable.woff2 \
     "https://cdn.jsdelivr.net/fontsource/fonts/<font>:vf@latest/latin-wght-normal.woff2"
   # TTF (Flutter / iOS / Android natif) — --globoff pour les [brackets] des variable fonts
   curl -L --globoff -o ds/assets/fonts/<font>/<font>-variable.ttf \
     "https://raw.githubusercontent.com/google/fonts/main/ofl/<font>/<Font>[axes].ttf"
   ```
   Pour fonts statiques (non-variable) : un fichier par graisse dans chaque format.
   Alternative tout-en-un : `https://gwfh.mranftl.com/api/fonts/<font>?download=zip&formats=woff2,ttf`
4. **Remplacer le `@import`** dans `tokens.css` par des `@font-face` locaux :
   ```css
   @font-face {
     font-family: '<Font>';
     font-style: normal;
     font-weight: 400 700;
     font-display: swap;
     src: url('assets/fonts/<font>/<font>-variable.woff2') format('woff2-variations'),
          url('assets/fonts/<font>/<font>-variable.ttf')   format('truetype-variations');
   }
   ```
5. **Screenshot post-migration** via chrome-devtools — vérifier que la typo s'affiche correctement (pas de fallback system inattendu, espacements lettres préservés).
6. **Ajouter le snippet `pubspec.yaml`** dans `GUIDELINES.md` ou `README.md` pour le dev Flutter.

### Migrer un kit aux conventions sémantiques (data-os-chrome, data-uses, data-hint, data-nav-target, data-api-call)

À appliquer sur un kit antérieur aux conventions sémantiques. Symptômes typiques : status-bar/dynamic-island/home-indicator sans `data-os-chrome`, SVG signal/wifi/battery porteurs de `data-asset` (faux), patterns carousel/dialog/bottom-sheet sans `data-uses="ui:..."`, image-picker recodé en UI custom au lieu de `data-uses="native:image-picker"`, CTA navigants sans `data-nav-target`, pages data-driven (liste, détail, feed) sans `data-api-call`.

**Pourquoi c'est important** : sans ces conventions, un outil de code-gen génère du code qui mocke le chrome système (mort), recode des UI custom au lieu d'utiliser les widgets partagés du projet, et liste des assets fantômes dans le contrat émis. Bug invisible avant intégration.

**Workflow strict** :

1. **Inventaire `data-os-chrome` à poser** :
   ```bash
   # Conteneurs OS-chrome non marqués
   grep -rE 'class="(status-bar|dynamic-island|home-indicator|keyboard|gesture-bar|notch)"' flows/ "UI Kit.html" \
     | grep -v 'data-os-chrome'
   ```
   Pour chaque match : ajouter `data-os-chrome="<type>"` sur le conteneur. **Retirer `data-asset`** sur tous les SVG enfants (signal, wifi, battery, time).

2. **Inventaire `data-uses="ui:..."` à poser** — patterns à comportement non-évident :
   ```bash
   # Carrousels
   grep -rE 'class="[^"]*\b(carousel|swiper|slider-cards)\b' flows/ "UI Kit.html"
   # Bottom sheets
   grep -rE 'class="[^"]*\b(bottom-sheet|sheet-modal|action-sheet)\b' flows/ "UI Kit.html"
   # Dialogs
   grep -rE 'class="[^"]*\b(dialog|modal-dialog|alert)\b' flows/ "UI Kit.html"
   # Indicators / nav
   grep -rE 'class="[^"]*\b(dots|page-indicator|tab-bar|bottom-nav|app-bar|stepper|segmented)\b' flows/ "UI Kit.html"
   ```
   Pour chaque match, déterminer la valeur `ui:*` correspondante (cf. table dans Conventions sémantiques) et l'ajouter en attribut.

3. **Inventaire `data-uses="native:..."` à poser** — fonctions natives :
   ```bash
   # Boutons "Ajouter photo / prendre photo / scanner / partager / etc."
   grep -rE '(photo|caméra|camera|scanner|partager|share|biom|empreinte)' flows/ "UI Kit.html"
   ```
   Pour chaque CTA déclenchant une fonction native, ajouter `data-uses="native:image-picker|camera|share-sheet|biometric-auth|..."`.

4. **Inventaire `data-hint` à poser** — ambiguïtés métier :
   - Galeries de photos : "BDD app" vs "pellicule device" ? Marquer.
   - Listes avec swipe : "swipe-to-delete iOS" vs "long-press menu" ? Marquer.
   - Permissions : "demandée maintenant" vs "déjà accordée" ? Marquer.

5. **Inventaire `data-nav-target` à poser** — graphe de navigation :
   ```bash
   # CTA / rows / liens cliquables qui devraient naviguer mais qui ne le déclarent pas
   grep -rE 'class="(btn|task-row|nav-action|link|chip)' flows/ "UI Kit.html" \
     | grep -v 'data-nav-target'
   ```
   Pour chaque match, déterminer **vers quel écran** ce CTA navigue (depuis le PRD, le bon sens UX du flow, ou en demandant à l'utilisateur si ambigu) :
   - Bouton "Se connecter" sur `auth/login:default` → probablement `tasks/list:default`
   - Click sur `.task-row` dans `tasks/list` → probablement `tasks/detail:default`
   - Bouton "Nouvelle tâche" / FAB → probablement `tasks/list:edit-modal` (état modal de la même page) ou un autre écran de création
   - Bouton retour avec icône chevron-left → `data-nav-target="back" data-nav-back="true"`

   **Cross-check** : pour chaque `data-nav-target` posé, vérifier que la cible existe (cell `data-screen-label` correspondante dans le bon fichier flow).

6. **Inventaire `data-api-call` à poser** — endpoints consommés par chaque page :
   ```bash
   # Pages qui rendent une liste / détail / feed / form — candidates fortes pour data-api-call
   grep -lrE 'class="[^"]*\b(list|grid|feed|detail-card|form)\b' flows/
   ```
   Pour chaque page candidate, identifier le(s) endpoint(s) consommé(s) :
   - Page liste de recettes → `<main data-api-call="GET:/recipes">`
   - Page détail recette → `<main data-api-call="GET:/recipes/{recipe_id};GET:/recipes/{recipe_id}/comments">`
   - Bouton "Prendre en photo" qui upload → `<button data-api-call="POST:/recipes/scan" data-uses="native:camera">`
   - Form de modification → `<form data-api-call="PUT:/recipes/{recipe_id}">` (le path param vient du context route)
   - Bouton "Supprimer" → `<button data-api-call="DELETE:/recipes/{recipe_id}">`

   **Règles** :
   - Poser sur le **conteneur le plus haut** qui consomme l'endpoint (typiquement `<main>` ou la `.phone__screen`)
   - **Path params en `{name}`**, jamais avec valeur en dur (`{recipe_id}` pas `42`)
   - **Pas de query params** dans `data-api-call` (`?cuisine=italian` interdit — vit dans le state du cubit)
   - Multiples endpoints séparés par `;`
   - Une page entièrement statique (splash, onboarding, marketing pré-auth) reste sans `data-api-call`

   **Cross-check OpenAPI** : si le projet a un fichier OpenAPI (typiquement `docs/openapi.yaml` ou `backend/openapi.yaml`), vérifier que chaque `<METHOD>:/path` posé existe bien dedans. Sinon `unknowns` + signaler à l'utilisateur.

7. **Pour chaque modif**, suivre la phase 4 vérification (screenshot viewport-by-viewport). Le rendu visuel ne doit pas changer — c'est purement du tagging.

9. **Audit final** :
   ```bash
   # Aucun SVG du chrome système ne porte data-asset
   grep -E '<svg[^>]*data-asset[^>]*>' flows/ -A 5 | grep -B 2 -E '(class="status-bar"|<rect.*[0-9]+,[0-9]+|wifi|battery)' || echo "✓ aucun"
   # Tous les patterns UI évidents portent data-uses
   # Toutes les fonctions natives portent data-uses
   # data-api-call : format valide partout
   grep -rE 'data-api-call="[^"]*"' flows/ | grep -vE 'data-api-call="(GET|POST|PUT|PATCH|DELETE):/[^?"]+(;\s*(GET|POST|PUT|PATCH|DELETE):/[^?"]+)*"' && echo "✗ data-api-call mal formé (query string ou METHOD invalide)" || echo "✓ data-api-call format OK"
   ```

10. **Screenshot viewport-by-viewport** chaque page touchée + vérifier zéro régression.

**Note** : cette recette est purement additive. Elle n'enlève rien d'existant sauf les `data-asset` parasites sur le chrome système. Le rendu visuel reste identique avant/après.

### Migrer un widget redéfini localement vers le catalogue

Cas typique : un Flow contient `<style>.task-row { … }</style>` au lieu de consommer `.task-row` du DS.

1. Identifier toutes les occurrences locales du widget (grep `<style>` dans tous les Flows)
2. Comparer les versions — laquelle est la canonique ?
3. Ajouter la version canonique dans `ds/components.css`
4. Documenter dans la section Composants de `UI Kit.html`
5. Supprimer le `<style>` local dans chaque Flow
6. **Screenshot via chrome-devtools** chaque Flow modifié, vérifier qu'aucune régression visuelle n'apparaît
7. Si variantes divergentes : créer des modifiers (`.task-row--compact`) plutôt que dupliquer

### Consolider un kit non-DRY (audit + refactor)

À proposer quand on découvre un ou plusieurs des problèmes :
- `tokens.css` mélange tokens + classes widgets
- pas de `ds/components.css`
- icônes SVG inline **sans** `data-asset` (perte de traçabilité code-gen)
- assets directement sous `ds/icons/`, `ds/brand/`, `ds/illustrations/`, `ds/images/`, `ds/fonts/` (au lieu de `ds/assets/<famille>/`)
- sprite icônes en fichier externe référencé via `<use href="ds/assets/icons/icons.svg#…"/>` (cassé en file://)
- pages flow à la racine (`Flow 01 - Tasks.html`) au lieu de `flows/<flow>/<page>.html`
- doc UI Kit éclatée en plusieurs pages (Components.html, Icons.html)
- markup HTML apparent (`<pre><code>`) dans la doc
- **conventions sémantiques absentes** : `data-os-chrome`, `data-uses` (native:* / ui:*), `data-hint` jamais utilisés (la recette "Migrer un kit aux conventions sémantiques" doit être appliquée en parallèle)

**Workflow strict** :

1. **Inventaire** : lister chaque widget répété, chaque icône inline répétée, chaque page de doc séparée
2. **Vérifier visuellement l'état actuel** (screenshot via chrome-devtools de chaque page) — c'est la baseline anti-régression
3. **Proposer le plan** à l'utilisateur (impact, fichiers créés/modifiés/supprimés, ordre)
4. **Si nouvelles icônes nécessaires** : fetch depuis la source officielle (cf. recette icône) avant tout
5. Split tokens/components : créer `ds/components.css` avec toutes les classes widgets, strip `tokens.css` aux `:root` + reset
6. **Réorganiser** `ds/` : déplacer `ds/icons/`, `ds/brand/`, `ds/illustrations/`, `ds/images/`, `ds/fonts/` sous `ds/assets/`. Mettre à jour tous les chemins `<img src>` / `background-image: url()` dans les HTML.
7. Construire `ds/assets/icons/<name>.svg` (un fichier par icône, depuis source officielle) + `ds/assets/icons/icons.svg` (sprite source consolidé, **non inliné** dans les HTML)
8. **Convertir chaque icône inline** dans les HTML au pattern `data-asset` : pour chaque `<svg>...<path>` ad-hoc trouvé, identifier l'icône Lucide canonique correspondante, remplacer par le SVG inline complet du fichier source + ajouter `data-asset="ds/assets/icons/<name>.svg"`. Pour chaque `<svg><use href="ds/...#icon-X"/></svg>` (sprite externe cassé), remplacer aussi par le SVG inline + `data-asset`.
9. **Migrer les flows à la racine** (`Flow 01 - Tasks.html`, etc.) vers `flows/<flow>/<page>.html`. Mettre à jour les chemins relatifs des stylesheets (`href="../../ds/tokens.css"`) et des assets (`src="../../ds/assets/..."`).
10. **Fusionner `Components.html` + `Icons.html`** dans `UI Kit.html` (page unique). Supprimer les pages séparées. **Retirer tout `<pre><code>`** — la doc devient purement visuelle.
11. **Migrer une famille à la fois** (boutons, puis cards, puis list rows…) en consolidant
12. **À chaque étape majeure : screenshot via chrome-devtools** pour vérifier zéro régression visuelle
13. **Comparer aux screenshots baseline** (étape 2) avant de livrer
14. Livrer en plusieurs commits/étapes pour faciliter la review

### Modifier un token du DS

⚠️ **Haut impact**. Procéder ainsi :

1. Grep tous les consommateurs du token
2. Estimer l'impact visuel sur chaque fichier
3. Proposer le changement à l'utilisateur avec liste impactés
4. Si validé : modifier le token
5. Ouvrir chaque fichier impacté, valider visuellement
6. Ajuster les fichiers où le changement casse quelque chose

### Refactorer un fichier devenu trop gros

1. Proposer un découpage par flow logique
2. Identifier les styles communs à extraire dans un `_shared.css`
3. Lister les fichiers résultants + leurs contenus
4. Valider avec l'utilisateur
5. Exécuter le split

### Audit de cohérence

Checklist agnostique du produit :

- [ ] Tous les fichiers importent le DS (`tokens.css` + `components.css`)
- [ ] Aucun hex hardcodé (en dehors de palettes fixes connues : bezel iOS, dynamic island)
- [ ] Tailles de texte dans l'échelle DS
- [ ] Boutons primaires identiques partout (height, radius, color) — **même classe** depuis le DS
- [ ] **Aucun `<style>` local de widget** dans les Flows (seuls canvas/frame autorisés)
- [ ] **SVG d'icônes inlinés en plein avec `data-asset`** — chaque `<svg>` racine porte `data-asset="ds/assets/icons/<name>.svg"`
- [ ] **Aucune référence `<use href="ds/`** (sprite externe banni en `file://`)
- [ ] **Aucun `<use href="#icon-`** (le sprite inline est obsolète, on inline les SVG en plein)
- [ ] Toutes les icônes utilisées (grep `data-asset="ds/assets/icons/`) existent bien comme fichier source dans `ds/assets/icons/`
- [ ] **`xmllint --noout`** sur tout SVG modifié/ajouté → 0 erreur (sinon `<img src=...svg>` casse silencieusement)
- [ ] **Caractères spéciaux échappés** dans les SVG : `&` → `&amp;`, `<` → `&lt;` dans `aria-label`, `<title>`, `<desc>` et attributs
- [ ] **Aucun screenshot `fullPage:true`** utilisé pour la validation — toujours viewport-by-viewport
- [ ] **Toutes les sections couvertes** au screenshot, pas un échantillon (un bug en section Brand a déjà été raté en sautant des sections)
- [ ] **Inline styles** sur des patterns visuels répétés ≥2× → factorisés en classe `.foo` dans `components.css`
- [ ] Icônes Lucide (ou autre famille) **canoniques** — paths fetched depuis la source officielle, pas approximés
- [ ] Tous les widgets utilisés sont documentés dans la section Composants de `UI Kit.html`
- [ ] **Doc UI Kit en page unique** (pas de Components.html/Icons.html séparés)
- [ ] **Pas de `<pre><code>`** dans la doc UI Kit (rendu visuel uniquement)
- [ ] Tout asset référencé par `<img src="ds/...">` ou `background-image: url('ds/...')` existe bien dans le bon sous-dossier (`brand/`, `illustrations/`, `images/`)
- [ ] Aucun asset inline ou en-dehors de `ds/` (pas de logo en data-URL dans un Flow, pas d'image dans le dossier racine)
- [ ] Sections Brand / Illustrations / Images de `UI Kit.html` à jour avec les nouveaux assets ajoutés
- [ ] Stroke/fill cohérent entre toutes les icônes (même famille, même stroke-width)
- [ ] **Pages flow sous `flows/NN-<flow_id>/<page_id>.html`** (aucun `Flow NN.html` à la racine ; préfixe `NN-` obligatoire sur le dossier flow, optionnel sur la page)
- [ ] **Cellules d'état** avec `data-screen-label="NN <Page> <state>"` cohérent (le dernier mot doit être un state_id valide : default, loading, empty, error, edit, success…)
- [ ] **Tous les assets sous `ds/assets/`** (aucun `ds/icons/`, `ds/brand/`, etc. au niveau de `ds/`)
- [ ] **`data-os-chrome`** posé sur tous les conteneurs OS-chrome présents (status-bar, dynamic-island, home-indicator, keyboard, gesture-bar). SVG enfants de ces conteneurs **ne portent jamais** `data-asset`.
- [ ] **`data-uses="ui:..."`** sur les patterns à comportement non-évident (carousel, dialog, bottom-sheet, etc.). Sans ça, l'agent codeur recode en custom au lieu d'utiliser le widget partagé / framework primitive.
- [ ] **`data-uses="native:..."`** sur les fonctions natives sollicitées (image-picker, camera, biometric, etc.). L'agent codeur priorise widget partagé > package natif si le widget existe.
- [ ] **`data-hint`** posé quand la sémantique métier n'est pas évidente (galerie BDD vs native, etc.).
- [ ] **`data-nav-target`** posé sur tous les éléments qui naviguent + cibles existantes dans l'inventaire (vérifier les liens cassés via grep + cross-check `data-screen-label`).
- [ ] Zéro emoji dans l'UI (sauf exceptions documentées)
- [ ] Zéro font icon
- [ ] Tous les écrans ont des métadonnées (id, description, tag)
- [ ] Copy cohérent (formel/familier, longueur)
- [ ] **Aucun terme technique dans la copy user-facing** : pas de nom de modèle IA (Sonnet, GPT, Claude…), pas de provider (Groq, Anthropic, OpenAI, RevenueCat…), pas d'acronyme tech (OCR, ASR, LLM, JWT, API, SDK…). Grep `Sonnet|Opus|Haiku|GPT|Groq|Anthropic|OpenAI|RevenueCat|OCR|ASR|LLM|JWT|SDK` dans les fichiers `flows/**/*.html` et `UI Kit.html` → 0 match dans du texte visible. Termes commerciaux d'OS/store autorisés (App Store, Google Play, iCloud, Apple Pay, Sign in with Apple).
- [ ] Pas de texte < taille minimum définie par le DS
- [ ] Contrastes conformes niveau d'accessibilité cible
- [ ] Hit targets ≥ 44×44 sur mobile
- [ ] **Screenshot via chrome-devtools** de chaque page — toutes les icônes rendent, aucun élément cassé
- [ ] **`data-nav-target` cohérent** : tous les CTA navigants en portent, toutes les cibles existent dans l'inventaire des écrans (cross-check via grep `data-screen-label`).

## 🚫 Interdictions universelles

1. **Modifier le DS silencieusement** sans évaluer l'impact
2. **Inventer une couleur / un token** qui n'existe pas
3. **Hardcoder des valeurs** au lieu d'utiliser les tokens
4. **Remplacer un écran approuvé** sans garder l'original (faire variante)
5. **Ajouter un emoji** dans l'UI (exceptions rares, demander avant)
5b. **Faire fuiter la stack technique dans les copy user-facing** — aucun nom de modèle IA (Sonnet, Opus, Haiku, GPT-4, Claude 4.6…), aucun nom de provider (OpenAI, Anthropic, Groq, RevenueCat, Supabase…), aucun acronyme tech (OCR, ASR, LLM, API, SDK, JWT…), aucun nom interne d'endpoint, queue, service. L'utilisateur ne doit voir que des termes métier : "scans photo" pas "scans IA (Sonnet 4.6)" — "synchro" pas "WebSocket sync" — "compte" pas "JWT session". Les noms commerciaux d'OS / store sont OK (App Store, Google Play, iCloud, Apple Pay, Sign in with Apple) — c'est ce que l'utilisateur connaît. À chaque copy ajoutée ou modifiée : grep ce qui ressemble à un identifiant technique et neutralise. Les attributs `data-uses="native:..."` / `data-uses="ui:..."` (eux machine-readable) restent inchangés — l'interdiction porte uniquement sur le **texte visible**.
6. **Utiliser un font icon** — SVG inline uniquement
7. **Inliner un SVG d'icône sans `data-asset`** — perte de traçabilité pour les outils de code-gen. Toujours `data-asset="ds/assets/icons/<name>.svg"` qui pointe vers le fichier source.
8. **Référencer un sprite externe** (`<use href="ds/assets/icons/icons.svg#…"/>`) — cassé en file://, ce pattern est banni
9. **Mettre des assets directement sous `ds/`** (au lieu de `ds/assets/<famille>/`) — tous les assets vivent sous `ds/assets/`
10. **Mettre un fichier flow à la racine** (`Flow NN.html`) — toujours sous `flows/<flow>/<page>.html`
9. **Approximer les paths SVG** — toujours fetch depuis la source officielle (Lucide GitHub raw, etc.)
10. **Définir un widget en local** dans un Flow au lieu de l'ajouter au catalogue partagé
11. **Créer des pages de doc séparées** (Components.html, Icons.html) — tout doit être dans `UI Kit.html`
12. **Ajouter du `<pre><code>` apparent** dans la doc UI Kit — rendu visuel uniquement
13. **Dessiner une illustration complexe** en SVG inline (placeholder + asset externe)
14. **Copier-coller du CSS** qui existe déjà dans le DS
15. **Dupliquer un widget** sous un nouveau nom au lieu d'ajouter un modifier (`.card.card--interactive` > `.card-2`)
16. **Mélanger les conventions de nommage** (tout BEM ou tout utility, pas les deux)
17. **Livrer sans screenshot via chrome-devtools** — la validation visuelle est obligatoire
18. **Ajouter `.btn--lg` (52px) ou `.btn--xl` (60px) à un kit mobile** — la hauteur 44px par défaut est le touch-target Apple HIG / Material. Plus grand = écrasement de la hiérarchie visuelle mobile. Pour différencier un CTA important : couleur (primary), position (sticky bottom) ou `btn--block` (pleine largeur). Garder uniquement `.btn--sm` (36px, pour boutons inline dans chips/badges/tags) + taille par défaut. Si on trouve `.btn--lg` / `.btn--xl` dans `components.css` d'un kit mobile, **les supprimer** (recette modif token, vérifier consommateurs avant).
19. **Garder un `@import` Google Fonts CDN** dans un kit livré — casse Flutter (qui ne lit pas le woff2), casse offline. Toujours **localiser les fonts** (recette dédiée) : télécharger woff2 + ttf dans `ds/assets/fonts/<font>/` + remplacer l'`@import` par des `@font-face` locaux.
20. **`data-asset` sur des SVG du chrome système** (signal, wifi, battery, time). Ils ne sont **pas des assets app**. Le parent doit porter `data-os-chrome="status-bar"`, les enfants sont skipés par héritage. Si on trouve `data-asset` dessus, le retirer (et confirmer que le parent porte bien `data-os-chrome`).
21. **Élément OS-chrome non marqué** (status-bar, dynamic-island, home-indicator, keyboard, gesture-bar sans `data-os-chrome`) — l'agent codeur va générer du code mort qui conflit avec ce que l'OS dessine.
22. **Carousel / dialog / bottom-sheet / dot-indicator sans `data-uses="ui:..."`** — l'agent codeur recode une UI custom au lieu d'utiliser le widget partagé du projet ou la primitive framework adéquate.
23. **Fonction native (image-picker, camera, biometric, etc.) sans `data-uses="native:..."`** — l'agent codeur recode une UI custom au lieu d'invoquer le widget partagé / API native du framework.
24. **Recoder en custom une fonction native sans avoir d'abord cherché un widget partagé** dans `ui_components/` du projet codeur. L'ordre est strict : widget partagé > package natif > UI custom (cf. UI_KIT_CONTRACT du contrat upstream).
25. **CTA / row / lien navigant entre écrans sans `data-nav-target`** — `ui-kit-prototype` ne peut pas créer de prototype interactif fonctionnel, et l'agent codeur devine la route au lieu de la générer depuis une déclaration explicite. À poser systématiquement.
26. **`data-nav-target` cassé** (cible un écran qui n'existe pas) — au prototype le clic devient no-op et un toast s'affiche. À fixer en même temps que la modif (vérifier les cibles via cross-check `data-screen-label`).
27. **Modifier le `Prototype.html`** à la main — c'est un fichier généré par le skill `ui-kit-prototype`. Toute modif des Flows doit être suivie d'une re-génération du Prototype.

## 🎯 Gérer les cas ambigus

### Le token n'existe pas dans le DS
→ Signaler + proposer 2 options : (a) étendre le DS (impact systémique), (b) utiliser le token le plus proche

### Le widget n'existe pas dans `components.css`
→ Ne **jamais** l'écrire en local dans le Flow. Soit (a) l'ajouter au catalogue (recette dédiée), soit (b) si l'utilisateur veut juste un sketch jetable, demander explicitement la permission de mettre du `<style>` scoped, et marquer le Flow comme non-canonique.

### L'icône n'existe pas dans `ds/assets/icons/`
→ Fetch depuis source officielle, sauvegarder en `ds/assets/icons/<name>.svg`, **puis** inliner dans le HTML avec `data-asset` (cf. recette "Ajouter une icône"). Pas de SVG inline ad-hoc sans `data-asset`.

### Un composant similaire existe mais ne fait pas exactement ce qu'on veut
→ Étendre via modifier/variante (`.card--compact`, `.btn--icon-only`) plutôt que dupliquer. Si trop éloigné, créer nouveau composant + documenter dans `Components.html`.

### Le kit n'a pas de `components.css` ni de sprite
→ Signaler que le kit est en mode "DS partiel". Proposer la recette "Consolider un kit non-DRY" en parallèle de la modif demandée, ou en pré-requis si la modif touche plusieurs Flows.

### La copy est incohérente dans le projet existant
→ Signaler les incohérences, suivre le ton majoritaire, proposer un audit séparé si important.

### Le projet mélange plusieurs patterns/conventions
→ Identifier le plus récent/dominant, l'utiliser, suggérer un refactor si écart important.

### Demande qui casse les règles du DS
→ Pousser-back. Proposer alternative dans le cadre du DS. Si vraiment bloqué, demander validation explicite avant de casser.

## 📤 Format de livraison

```markdown
## Modif : [résumé 1 ligne]

**Fichier(s) modifié(s)** : `[chemins]`
**Sections/cellules touchées** : `[identifiants]`
**Composants nouveaux** : [liste / aucun]
**Impact DS** : [aucun / extension — préciser]

### Diff
[diff concis OU code complet du fichier si gros refactor]

### Vérifications
- [x] Tokens DS uniquement
- [x] Composants réutilisés
- [x] Naming conventions respectées
- [x] Console clean
- [x] Responsive OK
- [x] Cohérence avec écrans voisins

### Notes / alertes
- [rien OU points d'attention pour la review]
```

## 🔁 Quand bloqué

1. **Rouvrir le DS** — le token existe probablement
2. **Rouvrir `components.css` + section Composants de `UI Kit.html`** — le widget existe peut-être sous un modifier
3. **Lister `ds/assets/icons/` + section Iconographie de `UI Kit.html`** — l'icône est peut-être déjà disponible sous un autre nom (canonique de la famille)
4. **Screenshot chrome-devtools** d'un Flow qui marche pour comparer
5. **Grep les autres fichiers** pour pattern similaire
6. **Demander clarification** plutôt qu'inventer
7. **Proposer 2-3 options** avec trade-offs plutôt qu'imposer un choix

## 🐛 Symptômes courants → diagnostic

| Symptôme | Diagnostic probable |
|---|---|
| Toutes les icônes sont vides / "grabouillis" / un seul SVG visible | Sprite référencé en externe via `<use href="ds/assets/icons/icons.svg#…"/>` — cassé en file://. **Convertir en SVG inline complet avec `data-asset`** (cf. recette consolider). |
| Icône inline rendue mais pas matchable par un outil de code-gen | `<svg>` sans `data-asset`. Ajouter l'attribut pointant vers `ds/assets/icons/<name>.svg`. |
| Image cassée (broken image icon) dans un Flow | Mauvais chemin relatif. Depuis `flows/<flow>/<page>.html`, le préfixe est `../../ds/assets/...`. |
| Image broken icon dans la galerie Brand de l'UI Kit | SVG XML invalide (souvent `&` brut dans `aria-label` ou `<title>`). Lancer `xmllint --noout file.svg` et corriger les erreurs. Utiliser `&amp;`, `&lt;`, `&gt;` pour les caractères spéciaux. |
| Logo light s'affiche sur header dark | `<img>` ne propage pas `currentColor` aux SVG externes. Solution dual-img + CSS swap : `<img class="logo-light">` + `<img class="logo-dark">` avec `[data-theme="dark"] .logo-light{display:none} [data-theme="dark"] .logo-dark{display:block}`. |
| Texte d'un logo SVG invisible côté Flutter (rendu correct dans le browser) | `<text>` avec `font-family="A, B, serif"` (liste CSS) ou `font-variation-settings="…"` — non parsable par `flutter_svg`. Corriger : une seule valeur `font-family="X"`, remplacer `font-variation-settings` par `font-weight`/`font-style`. Vérifier que `X` est dans `ds/assets/fonts/<x>/` ET dans `pubspec.yaml > flutter > fonts:`. Pour un logo qui voyage (App Store, marketing), produire en plus une variante text-to-outline (`<text>` → paths). |
| Bug raté lors d'un screenshot validation | Probablement un `fullPage:true` (PNG trop grand pour le modèle) ou couverture par échantillon (début/milieu/fin) qui a sauté la section bugée. Refaire viewport-by-viewport, **toutes** sections. |
| "Ça a l'air bien" sur un Flow 3-phones côte-à-côte mais bugs vus à taille humaine | **Piège du zoom** : chaque phone n'occupe que ~250 px dans le PNG, les détails fins (texte qui déborde, mot dupliqué, mauvais centrage) sont invisibles à cette échelle. **Cropper la grille en colonnes** via PIL : 1 phone = 1 PNG ~1100 px → lire chaque crop séparément, scanner activement (centrage / overflow / z-index / auto-scroll / vars CSS undefined). |
| Texte de bouton qui déborde sur mobile (phone 360 px) | `.btn--block` + icône + label long, `white-space: nowrap` clippe à droite. Raccourcir label, retirer icône, ou ajouter `white-space: normal; line-height: 1.2`. |
| Élément (progress, illustration) collé à gauche au lieu d'être centré | Parent `<div>` sans `display: flex; align-items: center` ou `text-align: center`. Wrapper dans un container avec centrage explicite. |
| Container qui scroll seul au chargement (header haut disparaît) | `overflow-y: auto` sur tout l'écran + focus auto sur un élément bas → browser scroll. Layout split : header `flex-shrink: 0` + body `flex: 1; overflow-y: auto`. |
| Doublon visuel : un mot apparaît à la fois sous l'overlay ET dans la bulle | Élément avec `z-index` plus haut que l'overlay qui le traverse. Vérifier la hiérarchie z-index avant d'élever un élément du contenu. |
| Padding/margin qui semble = 0 sans cause apparente | `var(--space-NN)` undefined dans `tokens.css` → propriété invalide silencieusement. Diff usages composants vs tokens : `diff <(grep -oE 'var\(--space-[0-9]+\)' ds/components.css \| sort -u) <(grep -oE '^\s*--space-[0-9]+' ds/tokens.css \| tr -d ' :' \| awk -F'--' '{print "var(--"$2")"}' \| sort -u)`. |
| Une seule icône cassée | Mauvais id de symbol (vérifier `#icon-NAME` vs `<symbol id="icon-NAME">`) ou paths invalides |
| Icônes visibles mais "moches" / pas Lucide | Paths approximés à la main au lieu de fetched. Refetch depuis lucide-icons GitHub. |
| Layout de Flow éclaté visuellement | `<style>` local supprimé sans migrer le widget vers `components.css`. Vérifier que toutes les classes utilisées existent. |
| Couleur inattendue d'une icône | `currentColor` hérité de l'ancestor — wrapper avec `.icon-primary` etc., ou vérifier le contexte CSS |
| Sheet ne s'affiche pas en haut | Pas de `position:relative` sur le parent, ou z-index manquant |
