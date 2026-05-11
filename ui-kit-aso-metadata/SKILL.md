---
name: ui-kit-aso-metadata
description: Génère les textes de fiche App Store / Play Store haute conversion (App Name, Subtitle, Promotional Text, Description, Keywords, What's New) à partir d'un UI kit + brief produit, avec validation stricte des limites de caractères Apple / Google, analyse compétitive 3-5 concurrents, et storyboard de captures d'écran cohérent avec le brand du kit. Produit un livrable `ASO.md` à la racine du kit + une variante `ASO.json` consommable par les pipelines de release. Complémentaire du skill `aso-appstore-screenshots` (qui génère les visuels) — ici on génère les textes.
---

# UI Kit ASO Metadata — fiche store haute conversion

Génère les **textes** de fiche App Store + Play Store optimisés pour le référencement (ASO) et la conversion, à partir du UI kit du produit + d'un brief court. Le visuel des captures d'écran reste géré par le skill complémentaire `aso-appstore-screenshots` (qui utilise Nano Banana Pro).

**Livrable principal** : `ASO.md` à la racine du kit (validation par champ, character counts, recommendations) + variante `ASO.json` consommable par les pipelines de release.

## 🎯 Quand utiliser ce skill

- Soumission initiale d'une app sur l'App Store / Play Store
- Refonte de la fiche store pour A/B test ASO
- Pré-release : valider que tous les wordings respectent les limites Apple / Google
- Custom Product Pages (CPP) : générer des variantes textuelles ciblées par mot-clé

## 📋 Inputs requis

**Obligatoire** :
1. **Path du kit** — racine d'un kit produit par `ui-kit-creator` (avec `ds/`, `flows/`, `UI Kit.html`, `GUIDELINES.md`, `README.md`)

**Recommandé** :
2. **Brief produit** (extrait automatiquement de `README.md` / `PRD.md` du kit si présent) :
   - Nom du produit
   - Catégorie App Store / Play Store cible
   - 3-5 features clés
   - Tonalité / niche

3. **Concurrents directs** (3-5 noms ou URLs App Store) — pour la compétitive analysis. Si absent, le skill propose une liste basée sur la catégorie / mots-clés inférés.

4. **Mots-clés primaires ciblés** (5-10 — extraits du brief si présent, sinon proposés)

## 📏 Limites de caractères Apple (strictes)

Validation obligatoire — toute soumission qui dépasse est rejetée par Apple.

| Champ | Maximum | Notes |
|---|---|---|
| **App Name** | 30 caractères | Visible partout (résultats search, page produit, home screen) |
| **Subtitle** | 30 caractères | Sous le nom — bénéfice court / value prop |
| **Promotional Text** | 170 caractères | Modifiable sans nouvelle review — pour annonces et changements ponctuels |
| **Description** | 4000 caractères | Texte long. Les 3 premières lignes sont visibles avant "more" — critiques |
| **Keywords** | 100 caractères | Cumulés, séparés par virgules, **pas d'espaces** après les virgules (économie de chars) |
| **What's New** | 4000 caractères | Patch notes — visible dans l'onglet "What's New" de la page |

## 📏 Limites Google Play (différentes — à respecter en parallèle)

| Champ | Maximum |
|---|---|
| **App Title** | 30 caractères |
| **Short Description** | 80 caractères |
| **Full Description** | 4000 caractères |
| **What's New / Release Notes** | 500 caractères |

⚠️ Pas de "keywords" séparés sur Play Store — l'algorithme indexe directement la full description. Bourrer des mots-clés explicites est pénalisé.

## 📋 Lecture du PRD — règle obligatoire (anti-hallucination du brief produit)

Avant de générer les wordings ASO, le skill **doit** :

1. **Chercher le PRD** : `<kit-root>/PRD.md` → `<kit-root>/PRD_*.md` → `<kit-root>/docs/PRD.md` → `<kit-root>/../PRD.md` (parent) → `<kit-root>/README.md`. Le PRD donne les **bénéfices réels** du produit qui doivent apparaître dans la fiche store — pas des bénéfices inventés.
2. **Si trouvé** → lire intégralement. Les features mentionnées dans la Description / Subtitle / Keywords doivent **toutes** être présentes dans le PRD. Aucune feature inventée pour faire joli.
3. **Si AUCUN PRD** → demander à l'utilisateur :

   > "Je n'ai pas trouvé de PRD pour ce projet. Sans PRD, je risque d'inventer des bénéfices qui ne sont pas dans le produit. **(a)** Tu déposes un `PRD.md` à la racine. **(b)** Tu confirmes qu'il n'y a pas de PRD — alors décris-moi en 3-5 bullets les features réelles et le tone of voice voulu. **(c)** Tu pointes un autre path."

4. **Cas (b)** : utiliser uniquement les features fournies par l'utilisateur. Ne rien inventer au-delà — mieux vaut une description courte mais juste qu'une description longue avec des promesses non tenues (qui font fuir + risque rejet App Store §6 du playbook 1 HON-1).

## 🧭 Workflow

### Phase 1 — Reconnaissance du kit

1. **Lire `README.md` + `PRD.md`** (si présent) du kit → extraire nom du produit, tagline, features clés
2. **Lire `GUIDELINES.md` § Anti-patterns et brand** → tonalité voulue (chaleureux / techy / pro / etc.)
3. **Lister les flows présents dans `flows/`** → confirme les features visibles dans l'app (un flow = une promesse à exprimer)
4. **Inférer la catégorie App Store** (Productivity / Health & Fitness / Lifestyle / Education / Photo & Video / etc.) depuis le contenu
5. **Charger les mots-clés primaires** (depuis brief utilisateur ou inférés)

### Phase 2 — Compétitive analysis (optionnel mais recommandé)

Pour 3-5 concurrents directs :

```markdown
### 🎯 Competitive Analysis

**Market Positioning**
- Identifier 3-5 concurrents directs (par mots-clés et catégorie)
- Documenter leur keyword focus
- Noter leur différenciation messaging

**Keyword Gap Analysis**
- Mots-clés à fort volume sur lesquels les concurrents rankent
- Mots-clés sous-exploités (opportunité)
- Volume estimé pour les termes ciblés

**Messaging Comparison**
- Value props uniques dans leurs subtitles / descriptions
- Hiérarchie d'emphasis sur les features
- Patterns de ton et voix
```

Si l'utilisateur n'a pas fourni de concurrents, le skill propose une short list basée sur la catégorie et le brief — à valider avant de continuer.

### Phase 3 — Génération des textes

Pour chaque champ Apple + Play, produire le texte en respectant ces règles :

#### App Name (Apple ≤30 + Play ≤30)
- Format recommandé : `<NomCourt>: <BénéficeClé>` (ex: "Calm: Sleep & Meditation")
- Inclure 1-2 mots-clés primaires si possible (sans dénaturer le nom)
- ⚠️ Pas de wordings type "Best", "#1", "Top rated" → rejeté par Apple

#### Subtitle (Apple ≤30)
- Le bénéfice principal en une phrase courte
- Verbe action ou promesse mesurable
- Exemples qui convertissent : "Track your sleep, naturally" / "Build habits that stick" / "Plan your week in minutes"
- ⚠️ Apple compte les emojis comme 2-3 caractères selon le glyph — sécuriser en testant

#### Promotional Text (Apple ≤170)
- Visible **au-dessus** de la description, modifiable sans review
- Usage idéal : news / promo limited time / changements ponctuels (sortie d'une feature)
- Format : phrase d'annonce + CTA implicite

#### Description (Apple ≤4000 + Play full ≤4000)
- **Les 3 premières lignes** sont critiques — c'est ce que l'utilisateur voit avant le "more"
- Structure recommandée :
  1. Une phrase d'accroche (le bénéfice principal, en une ligne forte)
  2. Une phrase de validation (preuve sociale brève, niche cible)
  3. Une phrase de feature highlight la plus importante
  4. — (break)
  5. Liste à puces des 5-8 features clés
  6. — (break)
  7. Sections : "Pour qui ?", "Comment ça marche ?", "Subscription terms" (obligatoire si IAP)
  8. Footer légal : liens privacy / terms / contact

#### Keywords Apple (≤100, sans espaces après virgules)
- Stratégie : remplir le 100 chars exactement (single keywords + multi-words séparés par virgule)
- ⚠️ Pas de pluriels redondants (Apple matche les variations automatiquement)
- ⚠️ Pas de mots du App Name / Subtitle (déjà indexés) — gain de chars
- ⚠️ Pas de noms de concurrents (rejeté par Apple)
- Privilégier mots-clés à intent commercial > mots génériques

#### What's New (Apple ≤4000 + Play release notes ≤500)
- Quoi : nouveautés concrètes user-facing (pas le changelog tech interne)
- Format : liste à puces, max 5-8 entrées
- Ton : adresse direct à l'utilisateur ("Vous nous avez demandé X, on l'a fait")
- ⚠️ Apple permet 4000 chars, Play limite à 500 — produire 2 versions

### Phase 4 — Validation stricte par champ

Pour chaque champ, calculer le character count exact et marquer le statut :

```
✅ App Name (Apple)         : 28/30 (2 restants)
❌ Subtitle (Apple)         : 35/30 (5 en trop)
✅ Promotional Text         : 165/170 (5 restants)
✅ Description (Apple)      : 3847/4000 (153 restants)
✅ Keywords                 : 98/100 (2 restants)
✅ What's New (Apple)       : 1200/4000 (2800 restants)

✅ App Title (Play)         : 28/30
✅ Short Description (Play) : 75/80
✅ Full Description (Play)  : 3847/4000
✅ Release Notes (Play)     : 287/500
```

Si **n'importe quel** champ est en dépassement → bloquer la livraison et demander à l'utilisateur de raccourcir (proposer 2-3 variantes plus courtes).

### Phase 5 — Storyboard de captures d'écran (recommandations textuelles)

Sans générer les visuels (c'est le rôle de `aso-appstore-screenshots`), proposer une séquence narrative pour les **6-10 captures** App Store :

```markdown
### 📸 Screenshot Storyboard (recommandations)

1. **Hero / Bénéfice principal** — capture du écran-fierté (sharable moment) avec headline "Reprends le contrôle de ta journée"
2. **Feature flagship** — screenshot du core loop avec headline "Capture en 3 secondes"
3. **Avant / Après** — comparaison visuelle (si applicable)
4. **Social proof** — testimonial intégré dans une capture
5. **Catalogue de features** — vue d'ensemble compacte (3-4 features en grille)
6. **Achievement / streak** — capture du système de progression (cf. Playbook 2 §4)
7. **Settings / paramétrage** — montre que l'app est customisable
8. **CTA / engagement final** — "Rejoins X milliers d'utilisateurs"

À enchaîner avec `aso-appstore-screenshots` pour générer les visuels correspondants.
```

### Phase 6 — Custom Product Pages (CPP) — variantes textuelles

Si l'utilisateur cible plusieurs mots-clés différents, générer **2-3 variantes** d'App Name / Subtitle / Promotional Text optimisées par mot-clé. Garder Description et Keywords identiques (le storefront principal mute).

Exemple :
```
Variante "perte de poids" : Subtitle = "Perds tes kilos durablement"
Variante "musculation"    : Subtitle = "Construis ton physique"
Variante "santé"          : Subtitle = "Reste en forme au quotidien"
```

### Phase 7 — Génération des livrables

À la racine du kit, écrire deux fichiers :

**`ASO.md`** (humain, lisible) :
```markdown
# ASO Metadata — [Nom produit]

Date : YYYY-MM-DD
Catégorie ciblée : [Productivity / Health & Fitness / etc.]
Mots-clés primaires : [...]

## 📱 Apple App Store

### App Name (28/30)
[Nom]: [Bénéfice]

### Subtitle (28/30)
[Subtitle court action-verb]

### Promotional Text (165/170)
[Annonce / news]

### Description (3847/4000)
[Texte complet]

### Keywords (98/100)
[mot1,mot2,multi mot,...]

### What's New (1200/4000)
[Patch notes user-facing]

## 🤖 Google Play Store
[…idem…]

## 🎯 Competitive Analysis
[Sections de la phase 2]

## 📸 Screenshot Storyboard
[Recommandations 6-10 captures]

## 🎨 Custom Product Pages (variantes)
[Si applicable]

## ✅ Validation
[Character counts par champ avec status]
```

**`ASO.json`** (machine-readable, consommable par pipelines) :
```json
{
  "version": "1.0",
  "generated_at": "2026-05-11",
  "product": {
    "name": "Calm",
    "category_apple": "Health & Fitness",
    "category_play": "Health & Fitness",
    "primary_keywords": ["sleep", "meditation", "anxiety"]
  },
  "apple": {
    "app_name": {"text": "Calm: Sleep & Meditation", "count": 24, "limit": 30, "valid": true},
    "subtitle": {"text": "Sleep better tonight", "count": 20, "limit": 30, "valid": true},
    "promotional_text": {"text": "...", "count": 165, "limit": 170, "valid": true},
    "description": {"text": "...", "count": 3847, "limit": 4000, "valid": true},
    "keywords": {"text": "sleep,relax,anxiety,...", "count": 98, "limit": 100, "valid": true},
    "whats_new": {"text": "...", "count": 1200, "limit": 4000, "valid": true}
  },
  "play": {
    "app_title": {"text": "Calm - Sleep & Meditation", "count": 25, "limit": 30, "valid": true},
    "short_description": {"text": "...", "count": 75, "limit": 80, "valid": true},
    "full_description": {"text": "...", "count": 3847, "limit": 4000, "valid": true},
    "release_notes": {"text": "...", "count": 287, "limit": 500, "valid": true}
  },
  "all_valid": true,
  "competitors": ["Headspace", "Insight Timer", "Aura"],
  "cpp_variants": [
    {"keyword": "perte de poids", "subtitle": "Perds tes kilos durablement"},
    {"keyword": "musculation", "subtitle": "Construis ton physique"}
  ],
  "screenshot_storyboard": [
    {"slot": 1, "type": "hero", "headline": "Reprends le contrôle de ta journée"},
    {"slot": 2, "type": "feature-flagship", "headline": "Capture en 3 secondes"}
  ]
}
```

## 🚫 Anti-patterns

1. **Bourrer la description de mots-clés** — Apple et Play détectent et pénalisent. Le texte doit rester naturel.
2. **Utiliser "Best", "#1", "Top rated" dans l'App Name** — Apple rejette
3. **Nommer un concurrent dans les Keywords** — rejet App Store
4. **Pluriels redondants dans Keywords** ("recipe, recipes") — Apple matche les variations, gaspille des chars
5. **Description en bloc sans structure** — les 3 premières lignes doivent être percutantes, pas un mur de texte
6. **Subtitle sans verbe** ("The best app") — toujours action ou bénéfice mesurable
7. **What's New = changelog tech interne** ("Fixed bug #1234") — toujours user-facing
8. **Emojis dans App Name / Subtitle Apple** — comptés comme 2-3 chars, casse souvent les limites
9. **Promesses non tenues** ("Lose 20kg in 1 week") — rejet App Store + ruine la confiance
10. **Identique iOS / Android sans adaptation** — limites différentes (Short Description Play 80 vs Subtitle Apple 30), tonalité Android plus terre-à-terre

## ✅ Validation finale

Avant de livrer :
- Tous les champs `valid: true` dans le JSON
- Aucun mot interdit (Best / #1 / Top rated / nom concurrent)
- Description : 3 premières lignes vraiment percutantes (test : si on coupe au "more", l'utilisateur comprend ?)
- Keywords : 100 chars exactement utilisés
- Storyboard captures : 6-10 slots avec headlines distinctes
- CPP variants : cohérentes entre elles (même app, framing différent)

## 📤 Livrable

Deux fichiers à la racine du kit :
- `ASO.md` — humain, lisible, copy-paste vers App Store Connect / Play Console
- `ASO.json` — machine-readable, consommable par CI/CD ou pipelines de release

Cohérence brand : le tone of voice, les couleurs implicites et le storyboard captures doivent matcher l'identité du kit (cf. brand du kit + Playbook 3 §3.3 — cohérence brand → sharable → App Store).

## 🔄 Boucle d'usage avec les autres skills

```
ui-kit-creator → kit complet avec brand
       ↓
ui-kit-aso-metadata → ASO.md + ASO.json (textes)
       ↓
aso-appstore-screenshots → images des captures (visuels)
       ↓
Soumission App Store + Play Store
```

Re-invocation idempotente : régénérer après chaque évolution majeure du kit. Diff git de `ASO.md` montre l'évolution du wording entre versions.

## 🎓 Source d'inspiration

Skill inspiré de [TimBroddin/app-store-aso-skill](https://github.com/TimBroddin/app-store-aso-skill) — adapté à notre stack mobile-only et complémentarisé avec `aso-appstore-screenshots` pour couvrir l'ensemble du flow ASO (textes + visuels).
