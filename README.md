# UI Kit Skills

**Version 1.12** — Skills pour [Claude Code](https://claude.com/claude-code), inspiré de [claude.ai/design](https://claude.ai/design).

| Skill | Usage |
|---|---|
| [`ui-kit-creator`](ui-kit-creator/SKILL.md) | Créer un UI kit HTML depuis zéro (DS + composants + flows + assets + 3 fichiers de doc templatés : GUIDELINES.md, README.md, CLAUDE_SKILLS.md) |
| [`ui-kit-editor`](ui-kit-editor/SKILL.md) | Modifier, étendre ou auditer un kit existant |
| [`ui-kit-prototype`](ui-kit-prototype/SKILL.md) | Générer un prototype interactif style Figma à partir d'un kit (clics → navigation, transitions temporisées, sidebar, animations) |
| [`ui-kit-brand-explorer`](ui-kit-brand-explorer/SKILL.md) | Explorer plusieurs directions logo (6 directions × 4 variantes), choisir, générer les 7 fichiers brand cohérents — neutre côté esthétique (suit le DS et le ton du brief, dark-first / sobre / chaleureux / technique selon le projet) |
| [`ui-kit-to-code`](ui-kit-to-code/SKILL.md) | Transformer une maquette HTML en code (React, Vue, Flutter, SwiftUI, Compose…) en lisant les conventions sémantiques pour générer du code idiomatique au lieu d'un mockup mort |

Ces skills produisent des kits **machine-readable** : conventions explicites sur les attributs HTML (`data-asset`, `data-os-chrome`, `data-uses`, `data-hint`, `data-screen-label`, `data-nav-target`, `data-auto-advance`) qui permettent à n'importe quel outil de code-gen (scripts custom, agents IA) de mapper les maquettes vers du code mobile sans deviner.

## Installation

```bash
git clone https://github.com/kftof/ui-kit.git
cp -r ui-kit/ui-kit-* ~/.claude/skills/
```

Claude détecte les skills automatiquement.

## Arborescence d'un kit généré

```
[projet]/
│
├── ds/
│   ├── tokens.css                ← variables CSS (:root + reset)
│   ├── components.css            ← classes widgets
│   │
│   └── assets/                   ← TOUS LES ASSETS, regroupés par famille
│       │
│       ├── icons/                ← ICÔNES UI (vectorielles, monochrome ≤24px)
│       │   ├── icons.svg         ← sprite consolidé source (référence designer/dev)
│       │   ├── plus.svg          ← un fichier par icône, contenu Lucide canonique
│       │   ├── check.svg
│       │   └── …
│       │
│       ├── brand/                ← IDENTITÉ DE MARQUE
│       │   ├── logo.svg
│       │   ├── logo-mark.svg
│       │   ├── logo-monochrome.svg
│       │   ├── logo-dark.svg
│       │   ├── app-icon.svg      ← source 1024×1024
│       │   ├── favicon.svg
│       │   └── splash/
│       │       ├── splash.svg    ← composition source vectorielle
│       │       ├── splash@2x.png ← exports raster device (optionnel)
│       │       └── splash@3x.png
│       │
│       ├── illustrations/        ← SCÈNES SVG (multi-couleur, 80-400px)
│       │   ├── empty-tasks.svg
│       │   ├── onboarding-1.svg
│       │   ├── error-404.svg
│       │   └── decoration-blob.svg
│       │
│       ├── images/               ← MATIÈRES RASTER (PNG / JPG / WebP)
│       │   ├── hero.jpg
│       │   ├── placeholder-avatar.png
│       │   └── pattern-bg@2x.png
│       │
│       └── fonts/                ← typo custom (.woff2)
│
├── UI Kit.html                   ← PAGE UNIQUE — sections rendu visuel :
│                                    Couleurs / Typo / Spacing / Radii / Élévations /
│                                    Motion / Brand / Iconographie / Illustrations /
│                                    Images / Composants / iOS Chrome
│                                    Icônes inlinées en plein SVG avec data-asset
│
├── flows/                        ← FLOWS UTILISATEUR
│   ├── 01-tasks/                 ← un dossier par flow
│   │   ├── list.html             ← une page HTML par écran
│   │   ├── detail.html              (chaque page contient ses cellules d'états)
│   │   └── settings.html
│   ├── 02-onboarding/
│   │   └── …
│   └── …
│
├── GUIDELINES.md
├── README.md
└── CLAUDE_SKILLS.md
```

**Principes** :
- `ds/` = source de vérité unique (tokens, composants, **tous les assets sous `ds/assets/`**)
- `UI Kit.html` = **page unique** de doc visuelle (pas de markup HTML apparent)
- `flows/<flow>/<page>.html` = un dossier par flow, un fichier par écran/page (1 fichier = 1 widget Flutter / 1 View SwiftUI / 1 composable Compose côté code)
- 5 familles d'assets distinctes sous `ds/assets/` : `icons/`, `brand/`, `illustrations/`, `images/`, `fonts/`

**Convention de nommage flows / pages / cells** (machine-readable, pour outils de code-gen) :

| Niveau | Format | Exemple | ID dérivé |
|---|---|---|---|
| Flow | `flows/NN-<id>/` | `flows/01-tasks/` | `flow_id = "tasks"` (strip `/^\d+-/`) |
| Page | `<id>.html` (ou `NN-<id>.html` si ordre voulu) | `list.html` | `page_id = "list"` |
| État (cell) | `data-screen-label="NN <Page> <state>"` | `"01 List default"` | `state_id = "default"` (dernier mot, lower) |
| Screen global | `<flow_id>/<page_id>[:<state_id>]` | `tasks/list:loading` | identifiant stable |

Préfixe `NN-` obligatoire sur les dossiers flow (séquencement, dépendances). Optionnel sur les pages (uniquement si ordre voulu, ex: onboarding séquentiel).

**Convention `data-asset` pour les icônes** :
```html
<svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor"
     stroke-width="2" stroke-linecap="round" stroke-linejoin="round"
     data-asset="ds/assets/icons/plus.svg">
  <path d="M5 12h14"/>
  <path d="M12 5v14"/>
</svg>
```

Chaque icône est **inlinée en plein SVG** (rendu autonome en `file://`) avec un attribut `data-asset` qui pointe vers le fichier source. Cet attribut est la **clé de matching** pour les outils de code-gen : `grep data-asset` dans la maquette → liste exacte des assets à importer dans le projet mobile.

Les autres assets (brand / illustrations / images) utilisent `<img src="ds/assets/...">` ou `background-image: url(...)` — le chemin joue le même rôle de matching.

**Conventions sémantiques** pour donner du contexte à l'agent codeur :

| Attribut | Quoi |
|---|---|
| `data-os-chrome="<type>"` | Élément rendu par l'OS (status-bar, dynamic-island, home-indicator, keyboard, gesture-bar, etc.). **Héritage parent** : tous les descendants sont skipés automatiquement. L'agent ne génère pas de widget pour ces éléments — la `SafeArea` du framework s'en charge. |
| `data-uses="native:<feature>"` | Fonction native du device (`native:image-picker`, `native:camera`, `native:date-picker`, `native:biometric-auth`, `native:share-sheet`, etc.). L'agent priorise un widget partagé du projet, fallback sur l'API native si rien d'existant — sémantique pure, non prescriptive d'implémentation. |
| `data-uses="ui:<pattern>"` | Pattern UI au comportement non-évident (`ui:carousel`, `ui:dialog`, `ui:bottom-sheet`, `ui:dot-indicator`, `ui:swipe-actions`, `ui:pull-to-refresh`, etc.). Sans cet attribut, l'agent recode en custom au lieu d'utiliser la primitive framework. |
| `data-uses-context="..."` | Texte libre paramétrant `data-uses` (multi-select, max items, options spécifiques). |
| `data-hint="..."` | Hint sémantique métier libre (galerie BDD vs native, swipe behavior, logique non visible). |
| `data-nav-target="<flow>/<page>[:<state>]"` | CTA / row / lien qui navigue vers un autre écran. `+ data-nav-back="true"` pour le retour. Consommé par le prototype interactif (clics navigables) ET par les outils de code-gen (génération directe des routes). |
| `data-auto-advance="<flow>/<page>[:<state>]"` + `data-auto-advance-delay="<ms>"` | Posés sur la cell `[data-screen-label]`, signalent une navigation **temporisée** (splash → home, processing → result, toast auto-dismiss). Délai par défaut 3000 ms. Consommé par le prototype (Timer + barre de progress + bouton stop) ET par les outils de code-gen (Timer/LaunchedEffect/.task avec cancel-on-dispose). |

Ces conventions permettent à un script `grep` ou un agent LLM de comprendre la **sémantique** des éléments (pas seulement leur visuel), ce qui change radicalement la qualité du code généré.

## Prototype interactif (style Figma)

Une fois ton kit créé et tes `data-nav-target` posés sur les CTA navigants, invoque `ui-kit-prototype` pour générer un fichier `Prototype.html` à la racine du kit. C'est une vue navigable autonome :

- Un seul écran s'affiche à la fois en grand (pas la grille canvas)
- Sidebar avec tous les écrans groupés par flow, click pour naviguer directement
- Clics sur les `[data-nav-target]` du screen courant → navigation animée (slide entre flows/pages, fade entre états)
- Breadcrumb `flow › page › état`, prev/next, raccourcis clavier (`← → B S Esc`)
- Mode présentation (sidebar masquable)

Ouvrable directement en `file://`, autonome (vanilla JS, pas de build). Partageable en zip à un stakeholder pour démo.

Les sous-dossiers `brand/`, `illustrations/`, `images/`, `fonts/` sont **optionnels** — créés à la demande selon le scope du projet.
