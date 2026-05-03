---
name: ui-kit-creator
description: Crée un nouveau design system + UI kit HTML complet depuis zéro pour n'importe quel produit (mobile, web, desktop). Génère tokens DS, composants, patterns d'écrans et documentation visuelle. À invoquer quand l'utilisateur commence un nouveau projet design sans base existante, ou veut formaliser un DS pour un produit en cours.
---

# UI Kit Creator — Nouveau projet de zéro

Crée un design system + kit HTML complet pour un nouveau projet, depuis zéro. Output : package HTML statique prêt à servir de référence hi-fi et de handoff dev.

## 🎯 Quand utiliser ce skill

- Nouveau produit, aucune base visuelle
- Produit existant sans DS formalisé
- Rebranding complet
- Proof of concept visuel avant dev

## 📋 Inputs requis (questions à poser si absents)

1. **Nom du produit** + description en 1 phrase
2. **Plateforme cible** : iOS / Android / Web / Desktop / multi
3. **Domaine** : fintech, santé, social, productivité, e-commerce…
4. **Public** : grand public / pro / enfants / seniors…
5. **Ton visuel souhaité** : minimaliste, dense, ludique, institutionnel, éditorial, techy
6. **Contraintes de marque** : logo existant ? palette imposée ? typo imposée ?
7. **Mode sombre** : requis / optionnel / non
8. **Accessibilité** : niveau WCAG cible (AA par défaut)
9. **Langues supportées** : pour dimensionner les labels (FR/EN/AR/ZH/...)
10. **Écrans prioritaires** : lister 5-10 écrans clés à mocker en premier

Si infos manquantes : poser les questions. Ne pas inventer le ton de marque.

## 🏗️ Structure du package à générer

```
/
├── ds/
│   ├── tokens.css                ← Variables CSS (couleurs, spacing, type, radius, shadows)
│   ├── components.css            ← Classes des widgets partagés (.btn, .card, .dialog, .tile…)
│   │
│   └── assets/                   ← TOUS LES ASSETS, regroupés par famille
│       │
│       ├── icons/                ← ICÔNES UI (vectorielles, monochrome, ≤24×24)
│       │   ├── icons.svg         ← Sprite consolidé source (référence designer + dev)
│       │   ├── plus.svg          ← Un fichier par icône (assets pour designer / dev mobile)
│       │   ├── check.svg
│       │   └── …
│       │
│       ├── brand/                ← IDENTITÉ DE MARQUE
│       │   ├── logo.svg          ← logo principal (full, avec wordmark si applicable)
│       │   ├── logo-mark.svg     ← symbole seul, sans texte
│       │   ├── logo-monochrome.svg
│       │   ├── logo-dark.svg     ← inversé pour fonds sombres
│       │   ├── app-icon.svg      ← icône d'app (source 1024×1024)
│       │   ├── favicon.svg
│       │   └── splash/           ← SPLASH SCREEN (mobile)
│       │       ├── splash.svg    ← composition vectorielle source
│       │       ├── splash@2x.png ← exports raster aux résolutions device si fournis
│       │       └── splash@3x.png
│       │
│       ├── illustrations/        ← SCÈNES VECTORIELLES (SVG, multi-couleur, 80-400px)
│       │   ├── empty-tasks.svg
│       │   ├── onboarding-1.svg
│       │   ├── error-404.svg
│       │   ├── decoration-blob.svg
│       │   └── …
│       │
│       ├── images/               ← MATIÈRES RASTER (PNG / JPG / WebP)
│       │   ├── hero.jpg
│       │   ├── placeholder-avatar.png
│       │   └── pattern-bg@2x.png
│       │
│       └── fonts/                ← .ttf / .woff2 + @font-face dans tokens.css
│           └── …
│
├── UI Kit.html                   ← PAGE UNIQUE : tokens + iconographie + brand + illustrations
│                                    + images + composants (visuel only)
│
├── flows/                        ← FLOWS UTILISATEUR
│   ├── 01-tasks/                 ← Un dossier par flow (préfixe numérique pour ordre)
│   │   ├── list.html             ← Une page HTML par écran/page du flow.
│   │   ├── detail.html              Chaque page contient les cellules pour ses
│   │   └── settings.html            différents états (default, loading, empty, error…)
│   ├── 02-onboarding/
│   │   ├── welcome.html
│   │   ├── permissions.html
│   │   └── …
│   └── …
│
├── GUIDELINES.md
├── README.md
└── CLAUDE_SKILLS.md              ← Prompts pour reprendre plus tard
```

**Convention `flows/<flow>/<page>.html`** (machine-readable pour outils de code-gen) :

| Niveau | Format | Exemple | ID dérivé | Règle de dérivation |
|---|---|---|---|---|
| Dossier flow | `NN-<id>` | `01-tasks` | `flow_id = "tasks"` | strip `/^\d+-/` du dossier |
| Fichier page | `<id>.html` (sans préfixe) **ou** `NN-<id>.html` (si ordre voulu) | `list.html` ou `01-welcome.html` | `page_id = "list"` ou `"welcome"` | strip `/^\d+-/` puis strip `.html` |
| Cellule (état) | attribut `data-screen-label="NN <Page> <state>"` | `data-screen-label="01 List default"` | `state_id = "default"` | dernier mot du label, en kebab-case lower |
| Identifiant global | `<flow_id>/<page_id>` (+ optionnellement `:<state_id>`) | `tasks/list` ou `tasks/list:loading` | screen unique stable | composé des trois ci-dessus |

**Règles** :
- **Préfixe `NN-` obligatoire sur les dossiers flow** — l'ordre matérialise la séquence/priorité produit, et permet à Mirror de gérer les dépendances entre flows (M2+ avec `flow.depends_on`)
- **Préfixe `NN-` optionnel sur les pages** — utiliser uniquement si l'ordre des pages a un sens (onboarding séquentiel : `01-welcome.html`, `02-permissions.html`, `03-ready.html`). Pour des flows à navigation libre (tasks : list ↔ detail ↔ settings), pas de préfixe.
- **Une page = un écran/route** côté code mobile (1 widget Flutter / 1 View SwiftUI / 1 composable Compose). Cell d'état = un rendu visuel de cet écran à un moment donné.
- **Cellules d'état dans la page** : chaque cellule du canvas représente un état visuel de la page (default, loading, empty, error, edit-modal, success…). Format `data-screen-label="NN <Page> <state>"` où `NN ` est l'ordre dans le canvas, `<Page>` redondant avec le filename (lisibilité humaine), `<state>` le dernier mot = `state_id` machine-readable. Cohérent avec ingest-mockup v2.0 / Playwright per-state rendering.

**Principes durs** :

1. **Une seule page de doc** : `UI Kit.html` regroupe **tout** (tokens, composants, icônes, brand, illustrations, images). Pas de pages séparées Components.html/Icons.html — l'utilisateur veut une référence unique navigable.

2. **Pas de markup HTML apparent dans la page** : aucun bloc `<pre><code>` qui affiche le code des composants. Le UI Kit est une **doc visuelle**, pas une doc technique. Le dev mobile/Flutter/iOS/etc. n'a rien à faire d'un snippet HTML — il regarde le rendu et code dans son framework.

3. **Icônes inlinées en plein SVG dans le markup, marquées par `data-asset`** :
   ```html
   <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor"
        stroke-width="2" stroke-linecap="round" stroke-linejoin="round"
        data-asset="ds/assets/icons/plus.svg">
     <path d="M5 12h14"/>
     <path d="M12 5v14"/>
   </svg>
   ```
   Pas de sprite + `<use href>`. Chaque usage embarque tout le SVG (pour rendu autonome en `file://` sans bug Chrome) ET porte un attribut `data-asset` qui pointe vers le fichier source dans `ds/assets/icons/`. Cet attribut est la **clé de matching** pour les outils de code-gen (Mirror, etc.) : ils peuvent grep `data-asset` dans le HTML pour savoir quel asset importer dans le projet mobile.

4. **Quatre familles d'assets distinctes** sous `ds/assets/`, jamais mélangées :
   - `icons/` — icônes UI : vectorielles, monochrome (currentColor), ≤24×24, réutilisées partout, **inlinées en plein SVG avec `data-asset`**
   - `brand/` — identité de marque : logos (variantes), app-icon, favicon, splash screen
   - `illustrations/` — scènes vectorielles : empty states, onboarding, decorative blobs (SVG, 80-400px, multi-couleur)
   - `images/` — matières raster : photos, hero, placeholders, textures (PNG/JPG/WebP)
   - `fonts/` — typo custom (.woff2)

5. **Référencement dans le markup** :
   - Icons (inlinées) : `<svg ... data-asset="ds/assets/icons/NAME.svg">…paths…</svg>`
   - Brand / illustrations / images : `<img src="ds/assets/brand/logo.svg" alt="…">` ou `background-image: url('ds/assets/images/hero.jpg')` — le chemin dans `src`/`url()` joue le même rôle de matching que `data-asset` pour les icônes

6. `ds/` contient **toutes** les briques réutilisables. Les Flow ne définissent rien de visuel propre — ils composent à partir du DS et inlinent les SVG d'icônes avec leur `data-asset`.

## 🎨 Design system — construction

### 1. Palette couleurs

Définir par catégorie, avec nommage sémantique :

```css
:root {
  /* Brand */
  --primary:    /* hero color */
  --secondary:  /* complément ou version douce */
  --accent:     /* point chaud, CTAs secondaires */

  /* Semantic */
  --success:    /* vert pas trop vif */
  --warning:    /* jaune/orange */
  --error:      /* rouge attention */
  --info:       /* bleu info */

  /* Surfaces */
  --bg:         /* fond page */
  --surface:    /* cards, sheets */
  --overlay:    /* modal barrier */

  /* Neutrals (échelle 50-900) */
  --neutral-50:  --neutral-100: ... --neutral-900:

  /* Text */
  --text:       /* ~0xFF2E2E2E — pas pur noir */
  --text-muted:
  --text-on-primary:
}
```

**Règles** :
- Contraste primary sur surface ≥ 4.5:1
- Palette ≤ 12 couleurs nommées (pas 50 variantes)
- Si dark mode : dupliquer avec `[data-theme="dark"]`

### 2. Spacing — échelle 4/8

```css
--space-4: 4px; --space-8: 8px; --space-12: 12px; --space-16: 16px;
--space-20: 20px; --space-24: 24px; --space-32: 32px; --space-48: 48px;
```

Utiliser exclusivement cette échelle.

### 3. Radii

```css
--radius-xs: 4px;  /* chips, tags */
--radius-sm: 8px;  /* inputs, petits boutons */
--radius-md: 12px; /* cards, boutons primaires */
--radius-lg: 16px; /* sheets, modals */
--radius-pill: 999px;
```

### 4. Typographie

Choisir **1 famille max** (2 si display + body justifié). Privilégier :
- **Sans** : Inter, Geist, General Sans, Söhne, Manrope
- **Éditorial** : Fraunces (avec modération), Instrument Serif
- **Mono** : JetBrains Mono, Geist Mono

**Éviter les clichés IA** : Inter partout sans raison, Roboto, Arial, system-ui par défaut.

Échelle (à ajuster selon densité souhaitée) :
```css
--fs-xs: 12px;   --fs-sm: 14px;   --fs-base: 16px;
--fs-md: 18px;   --fs-lg: 20px;   --fs-xl: 24px;
--fs-2xl: 32px;  --fs-3xl: 48px;
```

Poids : 400/500/600/700 suffisent (rarement 300/800).

### 5. Élévation

```css
--shadow-sm: 0 1px 2px rgba(0,0,0,.05);
--shadow-md: 0 2px 8px rgba(0,0,0,.08);
--shadow-lg: 0 8px 24px rgba(0,0,0,.12);
--shadow-xl: 0 24px 48px rgba(0,0,0,.16);
```

Adapter la couleur d'ombre au ton du produit (warm/cool).

### 6. Motion (pour handoff dev, pas pour le kit HTML)

```
--duration-fast: 120ms
--duration-base: 200ms
--duration-slow: 320ms
--ease-out: cubic-bezier(0.2, 0.8, 0.2, 1)
--ease-in-out: cubic-bezier(0.4, 0, 0.2, 1)
```

## 🧩 Catalogue de composants partagés — `ds/components.css` + `Components.html`

Tout widget réutilisé sur 2+ écrans **doit** vivre dans le DS. C'est le cœur du livrable, pas un bonus.

### Principe

- **`ds/components.css`** : source CSS de tous les widgets (`.btn`, `.btn-primary`, `.card`, `.task-row`, `.dialog`, `.sheet`, `.tile`, `.list-row`, `.banner`, `.toast`, `.empty-state`, etc.)
- **`Components.html`** : page-catalogue qui rend chaque widget dans toutes ses variantes/états + affiche le markup HTML à copier-coller. C'est la doc vivante.
- **Les fichiers Flow** importent `tokens.css` + `components.css` et n'écrivent **aucun style de widget** en local. Si un Flow a besoin d'un visuel non couvert, on l'ajoute d'abord au catalogue, puis on le consomme.

### Liste exhaustive des widgets à fournir

Pour chaque produit, livrer **au minimum** chaque famille ci-dessous (avec toutes les variantes pertinentes, états et tailles). Ne pas se contenter d'un seul exemple par famille.

#### Atomes
- **Boutons** : primary, secondary, ghost, destructive, icon-only, FAB, link-button (+ états hover/active/disabled/loading + tailles sm/md/lg)
- **Inputs** : text, email, password, search, textarea, number, date (+ états focus/error/filled/disabled + variante avec icône)
- **Selects / Dropdowns** : single, multi, searchable
- **Checkboxes / Radios / Toggles / Switches**
- **Badges / Tags / Chips / Meta-pills** (couleur sémantique, dot, removable)
- **Avatars** : sizes (xs/sm/md/lg/xl), fallback initial, image, status dot, group
- **Dividers** : full, inset, with-label
- **Loaders** : spinner, dots, progress bar, skeleton (block + text-line + circle)
- **Tooltip / Popover trigger**

#### Molécules
- **Cards** : plate, elevated, interactive (clickable), with-header, with-media, with-footer-actions
- **List rows / Tiles** : simple, leading-icon, leading-avatar, with-trailing-action, two-lines, three-lines, swipeable (mobile)
- **Section-card** (groupes de field-rows façon iOS settings) — pattern clé sur mobile
- **Field-row** : label + value, label + input, with chevron
- **Alerts** : info, success, warning, error (inline + dismissible)
- **Banners** : full-width announcement, with action
- **Toasts / Snackbars** : neutral, success, error, with action
- **Tabs** : underline, pill, segmented control (iOS)
- **Breadcrumbs**
- **Pagination**
- **Search bar** (avec clear button + suggestions)
- **Form-groups complets** : label + input + helper + error

#### Organismes
- **App bar / Top nav** : title, with back, with actions, large-title (iOS)
- **Bottom nav** (mobile) : 3-5 items, avec/sans label
- **Side nav** (desktop) : sections, collapse, active state
- **Tab bar** secondary
- **Dialog / Modal** : alert dialog, confirm dialog, full-screen modal, form dialog (titre + body + actions)
- **Sheet** : bottom sheet (mobile), drawer (desktop), action sheet (iOS)
- **Popover / Menu contextuel**
- **Empty states** : no-data, no-results, first-time
- **Error states** : 404, network, generic-error (avec retry)
- **Data tables** (web) : header, sortable, row-actions, sticky-header
- **Calendar / Date picker** (si utile au produit)

### Structure de `Components.html`

```
[Header : Composants — vivant, copiable]
[Sidebar / TOC : ancres vers chaque famille]

[Section : Boutons]
  → variantes côte-à-côte (preview)
  → bloc <pre><code> avec markup HTML à copier
  → liste des classes/modifiers utilisables

[Section : Cards]
  → idem
[…]
```

Chaque widget doit pouvoir être **copié-collé tel quel** dans un Flow et rendre identiquement.

### Règle anti-duplication
Avant d'écrire du markup dans un Flow, vérifier que la structure existe dans `Components.html`. Si elle existe : copier. Si elle existe à 80% : étendre le widget, pas le forker. Si elle n'existe pas : l'ajouter au catalogue d'abord.

## 🎨 Iconographie — librairie d'icônes du kit

L'iconographie est **un livrable concret**, pas un choix abstrait. Le kit doit contenir le set complet d'icônes utilisées dans les flows, sous forme exploitable et **visuellement identique à la famille choisie** (pas d'approximations).

### Choix de la famille

Choisir **une seule** famille (pas de mix) :
- **Lucide** : stroke, versatile, le plus safe — repo `github.com/lucide-icons/lucide`
- **Feather** : ancêtre de Lucide, similaire — `github.com/feathericons/feather`
- **Phosphor** : stroke ou fill, plus de variantes — `github.com/phosphor-icons/core`
- **Heroicons** : solid + outline, très web — `github.com/tailwindlabs/heroicons`
- **SF Symbols** : si produit iOS-first (export depuis l'app Apple SF Symbols)
- **Material Symbols** : si produit Android-first — `github.com/google/material-design-icons`

Stroke width 2px standard. Tailles canoniques : 16, 20, 24px.

### Récupération des SVG (étape obligatoire)

**Ne jamais ré-écrire les paths à la main.** Toujours fetch les SVG canoniques depuis la source officielle. Pour Lucide :

```
https://raw.githubusercontent.com/lucide-icons/lucide/main/icons/<name>.svg
```

Utiliser `WebFetch` (ou `find-docs`) pour récupérer chaque icône, puis sauvegarder verbatim dans `ds/assets/icons/<name>.svg`. Les noms d'icônes Lucide canoniques : `plus`, `check`, `chevron-left`, `chevron-right`, `ellipsis-vertical` (≠ "more-vertical"), `calendar`, `flag`, `info`, `folder`, `clock`, `user`, `trash-2` (≠ "trash"), `triangle-alert` (≠ "alert-triangle"), `clipboard-check`, `circle-check-big`, etc. **Vérifier sur lucide.dev avant de fetcher** — utiliser le nom exact du repo.

Pour les icônes custom (status bar iOS, badges spéciaux, glyphes spécifiques au produit), créer manuellement dans `ds/assets/icons/icon-NAME.svg` avec un viewBox cohérent.

### Format du livrable — deux étages

**Étage 1 — fichiers individuels** (`ds/assets/icons/<name>.svg`) :
- Un fichier par icône, contenu exact de la source officielle (Lucide ou autre)
- Utile pour : copie unitaire vers Figma, asset catalog iOS, drawable Android, export designer, **lecture par les outils de code-gen**
- Format : `<svg xmlns="…" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" …>…</svg>`

**Étage 2 — sprite consolidé** (`ds/assets/icons/icons.svg`) :
- Tous les SVG regroupés dans un seul fichier (référence designer + dev qui veut une vue d'ensemble)
- **Pas inliné dans le HTML** — sert uniquement de source de vérité agrégée

**Pattern d'usage dans les HTML (UI Kit + Flows) — SVG inlinés en plein avec `data-asset`** :
```html
<svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor"
     stroke-width="2" stroke-linecap="round" stroke-linejoin="round"
     data-asset="ds/assets/icons/plus.svg">
  <path d="M5 12h14"/>
  <path d="M12 5v14"/>
</svg>
```

**Pourquoi ce pattern** :
- ✅ **Rendu autonome** : chaque SVG embarque ses paths, fonctionne en `file://` sans le bug Chrome `<use href="external">`
- ✅ **Traçabilité** : `data-asset` pointe vers le fichier source dans `ds/assets/icons/`, ce qui permet aux outils de code-gen de matcher chaque icône inline avec son asset officiel et générer le bon import dans le projet mobile (`SvgPicture.asset('assets/icons/plus.svg')` Flutter, `Image('plus')` iOS, etc.)
- ✅ **Autonome par fichier** : un Flow HTML est lisible et auto-suffisant, pas de dépendance à un sprite à maintenir en bas du body
- ⚠️ **Coût** : duplication de SVG dans le HTML si la même icône est utilisée plusieurs fois. Acceptable pour un mockup (write-once) mais à garder en tête. Si une icône change, c'est `find/replace` ou rebuild depuis `ds/assets/icons/`.

**Comment construire le SVG inline** : copier le contenu exact de `ds/assets/icons/<name>.svg` (le fichier source), ajouter `data-asset="ds/assets/icons/<name>.svg"`, ajuster `width`/`height` selon le contexte d'usage.

⚠️ **Ne jamais utiliser** :
- `<use href="ds/assets/icons/icons.svg#…"/>` — chemin externe, cassé en `file://` Chrome
- `<svg>` inline sans `data-asset` — perd la traçabilité, les outils de code-gen ne savent pas quoi importer
- `<svg>` avec un path inventé / approximé — toujours coller depuis le fichier source officiel

### Inventaire minimum

Tout kit doit ship **au moins** ces icônes (adapter selon le produit) :

- **Navigation** : chevron-left, chevron-right, chevron-up, chevron-down, arrow-left, arrow-right, x (close), menu (hamburger), ellipsis-horizontal, ellipsis-vertical
- **Actions** : plus, check, pencil (edit), trash-2, copy, share, download, upload, search, filter, arrow-up-down (sort)
- **Status** : info, triangle-alert (warning), circle-x (error), circle-check (success), clock, loader
- **Objets** : user, users, calendar, clock, file, folder, image, link, bell, settings, house (home), star, heart, bookmark, flag, tag, lock, eye, eye-off
- **Toggle / form** : square (checkbox empty), square-check, circle (radio), circle-check

### Section Iconographie dans `UI Kit.html`

Une section dédiée dans la page unique qui rend **visuellement** toutes les icônes du dossier `ds/assets/icons/` (grille avec nom de fichier). Chaque icône est inlinée avec son `data-asset`. Pas de markup affiché. Sert à la fois de doc et de validation que les icônes sont bien chargées.

### Règles dures
- **Vrais SVG depuis la source officielle**, pas de path approximé à la main
- **SVG inlinés en plein dans le HTML** avec `data-asset` pointant vers `ds/assets/icons/<name>.svg`
- **Aucun `<use href>` externe** (cassé en `file://`)
- **Aucune icône inline sans `data-asset`** — perte de traçabilité pour les outils de code-gen
- **Pas de font icon** (Font Awesome, Material Icons font…). SVG inline uniquement
- Si un Flow a besoin d'une icône absente : la fetch depuis la source officielle, sauvegarder dans `ds/assets/icons/<name>.svg`, **puis** l'inliner dans le Flow avec son `data-asset`

## 📦 Assets non-icônes — brand, illustrations, images

Tout ce qui n'est pas une icône UI vit dans son dossier dédié sous `ds/`. Ne pas mélanger ces familles — chacune a une logique de production, de référencement et de mapping vers le code différente.

### Distinction par famille

| Famille | Vocabulaire visuel | Taille typique | Couleurs | Format | Référencement (matching key) | Exemples |
|---|---|---|---|---|---|---|
| **icons/** | symbole, glyphe | 16-24px | currentColor (1) | SVG | `<svg ... data-asset="ds/assets/icons/X.svg">…paths…</svg>` (inline) | plus, chevron, calendar |
| **brand/** | identitaire | 24-1024px | palette de marque | SVG (préféré) + PNG si besoin | `<img src="ds/assets/brand/...">` | logo, app-icon, splash |
| **illustrations/** | scène, narratif | 80-400px | multi-couleur DS | SVG | `<img src="ds/assets/illustrations/...">` | empty state, onboarding, decoration |
| **images/** | matière, texture | variable | photo / raster | PNG / JPG / WebP | `<img src="ds/assets/images/...">` ou `background-image: url(...)` | hero, photo, placeholder |

**Clé de matching** = ce qui permet aux outils de code-gen (scripts custom, agents IA) de relier un asset visible dans le HTML à son fichier source dans `ds/assets/`. Pour les icônes c'est `data-asset`, pour le reste c'est `src=` ou `url()` qui contient déjà le chemin.

### `ds/assets/brand/` — identité de marque

Contenu typique :
- `logo.svg` — version pleine (mark + wordmark si applicable)
- `logo-mark.svg` — symbole seul (pour favicon, watermarks)
- `logo-monochrome.svg` — variante 1 couleur, currentColor pour s'adapter au fond
- `logo-dark.svg` — version pour fonds sombres (couleurs inversées)
- `app-icon.svg` — source vectorielle 1024×1024 (l'export par taille devient un job du dev mobile)
- `favicon.svg` ou `favicon.ico`
- `splash/` — sous-dossier dédié au splash screen mobile (composition source + exports raster si fournis)

**Règle** : tout asset de marque que le développeur mobile va embarquer dans son app (asset catalog iOS, mipmap Android, assets/ Flutter) vit ici, à la source. Les exports multi-résolution sont optionnels — le dev sait les générer.

### `ds/assets/illustrations/` — scènes vectorielles

Contenu typique :
- Empty states custom (`empty-tasks.svg`, `empty-inbox.svg`)
- Onboarding (`onboarding-1.svg`, `onboarding-2.svg`, `onboarding-3.svg`)
- Error pages (`error-404.svg`, `error-network.svg`)
- Décor (formes, blobs, motifs vectoriels — `decoration-blob.svg`, `pattern-dots.svg`)
- Mascottes ou personnages

**Règle** : SVG only. Pour qu'une illustration s'adapte au dark mode et soit légère côté mobile (Flutter `flutter_svg`, iOS `SVGKit`, Android `VectorDrawable`).

Si une illustration ne peut pas être SVG (rendu 3D pré-calculé, photo composée), la mettre dans `images/` à la place.

### `ds/assets/images/` — matières raster

Contenu typique :
- Photos (hero, portraits, captures de produit)
- Placeholders (`placeholder-avatar.png`, `placeholder-card.jpg`)
- Patterns / textures (`pattern-bg@2x.png`)
- Screenshots (pour App Store, marketing)

**Règle nommage** : suffixer `@2x` / `@3x` si le designer fournit déjà les exports retina, sinon une seule version 2x suffit (le dev mobile gère le scaling). Format préféré : WebP > PNG (si transparence) > JPG (photo opaque).

### Section dans `UI Kit.html`

Documenter chaque famille dans une section dédiée du UI Kit (cf. structure UI Kit ci-dessus). Pour chaque asset : preview à taille raisonnable + nom de fichier en monospace sous l'asset. Pas de code, pas de chemin complet — juste le nom suffit.

### Règles dures (assets non-icônes)
- **Nommage** : kebab-case, descriptif (`empty-tasks.svg` pas `illustration1.svg`)
- **SVG sans dimensions fixes** : `viewBox` mais `width`/`height` omis sur l'élément `<svg>` racine — laisse le consommateur dimensionner via CSS / `<img width="…">`
- **Si asset utilisé une seule fois** dans tout le kit : à mettre quand même dans `ds/` (pour le rendre découvrable + reconsommable plus tard), pas en local dans le Flow
- **Ne pas dupliquer** entre `illustrations/` et `images/` : une scène vectorielle va dans illustrations, sa version raster (si fournie) reste dans illustrations aussi (`empty-tasks.png` à côté de `empty-tasks.svg`)
- **Splash screen** : la source vectorielle dans `brand/splash/splash.svg`. Si le designer fournit déjà les exports raster aux résolutions device (iPhone 14 Pro, etc.), les mettre à côté en `splash@2x.png`, `splash@3x.png`, etc.

## 🔤 Fonts — formats requis (woff2 + ttf)

Les fonts vivent dans `ds/assets/fonts/<font-name>/`. **Deux formats obligatoires** pour qu'un seul kit serve à la fois le mockup web et le dev mobile :

| Format | Cible | Pourquoi |
|---|---|---|
| `.woff2` | Mockup HTML, web React/Vue | Plus léger (~30-40% vs TTF), supporté tous browsers modernes |
| `.ttf` | Flutter (`pubspec.yaml`), iOS UIFont, Android typeface | Format natif que les frameworks mobile lisent — **ils ne lisent PAS le woff2** |

### Téléchargement automatique au moment de la création

Pour les fonts Google Fonts (cas majoritaire), le skill télécharge automatiquement les deux formats dans `ds/assets/fonts/<font>/` via WebFetch + Bash curl. Cf. workflow étape 3.5 pour les commandes exactes.

### Variable vs static

| Cas | Approche | Fichiers |
|---|---|---|
| **Font variable** dispo (Fraunces, Inter, Roboto Flex…) | Un seul `.woff2` + un seul `.ttf` couvrent toutes les graisses | `<font>-variable.woff2` + `<font>-variable.ttf` |
| **Font statique** uniquement | Un fichier par graisse, dans les deux formats | `<font>-400.woff2`, `<font>-500.woff2`, … + équivalents `.ttf` |

### `@font-face` dans `tokens.css`

```css
@font-face {
  font-family: 'Fraunces';
  font-style: normal;
  font-weight: 400 800;          /* range pour variable font, ou valeur unique pour statique */
  font-display: swap;
  src: url('assets/fonts/fraunces/fraunces-variable.woff2') format('woff2-variations'),
       url('assets/fonts/fraunces/fraunces-variable.ttf')   format('truetype-variations');
}
```

Le browser prend le premier format qu'il sait lire (woff2). Le `.ttf` est listé pour que le pipeline soit explicite et que Flutter / iOS / Android puissent le pointer directement depuis `pubspec.yaml` / `Info.plist` / `res/font/`.

### Fonts custom / payantes / SF Pro / etc.

Si la font n'est pas Google Fonts ou si elle est payante (Söhne, GT Walsheim, Calibre, etc.), l'utilisateur fournit les fichiers manuellement. Le skill :
1. Détecte la présence de `.woff2` et/ou `.ttf` dans `ds/assets/fonts/<font>/`
2. Si seul un format est fourni, signale au user et propose un convertisseur (FontSquirrel webfont generator pour générer le woff2 depuis un ttf, ou inverse via fonttools : `pyftsubset font.woff2 --output-file=font.ttf --flavor=…`)
3. Si fonts manquantes : fallback sur stack système (`-apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`) en attendant que le user fournisse

### Snippet `pubspec.yaml` à fournir au dev Flutter dans `GUIDELINES.md`

```yaml
flutter:
  fonts:
    - family: Fraunces
      fonts:
        - asset: assets/fonts/fraunces/fraunces-variable.ttf
    - family: Inter
      fonts:
        - asset: assets/fonts/inter/inter-variable.ttf
```

Le dev copie le contenu de `ds/assets/fonts/` directement dans `assets/fonts/` de son projet Flutter, ajoute le snippet ci-dessus à `pubspec.yaml`, et utilise via `TextStyle(fontFamily: 'Fraunces', fontVariations: [FontVariation('wght', 600)])`.

## ✏️ Texte dans les SVG assets — règles obligatoires

Tout `<text>` dans un SVG asset (`ds/assets/brand/*.svg`, `ds/assets/illustrations/*.svg`) doit respecter ces règles pour être correctement rendu côté code mobile (Flutter via `flutter_svg`, iOS via `SVGKit`, Android via `androidsvg`). Le browser tolère beaucoup, **les libs SVG mobiles non**.

### Règle 1 — Une seule famille de fonts (pas de fallback CSS)

Le `font-family` d'un `<text>` SVG doit contenir **une seule valeur**, sans liste CSS.

```svg
<!-- ❌ MAUVAIS — fallback CSS, ne match aucune font côté Flutter -->
<text font-family="Fraunces, Georgia, serif" font-size="24">Mon logo</text>

<!-- ✅ BON — une seule famille -->
<text font-family="Fraunces" font-size="24">Mon logo</text>
```

`flutter_svg` ne sait pas parser une liste CSS et ne tente pas de fallback : il cherche exactement la chaîne `"Fraunces, Georgia, serif"` comme nom de famille → introuvable → texte rendu en font système (souvent invisible ou cassé).

### Règle 2 — Pas d'attribut `font-variation-settings`

Cet attribut est non supporté par `flutter_svg`. Il n'est de toute façon pas nécessaire — pour les fonts variables, le poids et le style se passent via `font-weight` et `font-style` standards.

```svg
<!-- ❌ MAUVAIS — flutter_svg ignore cet attribut -->
<text font-family="Fraunces" font-variation-settings="'wght' 700, 'opsz' 14">…</text>

<!-- ✅ BON — attributs SVG standards -->
<text font-family="Fraunces" font-weight="700">…</text>
```

Le rendu est identique côté browser (les fonts variables interpolent automatiquement sur l'axe `wght`) et fonctionne côté Flutter.

### Règle 3 — La famille doit être déclarée dans `ds/assets/fonts/<x>/`

Toute famille référencée dans un `<text>` SVG doit avoir son fichier `.ttf` (ou `.otf`) présent dans `ds/assets/fonts/<x>/<x>-variable.ttf`. C'est ce fichier qui sera enregistré dans `pubspec.yaml > flutter > fonts:` côté projet Flutter.

Si le SVG utilise une famille absente de `ds/assets/fonts/` (ex: une font Google Fonts non téléchargée), le rendu côté Flutter est cassé même si le browser affiche correctement (le browser télécharge la font à la volée si CDN).

**Audit** : `grep -oE 'font-family="[^"]+"' ds/assets/{brand,illustrations}/*.svg | sort -u` → chaque famille listée doit exister dans `ds/assets/fonts/`.

### Bonus — Logos qui voyagent : text-to-outline

Pour les logos destinés à être utilisés **hors de l'app** (App Store screenshots, README, marketing, signature email, presse) : convertir les `<text>` en paths via "text-to-outline" dans un éditeur vectoriel (Figma, Illustrator, Inkscape) ou via un script (`opentype.js`, `text-to-svg`, etc.).

```svg
<!-- ✅ Logo qui voyage — paths au lieu de text -->
<svg viewBox="0 0 240 64" xmlns="http://www.w3.org/2000/svg">
  <path d="M12 16 L24 16 L24 48 L12 48 Z …" fill="var(--primary)"/>
  <!-- chaque lettre = un path, plus aucune dépendance à une font -->
</svg>
```

Avantages :
- Indépendant des fonts enregistrées côté Flutter / iOS / Android
- Rendu identique partout (browser, lib SVG mobile, navigateur de presse, README GitHub)
- Pas de risque que la font ne charge pas en `file://` ou en preview rapide

Inconvénients :
- Plus modifiable directement (modifier le wording = re-générer les paths)
- Fichier légèrement plus gros (≈ +2-5 KB selon la longueur du wordmark)

**Recommandation** : générer 2 versions du logo dans `ds/assets/brand/` :
- `logo.svg` avec `<text>` (édition facile, rendu app via fonts enregistrées)
- `logo-outlined.svg` avec paths (App Store, marketing, README, partage externe)

## 📱 Écrans à mocker

Pour chaque écran prioritaire, générer une cellule dans un fichier Flow HTML. Couvrir toujours ces **5 états minimum** si pertinent :

1. **Default** (happy path)
2. **Loading** (skeleton ou spinner)
3. **Empty** (aucune donnée)
4. **Error** (retry possible)
5. **Permission / Auth gate** (si applicable)

## 📐 Structure d'un fichier Flow HTML

Localisation : `flows/<flow>/<page>.html` (cf. arborescence). Un Flow **ne définit aucun style de widget**. Il importe le DS (tokens + components), inline les icônes SVG en plein avec `data-asset`, et ne contient que du markup composé à partir du catalogue. Les seuls styles locaux acceptables sont la grille canvas et la frame de plateforme. Les chemins vers le DS sont en relatif `../../ds/...` depuis `flows/<flow>/`.

```html
<!doctype html>
<html lang="[langue]">
<head>
  <meta charset="utf-8" />
  <title>[Produit] — [Flow] — [Page]</title>
  <link rel="stylesheet" href="../../ds/tokens.css">
  <link rel="stylesheet" href="../../ds/components.css">
  <style>
    body { margin:0; padding:40px 28px 80px; background:var(--bg); }
    .canvas {
      max-width:1480px; margin:0 auto;
      display:grid; grid-template-columns:repeat(3, 1fr); gap:28px;
    }
    @media (max-width:1100px) { .canvas { grid-template-columns:repeat(2, 1fr); } }
    @media (max-width:760px)  { .canvas { grid-template-columns:1fr; } }
    .cell { background:var(--surface); border-radius:var(--radius-lg); padding:22px 20px 26px; box-shadow:var(--shadow-md); }
    .cell-head { display:flex; justify-content:space-between; margin-bottom:8px; }
    .cell-head .name { font-weight:600; font-size:16px; }
    .cell-head .tag { font-family:ui-monospace; font-size:11px; color:var(--text-muted); background:var(--neutral-100); padding:2px 8px; border-radius:4px; }
    .cell-desc { color:var(--text-muted); font-size:13px; margin-bottom:18px; line-height:1.5; }

    /* Frame selon plateforme : .phone pour mobile, .browser pour web, .app-window pour desktop */
  </style>
</head>
<body>
  <div class="page-head">
    <h1>[Page]</h1>
    <div class="sub">[Description : flow / nb cellules / états]</div>
  </div>
  <div class="canvas">
    <!-- data-screen-label="<NN> <Page> <state>" — le dernier mot = state_id machine-readable.
         Pour flows/01-tasks/list.html, les cells seraient :
           "01 List default" → state_id="default"
           "02 List loading" → state_id="loading"
           "03 List empty"   → state_id="empty"   -->
    <div class="cell" data-screen-label="01 List default">
      <div class="cell-head"><div class="name">① Default</div><div class="tag">happy path</div></div>
      <div class="cell-desc">Contexte.</div>
      <!-- frame approprié + écran composé uniquement de classes du DS.

           ICÔNES APP (consommées par l'app, ship dans le bundle) :
             <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor"
                  stroke-width="2" stroke-linecap="round" stroke-linejoin="round"
                  data-asset="../../ds/assets/icons/plus.svg">
               <path d="M5 12h14"/>
               <path d="M12 5v14"/>
             </svg>

           ÉLÉMENTS OS-CHROME (rendus par le système, JAMAIS de data-asset) :
             <div class="status-bar" data-os-chrome="status-bar">
               <span>9:41</span>
               <svg width="16" height="11"><path .../></svg>  <!-- skip par héritage -->
               <svg width="14" height="10"><path .../></svg>  <!-- skip par héritage -->
             </div>
             <div class="dynamic-island" data-os-chrome="dynamic-island"></div>
             <div class="home-indicator" data-os-chrome="home-indicator"></div>

           SÉMANTIQUE — fonction native (l'agent skip l'UI custom, utilise widget partagé sinon API native) :
             <button class="btn btn--primary"
                     data-uses="native:image-picker"
                     data-uses-context="multi-select max 10, allow-video=false">
               Ajouter une photo
             </button>

           SÉMANTIQUE — pattern UI au comportement non-évident :
             <div class="carousel" data-uses="ui:carousel">…</div>
             <div class="dots" data-uses="ui:dot-indicator">…</div>
             <div class="sheet" data-uses="ui:bottom-sheet">…</div>

           HINT métier libre (galerie BDD vs native, swipe behavior, etc.) :
             <div class="grid" data-hint="Galerie photos en BDD app via PhotoRepository.list(), pagination 20/page">…</div>

           NAVIGATION cliquable (consommée par ui-kit-prototype + code-gen) :
             <button class="btn btn-primary" data-nav-target="tasks/list:default">Se connecter</button>
             <div class="task-row" data-nav-target="tasks/detail:default">…</div>
             <button class="nav-action" data-nav-target="back" data-nav-back="true">←</button>

           IMAGES / ILLUSTRATIONS (référencées par chemin) :
             <img src="../../ds/assets/illustrations/empty-tasks.svg" alt="" width="120">
             <img src="../../ds/assets/brand/logo.svg" alt="TaskMate" height="24">
      -->
    </div>
  </div>
</body>
</html>
```

## 🖼️ Frames selon plateforme

- **Mobile iOS** : phone frame 340×720 avec notch + statusbar + app bar + nav bar
- **Mobile Android** : phone frame 340×720 avec statusbar Material + bottom nav
- **Web desktop** : `.browser-chrome` 1280×800 avec tabs + url bar
- **Web mobile responsive** : frame 390×844 sans chrome natif
- **Desktop app** : `.app-window` avec traffic lights macOS OU title bar Windows

## 🏷️ Conventions sémantiques machine-readable

En plus de `data-asset` (matching asset → fichier source) et `data-screen-label` (états des cells), trois conventions sémantiques permettent aux outils de code-gen de générer du Dart/Swift/Kotlin pertinent au lieu de coder aveuglément ce qu'ils voient.

### `data-os-chrome` — éléments rendus par l'OS (skip total)

Tout élément visible dans le mockup mais que **le système d'exploitation rend lui-même** doit porter cet attribut. L'agent codeur skip totalement ces éléments — pas de widget Flutter, pas de `Container` mock, pas de `MockStatusBar`. Le code respecte juste la `SafeArea` correspondante.

**Vocabulaire fermé** :

| Valeur | Quoi | Mapping mobile |
|---|---|---|
| `device-frame` | Bezel décoratif du phone (mockup-only) | Skip total |
| `dynamic-island` | Île dynamique iOS | Skip — système |
| `notch` | Encoche écran | Skip — système |
| `camera-cutout` | Punch-hole Android | Skip — système |
| `status-bar` | Barre statut (heure + signal/wifi/batterie + Dynamic Island content) | Skip widgets, **`SafeArea.top`** réservé |
| `home-indicator` | Indicateur de glisse iOS | Skip, **`SafeArea.bottom`** réservé |
| `gesture-bar` | Barre gestuelle Android | Skip, inset bottom réservé |
| `keyboard` | Clavier système quand visible | Skip widgets, signaler "écran avec clavier ouvert" pour les tests d'intégration |
| `keypad` | Clavier numérique | Skip, idem |

**Héritage** : si un parent porte `data-os-chrome`, **tous ses descendants sont skipés automatiquement**. Pas besoin de marquer chaque enfant individuellement. Un agent codeur lit le DOM et remonte vers ancestor pour décider.

**Exemple** :
```html
<!-- 1 seul attribut sur le conteneur, suffit à skipper tous les <svg> enfants -->
<div class="status-bar" data-os-chrome="status-bar">
  <span>9:41</span>
  <span class="icons">
    <svg width="16" height="11"><path .../></svg>  <!-- skipé par héritage -->
    <svg width="14" height="10"><path .../></svg>  <!-- skipé par héritage -->
    <svg width="22" height="10"><rect .../></svg>  <!-- skipé par héritage -->
  </span>
</div>
<div class="dynamic-island" data-os-chrome="dynamic-island"></div>
<div class="home-indicator" data-os-chrome="home-indicator"></div>
```

**Règle dure** : les SVG d'icônes système (signal, wifi, battery, time) ne portent **JAMAIS** `data-asset`. Ce ne sont pas des assets de l'app. Ils sont du chrome décoratif du mockup, parent marqué `data-os-chrome`, point.

### `data-uses` — pattern UI ou fonction native sémantique

Indique au générateur de code la **sémantique** d'un élément quand le visuel statique seul ne suffit pas. Deux namespaces distincts :

#### `native:*` — fonction du device (l'agent ne code pas d'UI)

L'élément sollicite une API native (image picker, camera, biometric…). L'agent codeur applique cet **ordre de priorité strict** :

1. **Chercher d'abord** dans `ui_components/` (widgets partagés du projet) un widget qui couvre cette sémantique (ex: `SharedMap`, `SharedDatePicker`, `SharedImagePicker`)
2. Si trouvé → **utiliser le widget partagé**. Le projet peut wrapper MapBox, Apple Maps, ou tout autre fournisseur, c'est l'affaire du widget partagé pas du générateur
3. Si rien d'existant → fallback sur l'API native du framework cible
4. **Jamais** : recoder une UI custom alors qu'une fonction native existe

**Vocabulaire** :

| Valeur | Sémantique |
|---|---|
| `native:image-picker` | Sélectionner photo(s) depuis la pellicule |
| `native:camera` | Prise photo/vidéo via capteur |
| `native:file-picker` | Sélecteur fichier système |
| `native:date-picker` / `native:time-picker` / `native:date-range-picker` | UI native sélection date/heure/plage |
| `native:map` | Carte interactive (zoom, pan, marqueurs) |
| `native:share-sheet` | Partage système |
| `native:contacts` | Sélection contact carnet d'adresses |
| `native:scanner-qr` / `native:scanner-document` / `native:scanner-credit-card` | Scanner via caméra |
| `native:biometric-auth` | Touch ID / Face ID / empreinte |
| `native:notification-permission` / `native:location-permission` / `native:camera-permission` / `native:microphone-permission` | Demande de permission système |
| `native:web-view` | Browser embarqué |
| `native:haptic-feedback` | Vibration tactile |
| `native:audio-recorder` / `native:video-player` / `native:pdf-viewer` | Médias avec contrôles natifs |
| `native:in-app-purchase` | StoreKit / Google Play Billing |

#### `ui:*` — pattern UI dont le comportement échappe au visuel statique

Quand un visuel ne révèle pas le comportement (un carousel ressemble à 3 cards alignées, un bottom sheet ressemble à un panel en bas — mais leurs comportements sont radicalement différents).

| Valeur | Pourquoi marquer (ce que l'agent ne devine pas du CSS) |
|---|---|
| `ui:carousel` | Scroll horizontal avec snap, swipe gestures, généralement avec `ui:dot-indicator` |
| `ui:dialog` | Modal centré + barrier + escape-to-close + focus-trap |
| `ui:alert-dialog` | Dialog de confirmation système (Yes/No, Confirm/Cancel) |
| `ui:bottom-sheet` | Feuille modale draggable depuis le bas |
| `ui:bottom-sheet-persistent` | Bottom sheet avec snap-points multiples |
| `ui:action-sheet` | iOS-style action sheet (liste de choix verticaux) |
| `ui:app-bar` | Top bar de page (optionnel : iOS large-title qui shrink au scroll) |
| `ui:bottom-nav` | Navigation principale 3-5 onglets racines |
| `ui:tab-bar` | Onglets segmentés dans une page (pas de navigation entre pages) |
| `ui:segmented-control` | Choix exclusif 2-3 options inline |
| `ui:dot-indicator` | Dots de pagination (carousel, onboarding) |
| `ui:stepper` | Navigation séquentielle multi-étapes avec progression |
| `ui:snackbar` | Feedback temporaire en bas de l'écran (auto-dismiss) |
| `ui:tooltip` | Popover info au hover/long-press |
| `ui:popup-menu` | Menu contextuel sur clic |
| `ui:autocomplete` | Input avec suggestions filtrées en dropdown |
| `ui:pull-to-refresh` | Drag down beyond top → trigger refresh |
| `ui:infinite-scroll` | Pagination automatique au scroll |
| `ui:swipe-actions` | Swipe pour révéler actions cachées (iOS-style) |
| `ui:expansion-panel` | Accordion / collapsible content |
| `ui:skeleton-loader` | Placeholder shimmer pendant chargement |
| `ui:progress-linear` / `ui:progress-circular` | Indicateurs de progression |
| `ui:slider` | Range input continu draggable |
| `ui:rating-stars` | Notation par étoiles tappables |

**Règle de tri** : si le pattern est entièrement déductible des classes CSS (un `.btn` est évidemment un bouton, un `.task-row` une row de liste), pas besoin de `data-uses`. On marque uniquement quand le **comportement dynamique** ne saute pas aux yeux.

**Vocabulaire vivant** : un `data-uses` inconnu est signalé par les outils de code-gen dans `unknowns`, sans fail. On enrichit au fil des besoins en éditant le UI_KIT_CONTRACT.md de Mirror.

### `data-uses-context` — paramétrage textuel libre

Complète `data-uses` quand l'API/widget cible peut être paramétré (multi-select, max items, options spécifiques). Texte libre lu par le LLM.

```html
<button class="btn btn--primary"
        data-uses="native:image-picker"
        data-uses-context="multi-select, max 10 photos, allow-video=false, ouvre sur album 'Récent'">
  Ajouter une photo
</button>
```

### `data-hint` — hint sémantique général libre

Pour les cas où aucune valeur de vocabulaire fermé ne couvre — l'agent doit comprendre la **logique métier** que le visuel ne montre pas.

```html
<!-- Galerie photos APP (BDD utilisateur) — PAS native, custom UI à coder -->
<div class="grid"
     data-hint="Galerie de photos stockées en BDD app via PhotoRepository.list(). Pagination 20/page, tap → lightbox custom.">
  ...
</div>

<!-- Liste avec swipe -->
<ul class="task-list"
    data-hint="Swipe-to-delete iOS-style. Utiliser Dismissible Flutter, pas un long-press menu.">
  ...
</ul>
```

Ne pas mettre `data-hint` partout — uniquement quand l'agent risque de mal interpréter. Pour 80% des composants, les classes CSS du DS suffisent.

### `data-nav-target` — déclaration de navigation

Quand un CTA / row / lien navigue vers un autre écran (autre flow, autre page, autre état), le déclarer explicitement pour que le **prototype interactif** (cf. skill `ui-kit-prototype`) puisse intercepter le clic et naviguer, ET pour que les outils de code-gen sachent générer la route Flutter sans deviner.

**Format** : `<flow_id>/<page_id>` (état `:default` implicite) ou `<flow_id>/<page_id>:<state_id>` pour cibler un état précis.

```html
<!-- Submit du login → liste des tâches (état default) -->
<button class="btn btn-primary" data-nav-target="tasks/list:default">
  Se connecter
</button>

<!-- Click sur une row → écran détail -->
<div class="task-row" data-nav-target="tasks/detail:default">
  ...
</div>

<!-- Bouton "Nouvelle tâche" → modal d'édition (= état "edit" de la même page) -->
<button class="btn-fab" data-nav-target="tasks/list:edit-modal">
  <svg ...><use href="#icon-plus"/></svg>
</button>

<!-- Bouton retour : pop la pile, équivalent Navigator.pop() Flutter -->
<button class="nav-action" data-nav-target="back" data-nav-back="true">
  <svg ...><use href="#icon-chevron-left"/></svg>
  Retour
</button>

<!-- Lien vers un autre flow -->
<a class="link" data-nav-target="settings/profile:default">Mon profil</a>
```

**Co-bénéfices** :
- Le designer décrit la navigation côté maquette, pas le dev par devinette côté code
- Le skill `ui-kit-prototype` lit ces attributs et fait clignoter sur clic comme un Figma prototype
- Les outils de code-gen peuvent générer `Navigator.pushNamed('/<flow>/<page>')` directement depuis le bouton

**Règle** : poser `data-nav-target` sur **tout élément cliquable qui navigue** (button, .task-row, .nav-action, a). Ne pas en poser sur les éléments qui ouvrent juste un menu contextuel (`data-uses="ui:popup-menu"`) ou un toast — ce ne sont pas des navigations entre pages.

### Récapitulatif — quel attribut quand ?

| Tu veux exprimer… | Attribut | Lecture |
|---|---|---|
| "Ce SVG correspond à `ds/assets/icons/X.svg`" | `data-asset` | regex tools |
| "Cet élément est rendu par l'OS, skip" | `data-os-chrome` (héritage) | regex tools |
| "C'est une fonction native du device" | `data-uses="native:..."` | regex tools + LLM |
| "C'est un pattern UI au comportement non-évident" | `data-uses="ui:..."` | regex tools + LLM |
| "Paramètres de la fonction/pattern ci-dessus" | `data-uses-context="..."` | LLM |
| "Logique métier non visible (galerie BDD vs native, swipe behavior, etc.)" | `data-hint="..."` | LLM |
| "État visuel de la cell" | `data-screen-label` | regex tools |
| "Cet élément navigue vers un autre écran" | `data-nav-target="<flow>/<page>[:<state>]"` (+ optionnel `data-nav-back`) | regex tools + prototype runtime + code-gen |

## ✍️ `UI Kit.html` — page unique tout-en-un

Une seule page de doc visuelle. Tout est dedans, rendu visuellement, **sans aucun bloc de code apparent**. Structure :

```
[Header — nom produit + version + plateforme + mode + typo + famille d'icônes]
[Nav sticky avec ancres vers chaque section]

[Section Couleurs]        — swatches groupés (Brand / Sémantique / Surfaces & texte)
[Section Typographie]     — échelle complète, exemples lisibles
[Section Spacing]         — règle graduée
[Section Radii]           — cellules visuelles
[Section Élévations]      — cards en shadows croissantes
[Section Motion]          — durées + courbes (référence dev)

[Section Brand]           — galerie de l'identité :
                            - Logo principal (sur fond clair + sur fond sombre)
                            - Logo mark seul
                            - Logo monochrome (positif + négatif)
                            - App icon (avec masque rond iOS + carré Android)
                            - Favicon
                            - Splash screen (composition complète, à taille réduite)
                            Sous chaque asset : nom de fichier en mono.

[Section Iconographie]    — galerie complète du sprite (grille de cells icône+nom),
                            sous-sections : Lucide stroke / Filled / chrome iOS /
                            modifiers de couleur

[Section Illustrations]   — galerie des scènes vectorielles
                            (empty states, onboarding, decorative). Rendues à taille
                            réelle ou réduite cohérente. Nom de fichier sous chaque.

[Section Images]          — galerie des assets raster (photos, hero, placeholders).
                            Rendues en thumbnails. Nom de fichier sous chaque.
                            Si > 20 images : grouper par usage (Hero / Placeholders /
                            Patterns).

[Section Composants]      — rendu visuel de chaque widget shared, organisé par famille :
                            Boutons, FAB, Filter chips, Avatars, Meta pills, Cards,
                            Sections groupées + Task rows, Inputs & forms, Field rows,
                            Bottom sheet, Detail title block, Notes block, Empty state,
                            Skeleton loaders. Pas de markup apparent — juste le rendu +
                            une mini-ligne `.classname` discrète sous chaque preview.

[Section iOS Chrome]      — status bar / nav bar / large title / tab bar (si iOS)
                            Idem Android Chrome si applicable.

[Pas de sprite — chaque icône est inlinée en plein SVG au point d'usage,
 avec son attribut data-asset="ds/assets/icons/<name>.svg" pour la traçabilité]
```

**Sections Brand / Illustrations / Images optionnelles selon le scope** : si le projet n'a pas encore de splash, ne pas créer la section vide — créer la sous-section quand l'asset existe.

**Règles** :
- **Aucun `<pre><code>`** qui affiche le code des composants. Le UI Kit est une référence visuelle. Le dev qui implémente regarde le rendu et code dans son framework cible (Flutter, SwiftUI, Compose, React Native…).
- Sous chaque preview de composant, OK de mettre un libellé monospace discret avec les classes utilisées (`.btn · .btn-primary · .btn-pill`) — c'est de la doc, pas du markup HTML.
- La section Iconographie rend **toutes** les icônes du dossier `ds/assets/icons/`, chacune inlinée avec son `data-asset`. Sert de check de validation : si une icône s'affiche "vide" dans cette galerie, c'est qu'il y a un problème de markup.

## 🚫 Anti-patterns à éviter

1. **Gradients décoratifs** partout (slop IA typique)
2. **Glassmorphism** sans raison
3. **Rounded borders avec accent à gauche** (cliché)
4. **Inter/Roboto par défaut** sans considération du ton
5. **Emoji dans l'UI** (garder pour celebration states très localisés)
6. **Illustrations SVG inline complexes** (utiliser placeholder)
7. **Palette de 50 couleurs** (rester ≤12)
8. **Font icons** (Font Awesome etc.) — préférer SVG inline
9. **Line-height serré** sur body text (viser 1.4-1.6)
10. **Contrastes borderline** (forcer ≥ 4.5:1 sur tout texte)
11. **Widgets définis localement dans un Flow** — tout doit venir de `ds/components.css`. Si absent : ajouter au DS d'abord.
12. **Icône inline sans `data-asset`** — perte de traçabilité pour les outils de code-gen. Toujours `data-asset="ds/assets/icons/<name>.svg"` qui pointe vers le fichier source.
13. **Référence sprite externe** via `<use href="ds/assets/icons/icons.svg#…"/>` — cassé en `file://` Chrome.
14. **Paths SVG ré-écrits à la main** — toujours fetch depuis la source officielle (lucide-icons GitHub, etc.), c'est non négociable.
15. **Pages de doc séparées** (Components.html, Icons.html) — tout doit être dans `UI Kit.html`, page unique.
16. **Markup HTML apparent** dans la doc (`<pre><code>` avec snippets) — la doc est visuelle, pas technique.
17. **Flow à la racine** (`Flow 01 - Tasks.html`) — toujours sous `flows/<flow>/<page>.html` (un dossier par flow, un fichier par page).
18. **Composant "presque pareil" dupliqué sous un nouveau nom** — étendre le widget existant via modifier (`.card.card--interactive`) plutôt que créer `.card-2`.
19. **Inline styles sur les "patterns" du DS** (titres manuscript, citations, headers spéciaux). Si un pattern visuel apparaît dans plus d'un endroit, créer une classe dans `ds/components.css` (ex: `.recipe-card__manuscript-title`) plutôt que de répéter `style="font-family:...; font-size:...; font-style:..."`. Sinon : modifier le pattern devient un find/replace risqué dans tous les Flows.
20. **Screenshot `fullPage: true` sur UI Kit** — la page fait 8 000-15 000 px de haut, le modèle ne peut pas lire un PNG de 30 000 px en retina. Toujours viewport-by-viewport (cf. § Validation visuelle).
21. **Caractères spéciaux non-échappés dans les SVG** (`&`, `<` bruts dans `aria-label`, `<title>`, attributs). Les SVG chargés via `<img>` sont parsés en mode XML strict ; le browser les rejette silencieusement et affiche l'icône broken. Toujours `&amp;`, `&lt;`, `&gt;`. Valider via `xmllint --noout` (cf. workflow étape 6.5).
22. **Boutons trop gros sur mobile** (`.btn--lg` 52px, `.btn--xl` 60px). Ces variantes existent dans certains DS web mais **ne pas les ship sur un kit mobile**. La hauteur 44px par défaut est exactement le touch-target Apple HIG / Material guidelines — assez gros pour le doigt, sans écraser visuellement la mise en page mobile (où chaque pixel compte). Plus grand = écrasement de la hiérarchie visuelle, perte d'espace pour le contenu. Si tu veux un CTA "important", joue sur la couleur (primary), la position (collante en bas), ou `btn--block` pour la pleine largeur — pas sur la taille.
   Conséquence : **dans `ds/components.css`, ne PAS définir `.btn--lg` ni `.btn--xl`**. Garder uniquement `.btn--sm` (36px) pour les boutons inline dans les chips/badges/tags, et la taille par défaut (44px) pour tout le reste.
23. **`@import` Google Fonts CDN dans un kit livré** (`@import url('https://fonts.googleapis.com/css2?...')` en tête de `tokens.css`). Ça marche pour le mockup mais : casse en offline, force le dev mobile à re-télécharger les fonts ailleurs (avec risque de version drift), et **Flutter ne supporte pas le woff2** que la CDN sert. Toujours **télécharger les fonts dans `ds/assets/fonts/`** au moment de la création (cf. workflow étape 3.5) — au moins **woff2 + ttf**, les deux formats étant requis (woff2 web, ttf Flutter / iOS / Android natif).
24. **`data-asset` sur des SVG du chrome système** (signal, wifi, battery, time). Ces icônes sont rendues par l'OS — elles **ne sont pas des assets de l'app**. Si tu mets `data-asset` dessus, le générateur de code va les compter comme assets à importer dans le projet mobile → constantes mortes (`AppIcons.signal`) et `signal.svg` embarqué dans l'APK pour rien. Le parent doit porter `data-os-chrome="status-bar"`, point — les enfants sont skipés par héritage.
25. **Mock OS chrome dans le code généré attendu** : si tu écris dans GUIDELINES.md "le dev devra coder une StatusBar custom", c'est faux. Le système la dessine. La page Flutter commence sous `SafeArea`. Pas de `MockKeyboard`, pas de `Container(color: Colors.black, height: 60)` pour faire semblant. Ne jamais demander/laisser sous-entendre cette implémentation.
26. **Carousel / bottom-sheet / dialog sans `data-uses="ui:..."`** : un visuel statique ne révèle pas le comportement (snap, drag, modal barrier). L'agent codeur va recoder une UI custom en pensant que c'est juste 3 cards alignées. Toujours marquer ces patterns avec `data-uses="ui:carousel|bottom-sheet|dialog|..."`.
27. **Fonctionnalité native (image-picker, camera, biometric, etc.) recodée en widget custom** : si la maquette montre "Ajouter une photo" en bouton, c'est probablement un image-picker natif. Marquer `data-uses="native:image-picker"` pour que l'agent priorise le widget partagé du projet OU l'API native, pas une UI custom.
28. **Galerie BDD app sans `data-hint`** : visuellement identique à une galerie native, mais le code à générer est radicalement différent. Si la galerie affiche des photos depuis la BDD utilisateur (pas la pellicule device), le marquer en `data-hint="Galerie en BDD app via XxxRepository, pagination N/page"` pour lever l'ambiguïté.
29. **CTA / row / lien navigant vers un autre écran sans `data-nav-target`** : le skill `ui-kit-prototype` ne pourra pas intercepter le clic, et l'agent codeur devra deviner où ce bouton mène. Coût : prototype incomplet + bug-prone côté routing. Toujours poser `data-nav-target="<flow>/<page>[:<state>]"` dès que le clic navigue. Pour les boutons retour : `data-nav-target="back" data-nav-back="true"`.
30. **`<text>` SVG avec font-family CSS-list (`"A, B, serif"`) ou `font-variation-settings`** : casse le rendu côté `flutter_svg` (parse strict, pas de fallback, pas de variation-settings). Voir § "Texte dans les SVG assets" : une seule famille, `font-weight`/`font-style` standards, famille présente dans `ds/assets/fonts/`.

## 📚 Templates des 3 fichiers de documentation

À l'étape 14 du workflow, écrire ces 3 fichiers à la racine du kit. Ils homogénéisent la sortie entre projets et permettent à un dev (ou un autre LLM) de reprendre le projet sans contexte. **Adapter le contenu au projet courant** mais respecter la structure des templates.

### `GUIDELINES.md` — pour le dev qui implémente côté code

Sections obligatoires :

```markdown
# Guidelines — [Nom du produit]

Document de référence pour implémenter le design system côté code (Flutter / SwiftUI / Compose / React / Vue).

## Tokens (source de vérité : `ds/tokens.css`)

| Catégorie | Variables | Mapping code |
|---|---|---|
| Couleurs | `--primary`, `--surface`, `--bg`, `--text`, `--text-muted`… | `AppColors.*` (Flutter) / `Color(...)` (SwiftUI) / `colorResource(...)` (Compose) |
| Typo | `--font-display`, `--font-body`, échelle `--fs-xs` → `--fs-3xl` | `AppTypography.*` |
| Spacing | `--space-4` → `--space-64` | `AppSpacing.*` |
| Radii | `--radius-sm`, `--radius-md`, `--radius-lg`, `--radius-pill` | `AppRadii.*` |
| Élévations | `--shadow-sm`, `--shadow-md`, `--shadow-lg` | `AppShadows.*` |
| Motion | `--motion-fast`, `--motion-normal`, `--motion-slow` (durations + easings) | `AppMotion.*` |

## Composants (catalogue dans `UI Kit.html` § Composants)

Liste les widgets partagés du kit avec leur API attendue côté code :

| Classe HTML | Widget cible | Props / variants |
|---|---|---|
| `.btn`, `.btn--primary`, `.btn--ghost`, `.btn--danger` | `<Button variant="primary|ghost|danger">` | size, leadingIcon, loading, disabled |
| `.card`, `.card--elevated` | `<Card elevated={bool}>` | — |
| `.task-row` | `<TaskRow task={...} onTap={...}>` | swipeActions optionnelles |
| `.dialog` | API native (`showDialog` Flutter, `.alert` SwiftUI, `AlertDialog` Compose) | — |
| `.bottom-sheet` | `showModalBottomSheet` Flutter, `.sheet` SwiftUI, `ModalBottomSheet` Compose | détents, dismissable |
| … | … | … |

## Conventions sémantiques (rappel — détaillé dans le README du repo skills)

| Attribut | Action côté code |
|---|---|
| `data-os-chrome="<type>"` | NE PAS générer de widget — wrapper le body dans `SafeArea` (Flutter) / safeAreaInsets (SwiftUI) / `statusBarsPadding()` (Compose) |
| `data-asset="ds/assets/icons/<name>.svg"` | Importer l'asset → `IconView(iconPath: AppIcons.<name>)` (wrapper projet) ou `SvgPicture.asset(...)` |
| `<img src="ds/assets/illustrations/<name>.svg">` | `ImageView(imagePath: AppIllustrations.<name>)` (wrapper projet) ou `Image.asset(...)` |
| `<img src="ds/assets/images/<name>.<ext>">` | `ImageView(imagePath: AppImages.<name>)` ou `Image.asset(...)` |
| `data-nav-target="<flow>/<page>[:<state>]"` | `Navigator.pushNamed('/<flow>/<page>')` / `context.push(...)` / `navController.navigate(...)` |
| `data-nav-target="back" data-nav-back="true"` | `Navigator.pop()` / `context.pop()` / `popBackStack()` |
| `data-uses="native:<feature>"` | Widget partagé du projet en priorité, package natif en fallback |
| `data-uses="ui:<pattern>"` | Primitive framework (PageView, BottomSheet…) au lieu de recodage custom |
| `data-uses-context="..."` | Transmettre les params à l'API native |
| `data-hint="..."` | Annotation pour la génération (texte rotatif, source BDD, swipe différencié…) |
| `data-auto-advance="<flow>/<page>[:<state>]"` + `data-auto-advance-delay="<ms>"` | `Timer` / `LaunchedEffect` / `.task` avec cancel-on-dispose |

## Snippet stack-spécifique

[Selon stack cible :]

### Si Flutter — `pubspec.yaml`
\`\`\`yaml
flutter:
  fonts:
    - family: <FontDisplay>
      fonts:
        - asset: assets/fonts/<font>/<font>-variable.ttf
  assets:
    - assets/icons/
    - assets/illustrations/
    - assets/images/
    - assets/brand/
\`\`\`

### Si iOS — Info.plist + Asset Catalog
\`\`\`xml
<key>UIAppFonts</key>
<array>
  <string><Font>-Variable.ttf</string>
</array>
\`\`\`
+ Drag-and-drop des SVG dans Asset Catalog avec target sélectionné.

### Si Compose — `app/build.gradle.kts` + `res/`
- Copier les `.ttf` dans `app/src/main/res/font/` (snake_case obligatoire)
- Copier les SVG dans `res/drawable/` ou utiliser une lib SVG

### Si React/Vue — bundler
- Fonts via `@font-face` (déjà fait dans `tokens.css`, copier-coller)
- Assets via import statique ou `public/`

## Anti-patterns du DS

- Ne pas hardcoder de couleur/spacing/radius — toujours passer par les tokens
- Ne pas créer un nouveau widget si une classe existe déjà
- Ne pas modifier `ds/tokens.css` côté projet — c'est la source de vérité du kit
- Ne pas générer de widget pour `data-os-chrome` (status-bar, home-indicator…)
- Ne pas mapper les icônes vers une autre famille que `ds/assets/icons/`

## Comment ajouter un widget / icône / asset

1. **Nouveau widget** : invoquer `ui-kit-editor` avec "ajouter le composant <X> au kit". Le skill ajoute la classe + le mock + une entrée dans `UI Kit.html § Composants`.
2. **Nouvelle icône** : invoquer `ui-kit-editor` avec "ajouter l'icône <name> du set Lucide". Le skill télécharge le SVG dans `ds/assets/icons/<name>.svg` et l'inline dans la galerie.
3. **Nouvelle illustration / image** : poser le fichier dans `ds/assets/illustrations/` ou `ds/assets/images/` et la référencer en `<img src="...">` dans la page concernée + dans la galerie de `UI Kit.html`.
4. **Nouveau flow / page** : invoquer `ui-kit-editor` avec "créer le flow <X> avec les pages <a>, <b>". Le skill crée `flows/NN-<x>/<a>.html` et `<b>.html` en respectant les conventions.
```

### `README.md` — pour quelqu'un qui ouvre le repo

```markdown
# [Nom du produit] — UI Kit

Maquettes hi-fi du produit en HTML/CSS pur. Source de vérité du design system avant implémentation côté code.

## Quickstart

\`\`\`bash
# Ouvrir la doc visuelle complète
open "UI Kit.html"

# Ouvrir un flow particulier
open flows/01-onboarding/01-welcome.html

# Ouvrir le prototype interactif (si présent)
open Prototype.html
\`\`\`

Tout fonctionne en `file://` — aucun serveur ni build requis.

## Arborescence

\`\`\`
[projet]/
├── ds/                       Design system (source de vérité)
│   ├── tokens.css            Variables CSS (couleurs, typo, spacing, radii…)
│   ├── components.css        Classes des widgets partagés
│   └── assets/               Tous les assets, par famille
│       ├── icons/            Icônes UI (Lucide canonique)
│       ├── brand/            Logo, favicon, splash, app-icon
│       ├── illustrations/    SVG multi-couleur (empty states, onboarding…)
│       ├── images/           PNG / JPG / WebP
│       └── fonts/            .woff2 (web) + .ttf (Flutter / iOS / Android)
│
├── UI Kit.html               Page unique de doc visuelle
│                             (Couleurs / Typo / Spacing / Composants / Iconographie / Brand…)
│
├── flows/                    Flows utilisateur
│   ├── 01-<flow>/
│   │   ├── <page>.html       1 fichier = 1 écran (= 1 widget Flutter / 1 View SwiftUI / 1 composable Compose)
│   │   └── …
│   └── …
│
├── Prototype.html            (optionnel) Vue navigable style Figma
├── GUIDELINES.md             Spec pour le dev qui implémente côté code
├── README.md                 Ce fichier
└── CLAUDE_SKILLS.md          Prompts prêts pour les skills ui-kit-*
\`\`\`

## Conventions HTML machine-readable

Le kit pose des attributs sémantiques qui permettent aux outils de code-gen de mapper les maquettes vers du code idiomatique sans deviner. Voir `GUIDELINES.md` pour le détail et le mapping framework par framework.

Attributs principaux : `data-asset`, `data-screen-label`, `data-os-chrome`, `data-uses`, `data-hint`, `data-nav-target`, `data-auto-advance`.

## Skills disponibles pour itérer

Voir `CLAUDE_SKILLS.md` — invoquer `ui-kit-editor` pour modifier, `ui-kit-prototype` pour générer la vue navigable, `ui-kit-brand-explorer` pour itérer sur le logo.
```

### `CLAUDE_SKILLS.md` — prompts prêts à l'emploi

```markdown
# Prompts pour reprendre avec Claude Code

Liste de prompts copy-paste pour itérer sur ce kit via les skills `ui-kit-*`.

## Modifier le kit (skill `ui-kit-editor`)

\`\`\`
# Ajouter une page à un flow existant
Ajoute la page <page-name> au flow <flow-name> dans ce kit, avec les états default / loading / empty / error.

# Ajouter une icône
Ajoute l'icône <name> du set Lucide à ce kit (galerie + fichier source).

# Ajouter un composant partagé
Ajoute un composant <Name> au kit (classes CSS dans components.css + mock dans UI Kit.html § Composants + utilisation dans <page>).

# Modifier un token
Change la couleur primary de <hex-actuel> à <hex-nouveau> dans ce kit, et propage à tous les flows.

# Auditer le kit
Audite ce kit selon les conventions sémantiques (data-os-chrome, data-uses, data-hint, data-nav-target). Liste les manques.

# Migrer un kit aux conventions
Migre ce kit aux conventions sémantiques v1.10 (data-os-chrome + data-uses + data-hint + data-nav-target + data-auto-advance).

# Localiser les fonts
Localise les fonts de ce kit (passer de @import Google Fonts CDN à des fichiers locaux dans ds/assets/fonts/).
\`\`\`

## Vue navigable (skill `ui-kit-prototype`)

\`\`\`
# Générer / regénérer le prototype
Génère le Prototype.html de ce kit.

# Avec un screen d'entrée spécifique
Génère le Prototype.html en partant de l'écran <flow>/<page>:<state>.
\`\`\`

Une fois généré, ouvrir `Prototype.html` dans un browser. Cliquer les CTA → navigation animée. Touches : `←` `→` (prev/next), `B` (back), `S` (toggle sidebar), `Esc` (entry).

## Itération logo (skill `ui-kit-brand-explorer`)

\`\`\`
# Explorer 6 directions logo
Propose 6 directions de logo pour ce kit basées sur le brief du README.

# Itérer sur la direction choisie
Itère sur la direction <N> avec les ajustements suivants : <couleurs / typo / échelle>.

# Une fois choisi
Génère les 7 fichiers brand finaux pour la direction <N>.
\`\`\`

## Handoff vers code (skill `ui-kit-to-code`)

\`\`\`
# Mapper un écran vers du code
Génère le code <framework> (Flutter / SwiftUI / Compose / React / Vue) pour la page flows/<NN-flow>/<page>.html en respectant les conventions du codebase <chemin/vers/repo>.
\`\`\`

## Brief du projet (à compléter)

[Court paragraphe de contexte : quel produit, quelle stack cible, quelles particularités. Sert de mémo pour les futurs invocations de skills.]
```

**Adaptations selon le projet** :
- Remplacer `[Nom du produit]` par le vrai nom partout
- Adapter le snippet stack dans `GUIDELINES.md` à la cible (ne garder que la stack pertinente, pas les 4)
- Préciser dans `CLAUDE_SKILLS.md` les noms de flows/pages réels du projet (pas `<page-name>`)
- Si le projet a un widget wrapper spécifique (`IconView`, `ImageView`, `MediaPicker`…), le mentionner dans `GUIDELINES.md` § Conventions sémantiques

## ✅ Workflow de création

L'ordre est strict — le DS doit exister **avant** que le premier Flow soit écrit.

1. **Poser les 10 questions** (ou valider les infos fournies)
2. **Proposer 3 directions visuelles** en 2 phrases chacune (ton, palette, typo) — faire choisir
3. **Créer `ds/tokens.css`** complet (couleurs, type, spacing, radii, shadows, motion). Pour les fonts : déclarer les variables (`--font-display`, `--font-body`) et l'échelle, MAIS pas encore les `@font-face` — ils sont écrits à l'étape 3.5 après le téléchargement des fichiers.

3.5. **Télécharger les fonts dans `ds/assets/fonts/<font>/` (woff2 + ttf)** — étape obligatoire. Le mockup HTML utilise woff2 (léger), Flutter / iOS native / Android native exigent du ttf. Ship les **deux formats** depuis la source officielle, **jamais** `@import` Google Fonts CDN dans un kit livré (force dev à dépendre de la CDN, casse offline, Flutter ne lit pas le woff2).

   **Source recommandée — Google Fonts GitHub repo + Fontsource CDN** :

   ```bash
   # Variable fonts (1 fichier couvre toutes les graisses)
   mkdir -p ds/assets/fonts/<font-name>

   # WOFF2 via Fontsource (web mockup, léger)
   curl -L -o ds/assets/fonts/<font-name>/<font-name>-variable.woff2 \
     "https://cdn.jsdelivr.net/fontsource/fonts/<font-name>:vf@latest/latin-wght-normal.woff2"

   # TTF via Google Fonts repo (Flutter / iOS / Android — format natif requis)
   # ⚠️ --globoff pour préserver les [brackets] de l'URL des variable fonts
   curl -L --globoff -o ds/assets/fonts/<font-name>/<font-name>-variable.ttf \
     "https://raw.githubusercontent.com/google/fonts/main/ofl/<font-name>/<Font>[axes].ttf"
   ```

   **Alternative tout-en-un — google-webfonts-helper** (zip avec les deux formats) :
   ```bash
   curl -L -o /tmp/<font-name>.zip \
     "https://gwfh.mranftl.com/api/fonts/<font-name>?download=zip&subsets=latin&variants=400,500,600,700&formats=woff2,ttf"
   unzip /tmp/<font-name>.zip -d ds/assets/fonts/<font-name>/
   ```

   **Pour fonts statiques (non-variable)** : un fichier `.woff2` + un `.ttf` par graisse, naming `<font>-<weight>.woff2` / `.ttf`.

   **Pour fonts custom / payantes / SF Pro / etc.** : l'utilisateur fournit les fichiers. Le skill détecte la présence dans `ds/assets/fonts/`, génère les `@font-face` correspondants. Si fonts manquantes : fallback sur stack système (`-apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`).

   **Ajouter les `@font-face` en tête de `tokens.css`** (servir woff2 au browser, ttf en fallback que le browser ignorera mais que Flutter lira) :
   ```css
   @font-face {
     font-family: '<Font>';
     font-style: normal;
     font-weight: 400 700;          /* range pour variable font */
     font-display: swap;
     src: url('assets/fonts/<font-name>/<font-name>-variable.woff2') format('woff2-variations'),
          url('assets/fonts/<font-name>/<font-name>-variable.ttf')   format('truetype-variations');
   }
   ```
   Chemin relatif à `tokens.css` qui vit dans `ds/`, donc `assets/fonts/...` (pas `ds/assets/...`).

4. **Fetch les icônes** depuis la source officielle de la famille choisie (Lucide via raw.githubusercontent.com/lucide-icons/lucide/main/icons/<name>.svg) → sauvegarder verbatim dans `ds/assets/icons/<name>.svg` (un fichier par icône)
5. **Construire le sprite consolidé source** `ds/assets/icons/icons.svg` (tous les SVG regroupés dans un fichier de référence — pas inliné dans les HTML)
6. **Récupérer / créer les assets non-icônes** dans `ds/assets/` :
   - `ds/assets/brand/` : si le user fournit un logo, le placer ; sinon **poser un placeholder SVG simple** (lettermark texte des initiales en `--font-display`, sur fond `--primary`). Idem app-icon, splash si applicable. **Ne pas chercher à itérer plusieurs directions ici — c'est le rôle du skill `ui-kit-brand-explorer`** invocable plus tard si l'utilisateur veut itérer sur le logo. Le placeholder doit être propre (validé `xmllint --noout`) mais générique.
   - `ds/assets/illustrations/` : créer (ou demander) les scènes nécessaires aux écrans prioritaires (empty states notamment)
   - `ds/assets/images/` : si le user fournit des photos/hero, les placer
   - `ds/assets/fonts/` : si typo custom
   - Dossiers optionnels — créer à la demande, ne pas créer vide

6.5. **Validation XML systématique de tous les SVG produits** (nouvelle étape obligatoire) :
   ```bash
   cd /path/to/project
   for F in ds/assets/icons/*.svg ds/assets/brand/*.svg ds/assets/brand/splash/*.svg ds/assets/illustrations/*.svg; do
     [ -e "$F" ] || continue
     xmllint --noout "$F" 2>&1 | grep -v "^$" || echo "✓ $F"
   done
   ```
   Tout SVG qui échoue à `xmllint --noout` ne se chargera **PAS** via `<img>` (le browser refuse les SVG mal formés en chargement externe — alors que le HTML est tolérant, ce qui rend le bug invisible jusqu'au test visuel). Causes les plus fréquentes :
   - `&` brut au lieu de `&amp;` dans `aria-label`, `<title>`, `<desc>`, attributs
   - `<` brut au lieu de `&lt;` dans du texte
   - Attribut sans guillemets
   - Namespace XML manquant (`xmlns="http://www.w3.org/2000/svg"`)
   - Tag non fermé

   **Aucun SVG ne doit être livré sans avoir passé cette validation.** Si `xmllint` n'est pas installé, l'installer (`brew install libxml2`) ou utiliser `python3 -c "import xml.etree.ElementTree as ET; ET.parse('file.svg')"` comme alternative.

7. **Créer `ds/components.css`** — classes de tous les widgets shared (boutons, inputs, cards, list rows, dialogs, sheets, banners, alerts, empty states…)
8. **Créer `UI Kit.html`** — page unique : header + nav sticky + sections (Couleurs, Typographie, Spacing, Radii, Élévations, Motion, Brand si non vide, Iconographie, Illustrations si non vide, Images si non vide, Composants, iOS Chrome si applicable). Chaque icône rendue dans la galerie est **inlinée en plein SVG avec son `data-asset="ds/assets/icons/<name>.svg"`**. **Aucun `<pre><code>`.** Pas de sprite à inliner — chaque icône au point d'usage.
9. **Vérifier visuellement `UI Kit.html`** via `chrome-devtools` MCP : `new_page` sur l'URL `file://` → `list_console_messages` (doit être clean) → `take_screenshot` `fullPage:true` → **lire le screenshot** pour vérifier que toutes les icônes rendent (pas de cellules vides dans la galerie), que les assets brand/illustrations s'affichent, que les composants ne sont pas cassés. **Étape obligatoire avant de passer à la suite.**
10. **Créer le premier flow** : `flows/01-<nom>/<page>.html` — 1 page représentative avec ses 3-5 cellules d'états. Consommer uniquement `ds/`. Inliner les icônes en SVG complet avec `data-asset`. Référencer brand/illustrations/images via `<img src="../../ds/assets/...">` (chemin relatif depuis `flows/<flow>/`). Aucun style de widget en local.
11. **Vérifier visuellement la page du flow** via `chrome-devtools` MCP (même protocole : new_page → console → screenshot → lecture).
12. **Demander validation utilisateur** avant de multiplier les pages/flows. Si la page ne rend pas correctement, **ne pas livrer** — investiguer.
13. **Créer les autres pages du flow + autres flows** une fois direction validée. Une page = un fichier `flows/<flow>/<page>.html`. Si un flow a besoin d'un widget/icône/illustration absent : retour à l'étape 4, 6 ou 7, jamais d'ajout local.
14. **Écrire `GUIDELINES.md` + `README.md` + `CLAUDE_SKILLS.md`** en suivant les templates de la section "📚 Templates des 3 fichiers de documentation". Adapter le contenu au projet (nom, stack cible, vrais flows/pages), ne pas livrer les templates bruts.
15. **Auto-audit final** :
    - Toutes les pages flow sont sous `flows/<flow>/<page>.html` (aucun `Flow NN.html` à la racine)
    - Tous les assets sont sous `ds/assets/` (aucun fichier asset à la racine de `ds/`)
    - grep `<style>` dans les Flows → ne doit contenir que de la grille canvas/frame
    - grep des hex hors `tokens.css` → 0 (sauf chrome iOS bezel)
    - grep `<svg` sans `data-asset` dans les HTML → 0 hors descendants d'un `data-os-chrome` (le `<svg>` peut être chrome système, son ancestor doit être marqué)
    - grep `<use href="ds/` → 0 (tout sprite externe banni)
    - **Pour chaque élément OS-chrome présent** (status-bar, dynamic-island, home-indicator, keyboard, gesture-bar) : son conteneur porte `data-os-chrome="<type>"`. Les SVG enfants de ces conteneurs ne doivent **JAMAIS** porter `data-asset` (ils ne sont pas des assets app)
    - **Pour chaque pattern UI au comportement non-évident** (carousel, dialog, bottom-sheet, dot-indicator, tab-bar, etc.) : un `data-uses="ui:..."` est posé. Sinon le générateur produit une UI custom incorrecte
    - **Pour chaque fonction native sollicitée** (image-picker, camera, biometric, etc.) : un `data-uses="native:..."` est posé. Documenter le paramétrage en `data-uses-context` si non-trivial
    - **Pour chaque ambiguïté métier** (galerie BDD vs native, swipe behavior, etc.) : un `data-hint="..."` est posé
    - **Pour chaque CTA / row / lien qui navigue** entre écrans : un `data-nav-target="<flow>/<page>[:<state>]"` est posé. Pour les boutons retour : `data-nav-target="back" data-nav-back="true"`. Vérifier que toutes les cibles existent (grep `data-nav-target` puis valider chaque target contre l'inventaire `flows/<flow>/<page>.html` × `data-screen-label`).
    - `xmllint --noout` sur **tous** les SVG de `ds/assets/` → 0 erreur (pour confirmer que rien n'a été ajouté qui casserait le rendu via `<img>`)
    - **screenshot viewport-by-viewport** de chaque page via chrome-devtools (cf. § Validation visuelle), relire chaque section, vérifier dark mode si supporté
16. **Livrer le package complet** avec un récap des screenshots vérifiés

## 🔍 Validation visuelle — protocole obligatoire

Avant toute livraison ou avant de dire "c'est fait", utiliser `chrome-devtools` MCP avec la méthode **viewport-by-viewport** (pas fullPage). Les pages UI Kit denses font 8 000–15 000 px de haut, et un screenshot fullPage à cette taille dépasse les capacités de lecture du modèle (×2 sur Retina = 30 000 px). Splitter en chunks via PIL crée de la confusion (faux positifs de duplication).

### Méthode — pages longues (UI Kit, flows à plusieurs cellules)

```
1. mcp__chrome-devtools__new_page  url: file:///path/to/UI%20Kit.html

2. mcp__chrome-devtools__list_console_messages  types: ["error", "warn"]
   → DOIT être vide. Si erreurs, ne pas continuer — investiguer.

3. mcp__chrome-devtools__evaluate_script
   () => {
     const sections = Array.from(document.querySelectorAll('section.doc-section, .cell, .showcase'))
       .map(el => ({
         id: el.id || el.dataset.screenLabel
           || el.querySelector('.showcase__title, .cell-head .name')?.textContent,
         top: el.offsetTop,
         height: el.offsetHeight
       }));
     return { bodyHeight: document.body.scrollHeight, viewport: window.innerHeight, sections };
   }
   → récupère la liste de TOUTES les sections + leur position Y.

4. POUR CHAQUE SECTION (pas juste début/milieu/fin) :
   a. mcp__chrome-devtools__evaluate_script
      () => { window.scrollTo({top: <SECTION_TOP - 20>, behavior: 'instant'});
              return new Promise(r => setTimeout(() => r(window.scrollY), 100)); }
      ⚠️ behavior: 'instant' OBLIGATOIRE — sinon le scroll smooth (souvent activé via tokens.css)
         renvoie un scrollY incorrect et le screenshot capture la mauvaise zone.

   b. mcp__chrome-devtools__take_screenshot  filePath: /tmp/<page>-<section>.png
      ⚠️ NE PAS utiliser fullPage: true — capture viewport uniquement.

   c. Read /tmp/<page>-<section>.png  → vérifier visuellement.

5. Tester aussi le DARK MODE (si le DS le supporte) :
   () => { document.documentElement.setAttribute('data-theme', 'dark');
           window.scrollTo({top: 0, behavior: 'instant'}); return 'dark'; }
   → puis re-screenshot des sections clés (header avec logo, components principaux, brand).
```

### Règle dure : couvrir TOUTES les sections, pas un échantillon

Échec déjà observé : un logo cassé dans la section Brand n'a pas été détecté car le screenshot couvrait début / milieu / fin / dark — sautant précisément la section Brand entre. Conséquence : bug livré, vu uniquement parce que l'utilisateur l'a signalé.

**Inventaire à screenshoter pour un UI Kit** : Header, Couleurs, Typo, Spacing, Radii, Shadows, Motion, **Brand** (split en sous-sections : Logo / App icon / Splash si présent), **Iconographie** (vérifier qu'aucune cellule de la galerie n'est vide), Illustrations, Images, Composants (sous-section par famille : Boutons, Inputs, Badges, Avatars, Cards, Lists, Tabs, Alerts, AppBar, Dialogs, patterns produit-spécifiques, Empty state, Activity), iOS/Android Chrome.

Pour chaque flow `flows/<flow>/<page>.html` : screenshoter chaque cellule du canvas (default / loading / empty / error / etc.). La taille permet généralement 2 cellules par viewport, donc 2-3 screenshots par page.

### Méthode — pages Flow avec phones côte-à-côte (3-colonne canvas)

⚠️ **Le piège du zoom** : un Flow contient typiquement 3-5 phones de 360×720 px en grille. Un screenshot capture l'ensemble (~1100×900 CSS px = 2200×1800 raw retina), mais quand le modèle lit le PNG, **chaque phone n'occupe que ~250-300 px** → les détails fins (texte bouton qui déborde, progress bar mal centrée, mot dupliqué dans une liste, doublon créé par un overlay foireux) deviennent invisibles. Tu vas valider faussement et louper des bugs visuels que l'utilisateur verra à taille humaine.

**La règle** : un phone = un screenshot lisible. Crop la grille en colonnes pour que chaque phone soit lu individuellement.

```python
from PIL import Image
img = Image.open('/tmp/flow-fullpage.png')
w, h = img.size
n_cells = 3   # nombre de phones dans le canvas
col_w = w // n_cells
for i, name in enumerate(['default', 'loading', 'empty']):
    img.crop((i * col_w, 0, (i+1) * col_w, h)).save(f'/tmp/cell-{i+1}-{name}.png')
```

Puis lire chaque `cell-N-NAME.png` séparément. À cette échelle (1 phone par image, ~1100 px de large) les défauts visuels deviennent visibles :
- Texte qui déborde d'un bouton (`white-space: nowrap` du `.btn` clippe à droite)
- Progress bar / éléments mal centrés (parent sans `align-items: center` ou `margin: 0 auto`)
- Z-index foireux qui laisse passer un texte sous un overlay (doublon visuel)
- Espacement inconsistant entre cellules
- Status bar avec mauvaise couleur (light text sur light bg)

### Checklist obligatoire après chaque crop par-cellule

À chaque cellule lue, **scanner activement** (ne pas se contenter de "ça a l'air bien") :

1. **Centrage** : titre, progress, illustration, CTA — chaque élément flex/block est-il bien aligné horizontalement ? (parent flex sans `justify-content: center` = piège classique)
2. **Overflow texte** : tout `.btn`, `.badge`, `.tag` avec `white-space: nowrap` — le label tient-il dans la largeur disponible ? Sur un phone 360 px de large, un `.btn--block` ne dépasse pas ~328 px utiles.
3. **Z-index sandwich** : si overlay + bubble + contenu z-indexé, vérifier qu'aucun élément du contenu (badge orange, tag, mot souligné) ne traverse l'overlay et crée un doublon visuel avec ce qui est dans la bulle.
4. **Auto-scroll parasite** : un container avec `overflow-y: auto` peut être scrollé par le browser au chargement (focus auto sur un élément focusable en bas) → le contenu en haut (manuscript, hero) disparaît. Pour éviter : split en `flex-direction: column` avec un enfant `flex-shrink: 0` (header fixe) + un enfant `flex: 1; overflow-y: auto` (corps scrollable).
5. **Variables CSS undefined** : un `var(--space-14)` non déclaré rend la propriété entière invalide → padding/margin = 0 sans erreur visible. Toujours diff les usages contre `tokens.css` :
   ```bash
   diff <(grep -oE 'var\(--space-[0-9]+\)' ds/components.css | sort -u) \
        <(grep -oE '^\s*--space-[0-9]+' ds/tokens.css | tr -d ' :' | awk -F'--' '{print "var(--"$2")"}' | sort -u)
   ```

### Bugs récurrents à détecter au screenshot

- **Image broken icon dans la galerie Brand** → SVG XML invalide. Causes fréquentes : `&` brut dans `aria-label` ou `<title>` (doit être `&amp;`), `<` brut, attribut sans guillemets. Validation systématique via `xmllint --noout file.svg` (cf. workflow étape 6.5) doit retourner zéro erreur. Le browser refuse les SVG mal formés en chargement externe alors que le HTML est tolérant — c'est ce qui rend le bug invisible avant le test visuel.
- **Logo light affiché sur header dark** → `<img>` ne propage pas `currentColor` aux SVG externes. Solution : dual-img + CSS swap : `<img class="logo-light">` + `<img class="logo-dark">` avec `[data-theme="dark"] .logo-light { display: none; } [data-theme="dark"] .logo-dark { display: block; }`.
- **Toutes les icônes vides / gros SVG cassé** → reste d'un sprite externe `<use href="ds/...">` qui ne fonctionne pas en `file://`. Remplacer par SVG inline + `data-asset`.
- **Une icône isolée vide** → `<svg>` sans `<path>` ou viewBox faux. Vérifier que le SVG a été copié en entier depuis le fichier source.
- **Image / illustration cassée dans Flow** → mauvais chemin relatif depuis `flows/<flow>/<page>.html`. Le bon préfixe est `../../ds/assets/...` (3 niveaux : flows/ + dossier flow + page).
- **Texte de bouton qui déborde** : sur mobile (phone 360 px), un `.btn--block` avec icône + label long déborde silencieusement. Tester chaque CTA en cropping le phone individuellement. Si débordement : raccourcir le label ("Sauvegarder" plutôt que "Sauvegarder dans ma bibliothèque"), retirer l'icône, ou ajouter `white-space: normal; line-height: 1.2` au bouton.
- **Élément non centré dans son parent** : progress bar, illustration, dots/percentage. Si le parent est un `<div>` sans `display: flex; align-items: center`, les enfants block s'alignent à gauche par défaut. Toujours wrapper dans un container avec centrage explicite.
- **`overflow-y: auto` qui scroll seul** : le browser peut scroller un container automatiquement (focus, anchor, layout shift). Pour un mockup statique, préférer un layout split : header fixe + body scrollable, plutôt qu'un seul container avec overflow auto.
- **Overlay qui ne couvre pas tout** : un overlay positionné en absolute peut laisser passer des éléments qui ont un `z-index` plus élevé. Avant d'ajouter z-index sur un élément du contenu, vérifier que ça ne crée pas de doublon visuel avec un élément flottant (bulle, popover, sheet).

### Fallback si chrome-devtools MCP indisponible

- Tenter `kill` sur le PID du chrome-devtools-mcp + supprimer le `SingletonLock` dans `~/.cache/chrome-devtools-mcp/chrome-profile/`
- En dernier recours : `open file://...` via Bash + **demander au user de confirmer visuellement**, ne jamais livrer sans confirmation

## 📤 Livrable final

Package HTML autonome, zippable, contenant :
- **Tokens** sémantiquement nommés (`ds/tokens.css`)
- **Composants partagés** (`ds/components.css`)
- **Dossier d'icônes** (`ds/assets/icons/<name>.svg` + `icons.svg` sprite consolidé source)
- **Dossier d'identité de marque** (`ds/assets/brand/` : logos, app-icon, favicon, splash) — selon scope
- **Dossier d'illustrations** (`ds/assets/illustrations/*.svg` : scènes vectorielles) — selon scope
- **Dossier d'images raster** (`ds/assets/images/` : photos, placeholders, hero) — selon scope
- **Fonts** custom (`ds/assets/fonts/`) — si typo non-system
- **UI Kit page unique** (`UI Kit.html`) avec tokens + brand + iconographie + illustrations + images + composants en rendu visuel. Icônes inlinées en plein SVG avec `data-asset`.
- **Dossier `flows/`** organisé par flow : `flows/<flow>/<page>.html` — chaque page composée uniquement à partir du DS, icônes inlinées avec `data-asset`, `data-nav-target` posés sur les CTA navigants pour traçabilité par les outils de code-gen + le prototype interactif
- **Documentation** pour modification future (GUIDELINES.md, README.md)
- **Prompts** pour reprendre avec IA (CLAUDE_SKILLS.md)

Ouvrable dans n'importe quel navigateur sans build, **y compris en `file://`**. Auto-cohérent : modifier un token ou un widget propage automatiquement à tous les flows qui le consomment. Modifier une icône dans `ds/assets/icons/<name>.svg` nécessite de re-propager le SVG inline dans les pages qui la consomment (find/replace par `data-asset`).

### Étapes optionnelles post-création

- **Prototype interactif** : invoquer `ui-kit-prototype` pour générer `Prototype.html` à la racine du kit — vue navigable style Figma qui parcourt les écrans via les `data-nav-target`. Pratique pour présentation user/stakeholder, validation de parcours, pré-handoff dev.
- **Itération sur le logo** : invoquer `ui-kit-brand-explorer` pour proposer plusieurs directions de logo en SVG, choisir, et regénérer les 7 fichiers brand cohérents (`logo.svg`, `logo-mark.svg`, `logo-monochrome.svg`, `logo-dark.svg`, `app-icon.svg`, `splash.svg`, `favicon.svg`). À utiliser quand le logo posé en placeholder à l'étape 6 du workflow ne convient pas.
