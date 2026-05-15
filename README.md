# UI Kit Skills

**Version 1.16** — Skills pour [Claude Code](https://claude.com/claude-code), inspiré de [claude.ai/design](https://claude.ai/design).

> **Nouveau dans 1.16** : (1) lecture du PRD obligatoire en pré-flight (anti-hallucination), (2) anti-monoculture palette + section Wording/Microcopy dans `ui-kit-creator` (bannit le défaut paresseux "terracotta + warm neutral + serif vivante" et le jargon tech / mots vides), (3) **audit auto post-création/édition** (4 checks : HTML bien formé via tidy/xmllint, composants partagés, `UI Kit.html` sync, pages obligatoires Settings/About/Legal/Suggestions). Voir [Audit auto](#audit-auto-post-créationédition).

| Skill | Usage |
|---|---|
| [`ui-kit-creator`](ui-kit-creator/SKILL.md) | Créer un UI kit HTML depuis zéro (DS + composants + flows + assets + 3 fichiers de doc templatés). Inclut Accessibilité WCAG 2.1 AA (`data-a11y-*`), catalogue de 15 styles UI nommés, scan anti-monoculture palette à l'étape 2, section Wording/Microcopy avec 8 règles et 4 grep d'audit. |
| [`ui-kit-editor`](ui-kit-editor/SKILL.md) | Modifier, étendre ou auditer un kit existant — recette dédiée pour fixer les fails d'un `Audit.md` + audit auto post-édition (intégrité structurelle). |
| [`ui-kit-prototype`](ui-kit-prototype/SKILL.md) | Générer un prototype interactif style Figma à partir d'un kit (clics → navigation, transitions temporisées, sidebar, animations) |
| [`ui-kit-brand-explorer`](ui-kit-brand-explorer/SKILL.md) | Explorer plusieurs directions logo (6 directions × 4 variantes), choisir, générer les 7 fichiers brand cohérents — neutre côté esthétique (suit le DS et le ton du brief, dark-first / sobre / chaleureux / technique selon le projet) |
| [`ui-kit-audit`](ui-kit-audit/SKILL.md) | Auditer un kit contre les 3 playbooks de stratégie + critères WCAG mobile (83 critères au total) et produire un rapport `Audit.md` avec score, critères pass/fail/n-a + justification, recommandations prioritaires ordonnées. Diagnostic uniquement — pour corriger, enchaîner avec `ui-kit-editor`. |
| [`ui-kit-aso-metadata`](ui-kit-aso-metadata/SKILL.md) | **NEW** — Générer les textes de fiche App Store / Play Store haute conversion (App Name, Subtitle, Promo Text, Description, Keywords, What's New) avec validation stricte des limites Apple/Google, competitive analysis 3-5 concurrents, storyboard captures et Custom Product Pages. Livrable `ASO.md` (humain) + `ASO.json` (machine-readable, consommable CI/CD). Complémentaire de `aso-appstore-screenshots` qui gère les visuels. |
| [`ui-kit-to-code`](ui-kit-to-code/SKILL.md) | Transformer une maquette HTML en code (React, Vue, Flutter, SwiftUI, Compose…) en lisant les conventions sémantiques pour générer du code idiomatique au lieu d'un mockup mort |

Ces skills produisent des kits **machine-readable** : conventions explicites sur les attributs HTML (`data-asset`, `data-os-chrome`, `data-uses`, `data-hint`, `data-screen-label`, `data-nav-target`, `data-auto-advance`, `data-api-call`) qui permettent à n'importe quel outil de code-gen (scripts custom, agents IA) de mapper les maquettes vers du code mobile sans deviner.

## Installation

```bash
git clone https://github.com/kftof/ui-kit.git

# 1) Copier les 7 skills
cp -r ui-kit/ui-kit-* ~/.claude/skills/

# 2) Copier le dossier playbooks (consommé par creator / editor / audit)
cp -r ui-kit/playbooks ~/.claude/skills/
```

Claude détecte les skills automatiquement. Les playbooks sont chargés à la demande par les skills concernés depuis `~/.claude/skills/playbooks/` (avec fallback sur le repo cloné si présent).

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

**Conventions sémantiques** pour donner du contexte à l'agent codeur. Liste exhaustive — tout outil de code-gen consommant ces kits peut s'aligner dessus.

### Identifiants & framing

| Attribut | Quoi |
|---|---|
| `data-screen-label="NN <Page> <state>"` | ID stable d'une cell (état d'un écran) — dernier mot = `state_id` (default, loading, empty, error, edit, success…). Permet aux outils de générer des routes / widgets nommés sans deviner. |
| `data-asset="ds/assets/icons/<name>.svg"` | Clé de matching entre un SVG inliné et son fichier source. `grep data-asset` → liste exacte des assets à importer dans le projet mobile. |
| `data-os-chrome="<type>"` | Élément rendu par l'OS (status-bar, dynamic-island, home-indicator, keyboard, gesture-bar, notch, camera-cutout, device-frame). **Héritage parent** : tous descendants skipés. L'agent ne génère pas de widget — la `SafeArea` du framework s'en charge. |
| `data-theme="dark"` | Posé sur `<html>` pour activer le mode sombre dans le mockup. Consommé par CSS via `[data-theme="dark"] { ... }`. |

### Capacités & patterns

| Attribut | Quoi |
|---|---|
| `data-uses="native:<feature>"` | Fonction native du device (`native:image-picker`, `native:camera`, `native:date-picker`, `native:biometric-auth`, `native:share-sheet`, `native:scanner`, `native:map`, `native:push-notification`, etc.). L'agent priorise un widget partagé du projet, fallback sur l'API native si rien d'existant — sémantique pure, non prescriptive d'implémentation. |
| `data-uses="ui:<pattern>"` | Pattern UI au comportement non-évident (`ui:carousel`, `ui:dialog`, `ui:bottom-sheet`, `ui:dot-indicator`, `ui:swipe-actions`, `ui:pull-to-refresh`, `ui:tab-bar`, `ui:stepper`, `ui:skeleton-loader`, etc.). Sans cet attribut, l'agent recode en custom au lieu d'utiliser la primitive framework. |
| `data-uses-context="..."` | Texte libre paramétrant `data-uses` (multi-select, max items, options spécifiques). |
| `data-hint="..."` | Hint sémantique métier libre (galerie BDD vs native, swipe behavior, logique non visible). Fallback quand aucun attribut typé ne convient. |

### Navigation

| Attribut | Quoi |
|---|---|
| `data-nav-target="<flow>/<page>[:<state>]"` | CTA / row / lien qui navigue vers un autre écran. `+ data-nav-back="true"` pour le retour. Consommé par le prototype interactif (clics navigables) ET par les outils de code-gen (génération directe des routes). |
| `data-auto-advance="<flow>/<page>[:<state>]"` + `data-auto-advance-delay="<ms>"` | Posés sur la cell `[data-screen-label]`, signalent une navigation **temporisée** (splash → home, processing → result, toast auto-dismiss). Délai par défaut 3000 ms. Consommé par le prototype (Timer + barre de progress + bouton stop) ET par les outils de code-gen (Timer/LaunchedEffect/.task avec cancel-on-dispose). |

### Backend & data

| Attribut | Quoi |
|---|---|
| `data-api-call="<METHOD>:/path[;<METHOD>:/path...]"` | Élément / conteneur qui consomme un endpoint backend. Posé sur le conteneur le plus haut (typiquement `<main>` ou `.phone__screen`) ou sur un bouton qui POST/PUT/DELETE au tap. **Héritage parent + set union enfants**. Skip sous `data-os-chrome`. Format strict : `METHOD` ∈ `GET\|POST\|PUT\|PATCH\|DELETE`, path identique à l'OpenAPI, path params en `{name}`, multiples séparés par `;`, **pas de query string**. Permet au code-gen de générer paresseusement la couche data uniquement pour les endpoints réellement consommés. |
| `data-runtime="<source>"` | Conteneur dont le contenu est **généré au runtime** (à NE PAS hardcoder dans la vue). `source` ∈ `user-generated` / `server-driven` / `algorithm-generated` / `local-storage`. Exemples : canvas d'un plan de table, feed social, galerie utilisateur, map de marqueurs, liste de suggestions, todos offline. |
| `data-runtime-shape="<type>"` | Compagnon de `data-runtime` — nom du modèle métier (`table`, `post`, `todo`, `marker`, `recipe`). Permet au code-gen de générer la data class. |
| `data-runtime-mock="true"` | Posé sur chaque enfant **mocké pour le visuel uniquement**. Le code-gen le saute (ne pas inclure dans le `build()`). Sans cette convention, les 6 tables fantômes du markup deviennent 6 widgets fixes au lieu d'une `List<Table>` vide + machinery d'ajout. |
| `data-runtime-seed="<n>"` (optionnel) | Combien d'items la maquette montre — signal au code-gen que c'est représentatif, pas exhaustif. |
| `data-runtime-source="<expr>"` (optionnel) | Source précise : `api:GET /events/{id}/tables` (couplé à `data-api-call`), `local:todos` (clé Hive/SQLite/SharedPreferences), `local-algorithm:recommend(user)`. |
| `data-runtime-empty="<flow>/<page>[:<state>]"` (optionnel) | Cell empty-state correspondante — affichée quand `items.isEmpty`. |

### Accessibilité

| Attribut | Quoi |
|---|---|
| `data-a11y-label="<texte>"` | Texte lu par le screen reader (équivalent `accessibilityLabel` mobile / `aria-label` web). Obligatoire sur les boutons iconiques sans texte visible. |
| `data-a11y-hint="<texte>"` | Indication supplémentaire (équivalent `accessibilityHint` iOS). |
| `data-a11y-hidden="true"` | Élément ignoré par les screen readers (images décoratives, séparateurs visuels). |
| `data-a11y-role="<role>"` | Rôle ARIA explicite (button, tab, header, alert, etc.) quand le markup natif ne le précise pas. |
| `data-a11y-live="polite\|assertive"` | Région annoncée dynamiquement (toasts, notifications). `polite` après actions courantes, `assertive` pour erreurs critiques. |

Ces conventions permettent à un script `grep` ou un agent LLM de comprendre la **sémantique** des éléments (pas seulement leur visuel), ce qui change radicalement la qualité du code généré.

## Audit auto post-création/édition

`ui-kit-creator` (étape 15) et `ui-kit-editor` (avant toute livraison) exécutent désormais 4 checks d'intégrité structurelle, indépendants des audits stratégiques de `ui-kit-audit` :

| Check | Vérifie | Outil |
|---|---|---|
| **A — Balises HTML** | Aucune balise manquante, orpheline ou mal fermée dans tous les `*.html` (invisible au browser mais casse `ui-kit-to-code` et autres outils de code-gen qui parsent en strict) | `tidy -errors -quiet` (fallback `xmllint --html --noout`) |
| **B — Composants partagés** | Aucun `<style>` de widget dans les flows, aucun inline style répété ≥ 2×, aucune classe ad-hoc sans pendant dans `ds/components.css` | `grep` |
| **C — UI Kit.html à jour** | Chaque widget consommé dans `flows/**/*.html` a sa cellule de doc dans `UI Kit.html` — sinon le kit ment et le dev code en aveugle | `grep` |
| **D — Pages obligatoires** | Présence de Settings, About, Legal (CGU + Privacy), Suggestions (accessible depuis Settings via `data-nav-target`). Exclusion possible si justifiée dans `GUIDELINES.md` | `find` + `grep` |

Tout fail bloque la livraison — corriger d'abord, ou documenter l'exclusion. Les commandes sont copiables/exécutables telles quelles dans chaque skill.

S'ajoutent les checks pré-existants : anti-monoculture palette (compare `--primary` du nouveau kit aux kits déjà shippés dans `~/Dev`), audit wording (4 grep mots vides / CTA génériques / empty states système-like / superlatifs absolus), validation `xmllint` sur tous les SVG.

## Playbooks de stratégie

Le repo inclut un dossier [`playbooks/`](playbooks/) avec 3 doctrines actionables (règles numérotées + critères d'audit machine-readable) que les skills consomment pour transformer un kit visuel en kit **stratégiquement bien conçu** — pas juste joli :

| Playbook | Sujet | Quand l'appliquer |
|---|---|---|
| [`01-onboarding-paywall.md`](playbooks/01-onboarding-paywall.md) | Tunnel de vente onboarding (3 actes : cadrage psychologique → climax engagement → transition paywall) + règles paywall (Weekly/Annual, trial, rappel J-1, preuve sociale, transparence) + **7 patterns 2026** (trial annual-only, soft-then-hard, Free vs Pro table embedded, post-close offer 24h, plan loader, PPP-adjusted pricing, AI personalization) | Kit avec flow onboarding ou écran paywall |
| [`02-engagement-virality.md`](playbooks/02-engagement-virality.md) | Core loop sans friction, sharable moments brandés, demande d'avis au pic émotionnel, progression / streaks, settings standardisés + **Hook Model** (Trigger / Action / Variable Reward / Investment + Fogg B=MAT) pour articuler la boucle d'habitude complète | Tout kit avec flow d'usage principal (≈ tous les kits mobile) |
| [`03-conversion-psychology.md`](playbooks/03-conversion-psychology.md) | Psychologie comportementale + stratégie visuelle marketing : Aha visuel, effet miroir, captures App Store axées bénéfice, sortie du "AI slope" via signe distinctif, mapping niche → palette/typo + **Peak-End Rule** (designer le pic + la fin de chaque flow) + **catalogue de 15 styles UI nommés** + **14 anti-patterns mobile** spécifiques | Tout kit qui vise une conversion App Store |

**Boucle d'usage typique** :
1. `ui-kit-creator` lit les playbooks pertinents au scope du kit avant d'écrire les écrans → kit **pass-par-construction**
2. `ui-kit-audit` évalue le kit a posteriori contre 49 critères binaires (18 + 17 + 14) et produit un rapport `Audit.md` à la racine
3. `ui-kit-editor` applique les fixes prioritaires identifiés (recette dédiée "Fixer les fails d'un Audit.md")
4. Re-audit pour mesurer la progression du score entre versions (diff git de `Audit.md`)

Chaque playbook se termine par une table machine-readable consommée mécaniquement par `ui-kit-audit`. Les règles évoluent indépendamment des skills (un fichier par playbook), faciles à étendre via PR.

## Prototype interactif (style Figma)

Une fois ton kit créé et tes `data-nav-target` posés sur les CTA navigants, invoque `ui-kit-prototype` pour générer un fichier `Prototype.html` à la racine du kit. C'est une vue navigable autonome :

- Un seul écran s'affiche à la fois en grand (pas la grille canvas)
- Sidebar avec tous les écrans groupés par flow, click pour naviguer directement
- Clics sur les `[data-nav-target]` du screen courant → navigation animée (slide entre flows/pages, fade entre états)
- Breadcrumb `flow › page › état`, prev/next, raccourcis clavier (`← → B S Esc`)
- Mode présentation (sidebar masquable)

Ouvrable directement en `file://`, autonome (vanilla JS, pas de build). Partageable en zip à un stakeholder pour démo.

Les sous-dossiers `brand/`, `illustrations/`, `images/`, `fonts/` sont **optionnels** — créés à la demande selon le scope du projet.
