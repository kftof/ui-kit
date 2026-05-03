---
name: ui-kit-brand-explorer
description: Propose plusieurs directions de logo SVG pour un UI kit existant (ou en création), permet à l'utilisateur de choisir une direction et de l'itérer, puis génère les 7 fichiers brand cohérents (logo, logo-mark, logo-monochrome, logo-dark, app-icon, splash, favicon) dans `ds/assets/brand/`. Brief court basé sur le PRD / description du projet — pas besoin d'être designer. À invoquer pour itérer sur un logo placeholder, refaire le logo d'un kit existant, ou explorer plusieurs directions visuelles avant de choisir.
---

# UI Kit Brand Explorer — propose et génère le logo + assets brand

Itère sur l'identité de marque d'un kit en proposant **plusieurs directions logo** distinctes, screenshotées côte-à-côte, parmi lesquelles l'utilisateur choisit. Une fois la direction validée, génère les 7 fichiers brand cohérents dans `ds/assets/brand/`.

## 🎯 Quand utiliser ce skill

- Le logo placeholder posé par `ui-kit-creator` ne convient pas
- Refaire le logo sur un kit existant (recette `ui-kit-editor` → "Refaire le logo")
- Explorer plusieurs directions visuelles avant de figer le brand
- Compléter les variantes manquantes (logo-dark si pas généré au creator, splash, favicon)
- L'utilisateur n'est pas designer et a besoin de **propositions** plutôt que de descriptions à fournir

## 📋 Inputs requis

**Obligatoire** :
1. **Path du kit** — racine d'un kit avec `ds/tokens.css` existant (palette + typo déjà décidées)

**Recommandé** :
2. **Brief court** — si fourni explicitement par l'utilisateur, l'utiliser. Sinon le skill l'extrait automatiquement de :
   - `PRD.md`, `PRD_M1.md`, `prd.md` (préfixe les premiers 500 mots)
   - `README.md` du kit
   - Section "description" / "tagline" / "produit" dans `GUIDELINES.md`
   - Si rien trouvé : demander à l'utilisateur en 1 phrase

Format minimum du brief (3 infos suffisent) :
- **Nom du produit** : ex. "TaskMate", "Lumen Notes", "Geocode Me"
- **Domaine / métier** en 5 mots : ex. "tâches productivité minimaliste", "carnet de recettes manuscrit", "scanner SEO local"
- **Ton** : 1-3 mots — ex. "techy clean", "chaleureux nostalgique", "pro institutionnel"

L'utilisateur n'a pas besoin de fournir des refs visuelles, mood-board, palette ou typo : tout vient des `ds/tokens.css` existants. Le skill choisit la direction visuelle dans le cadre des tokens du DS.

## 🧭 Workflow

### Phase 1 — Reconnaissance

1. **Charger les tokens** : grep dans `ds/tokens.css` les variables `--primary`, `--secondary`, `--accent`, `--text`, `--text-strong`, `--bg`, `--surface`, `--font-display`, `--font-body`, `--font-handwritten` (si présent). Ces couleurs et fonts sont la palette obligatoire du logo — pas d'invention.
2. **Charger le brief** : extraire des fichiers du projet ou demander à l'utilisateur (cf. inputs).
3. **Inventaire des assets brand existants** : lister `ds/assets/brand/*.svg` (s'ils existent). Si présents, ils servent de baseline (l'utilisateur veut les remplacer).
4. **Détecter le mode** :
   - **Mode placeholder** : `logo.svg` existe mais c'est juste un texte "TaskMate" en typo neutre → exploration totale (6 directions)
   - **Mode itération** : l'utilisateur a déjà choisi une direction (round précédent), demande des ajustements fins → 3-4 variantes de la direction validée
   - **Mode refonte** : l'utilisateur veut tout repartir de zéro → exploration totale

### Phase 2 — Proposition de 6 directions

Générer **6 directions logo distinctes** en SVG inline, chacune respectueuse de :
- Palette du DS (`--primary`, `--secondary`, `--text`, etc.)
- Typo du DS (`--font-display`, `--font-handwritten` si présent)
- Ton du brief

#### Vocabulaire des directions

1. **Lettermark** (initiales stylisées)
   - `<text>` SVG en `--font-display`, font-weight 700-800, kerning resserré
   - 1-2 lettres (initiales du nom)
   - Optionnel : container géométrique (cercle, carré arrondi, hexagone)

2. **Wordmark** (le nom complet en typo distinctive)
   - `<text>` SVG en `--font-display` avec un détail typographique (ligature, swash, italique partiel, couleur d'une lettre, point sur i, etc.)
   - Exploiter les axes opentype si la font est variable (Fraunces, Inter…) : opsz, wght, slnt, etc.
   - Tracking serré ou ouvert selon le ton

3. **Combinaison mark + wordmark**
   - Petit symbole à gauche + nom à droite (ou symbole au-dessus, nom dessous)
   - Le mark peut être abstrait, géométrique, ou métier
   - Équilibré pour usage horizontal (header) ou vertical (splash)

4. **Symbole géométrique abstrait**
   - Composition de formes simples : cercle + arc, carrés imbriqués, triangle découpé, ligne courbe, etc.
   - Pas de signification métier explicite, juste un signe distinctif
   - Doit fonctionner monochrome ET en couleur

5. **Symbole métier**
   - Évoque le domaine du produit en forme simple, lisible à 16×16 (favicon)
   - Cuisine → papillote, feuille, ustensile, casserole stylisée
   - Tâches → checkmark, liste, point qui se valide
   - Finance → barres, courbes, pièce, K minuscule
   - Social → bulles, points connectés, anneaux
   - Lecture → page repliée, signet, livre ouvert
   - **Pas de mascotte / illustration complexe** — limite SVG génératif. Si l'utilisateur veut une mascotte détaillée, le signaler en `unknowns` avec note "à confier à un designer humain".

6. **Monogramme / ligature**
   - Superposition stylisée de 2 lettres (initiales) qui se croisent ou s'imbriquent
   - Peut former une marque très distinctive et compacte
   - Excellente déclinaison favicon

Le LLM doit **adapter** ces directions au brief, en piochant dans la palette et la typo du DS — qui peuvent être de n'importe quelle famille. Le skill n'a aucun biais esthétique propre : il suit le DS et le ton du brief. Quelques mappings indicatifs (non exhaustifs) :

| Ton du brief | Choix typiques |
|---|---|
| Chaleureux nostalgique / éditorial | serif vivante (Fraunces, Playfair) + ligature manuscrite + couleurs choisies dans la palette du DS |
| Techy / clean / productivité | sans-serif géométrique + lettermark net + symbole abstrait minimal |
| Dark-first / techno / cyber | wordmark contrasté (claire sur fond sombre) + accent saturé + symbole géométrique précis ; la variante "claire sur fond `--text` ou `--bg-dark`" devient la principale, pas la secondaire |
| Sobre / institutionnel / fintech | sans-serif neutre (Inter, IBM Plex) + lettermark ou monogramme épais + palette restreinte (≤2 couleurs) |
| Moderne saturé / SaaS jeune | sans-serif rond ou variable + symbole géométrique coloré + accent vif (gradient si présent dans le DS) |
| Technique / dev tool / CLI-style | mono ou sans-serif technique (JetBrains Mono, Geist Mono) + lettermark monospace + symbole minimal type bracket / chevron / ASCII |
| Élégant / luxe / éditorial premium | serif display fine + wordmark large tracking + symbole abstrait minimal, peu d'effets |

Si le DS du projet est **dark-first** (`--bg` foncé, `--text` clair), le skill génère les 4 variantes en respectant cette dominance : la variante "claire sur fond sombre" devient le rendu canonique, le "couleur primary" reste la version brandée, le "monochrome dark" sert pour les fonds clairs occasionnels (impressions, cartes de visite). Aucun choix esthétique imposé — tout vient du DS.

### Phase 3 — Génération du `Brand explorer.html`

Fichier temporaire à la racine du kit :

```
[projet]/Brand explorer.html      ← généré par le skill, sera supprimé après choix
```

Structure :

```html
<!doctype html>
<html lang="fr">
<head>
  <link rel="stylesheet" href="ds/tokens.css">
  <link rel="stylesheet" href="ds/components.css">
  <style>
    body { padding: 32px 16px 96px; background: var(--bg); font-family: var(--font-body); }
    .head { max-width: 1480px; margin: 0 auto 32px; }
    .head h1 { font-size: var(--fs-2xl); font-weight: var(--fw-bold); margin: 0 0 6px; }
    .head .sub { color: var(--text-muted); font-size: var(--fs-md); }
    .head .brief { margin-top: 16px; padding: 16px; background: var(--surface); border-radius: var(--radius-md); }

    .grid {
      max-width: 1480px; margin: 0 auto;
      display: grid; grid-template-columns: repeat(2, 1fr); gap: 24px;
    }
    .direction {
      background: var(--surface);
      border-radius: var(--radius-lg);
      padding: 24px;
      box-shadow: var(--shadow-md);
    }
    .direction-head { display: flex; justify-content: space-between; align-items: center; margin-bottom: 16px; }
    .direction-num { font-family: var(--font-mono); font-size: var(--fs-sm); color: var(--text-muted); background: var(--neutral-100); padding: 4px 10px; border-radius: var(--radius-pill); }
    .direction-name { font-size: var(--fs-md); font-weight: var(--fw-semibold); }
    .direction-desc { font-size: var(--fs-sm); color: var(--text-muted); margin-bottom: 16px; line-height: 1.5; }

    /* 4 variantes côte-à-côte */
    .variants { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; }
    .variant { background: var(--bg-paper); border-radius: var(--radius-sm); padding: 24px; display: flex; align-items: center; justify-content: center; min-height: 120px; }
    .variant.dark-bg { background: var(--text); }
    .variant.app-icon { background: var(--primary); padding: 0; }
    .variant-label { position: absolute; bottom: 4px; right: 6px; font-size: 9px; color: var(--text-subtle); font-family: var(--font-mono); }
  </style>
</head>
<body>
  <div class="head">
    <h1>Brand explorer — [Nom produit]</h1>
    <div class="sub">6 directions logo proposées. Chaque ligne montre 4 variantes : couleur primary, monochrome dark, monochrome light (sur fond sombre), app-icon (sur fond primary).</div>
    <div class="brief">
      <strong>Brief :</strong> [contenu du brief court — nom + domaine + ton]
    </div>
  </div>
  <div class="grid">
    <!-- 6 directions × 4 variants = 24 cells -->
    <div class="direction">
      <div class="direction-head">
        <span class="direction-num">① Lettermark</span>
        <span class="direction-name">Initiales stylisées</span>
      </div>
      <div class="direction-desc">Description courte de cette direction.</div>
      <div class="variants">
        <div class="variant"><svg ...>logo couleur primary</svg></div>
        <div class="variant"><svg ...>logo dark monochrome</svg></div>
        <div class="variant dark-bg"><svg ...>logo light monochrome</svg></div>
        <div class="variant app-icon"><svg ...>app-icon avec safe area</svg></div>
      </div>
    </div>
    <!-- ② Wordmark, ③ Combination, ④ Géométrique abstrait, ⑤ Symbole métier, ⑥ Monogramme -->
  </div>
</body>
</html>
```

### Phase 4 — Validation visuelle obligatoire

Le skill **doit** screenshoter le `Brand explorer.html` avant de demander un choix à l'utilisateur :

```
1. mcp__chrome-devtools__new_page  url: file:///path/to/Brand%20explorer.html
2. mcp__chrome-devtools__list_console_messages → 0 erreur (sinon SVG mal formé)
3. xmllint --noout sur tous les SVG inline (extraction depuis le HTML, pipe vers xmllint)
4. mcp__chrome-devtools__take_screenshot fullPage:false (la page est généralement courte 2-3 viewports)
   → si > 2 viewports : viewport-by-viewport, sinon fullPage OK
5. Read le screenshot
6. Vérifier que les 6 directions rendent ET que les 4 variantes par direction sont visibles, distinguables, sans icône broken
```

Si une direction ne rend pas (broken icon ou SVG vide), corriger avant de présenter à l'utilisateur. **Ne jamais demander de choisir parmi des options dont au moins une est cassée**.

### Phase 5 — Choix utilisateur

Après screenshot validé, demander à l'utilisateur :

> "J'ai généré 6 directions de logo dans `Brand explorer.html`. Voir le screenshot ci-dessus. Tu choisis quelle direction (numéro 1-6) ? Si tu veux affiner une direction avant de figer (couleurs, typo, échelle), dis-moi laquelle et ce que tu veux ajuster."

L'utilisateur peut :
- Choisir un numéro → on passe à phase 6
- Demander des ajustements sur une direction → on regénère uniquement cette direction avec les ajustements (mode itération, 3-4 variantes fines)
- Demander une 7ème direction non couverte → on l'ajoute au `Brand explorer.html` et on re-screenshot
- Tout rejeter et demander un nouveau set → on regénère 6 directions différentes

### Phase 6 — Génération des 7 fichiers brand

Une fois la direction validée, écrire dans `ds/assets/brand/` :

| Fichier | Contenu | Variante de la direction validée |
|---|---|---|
| `logo.svg` | Logo principal (mark + wordmark si direction "combinaison", sinon juste mark ou wordmark) | A — couleur primary |
| `logo-mark.svg` | Symbole seul, sans texte (utile pour favicon, watermark, app-icon) | A — couleur primary |
| `logo-monochrome.svg` | Variante 1 couleur, `currentColor` pour s'adapter au fond | toutes les couleurs remplacées par `currentColor` |
| `logo-dark.svg` | Inversé pour fond sombre (texte clair sur transparent) | C — light sur fond sombre |
| `app-icon.svg` | Icône d'app (1024×1024, square avec safe-area iOS) | D — mark dans un carré arrondi avec fond primary |
| `splash.svg` | Composition centrée du logo + tagline optionnel, sur fond `--bg` ou `--surface` | composition pleine, viewBox 1080×1920 |
| `favicon.svg` | Logo simplifié pour 16×16 / 32×32, mono ou color | logo-mark optimisé low-res |

**Règles** :
- Tous les SVG sont **autonomes** (pas de référence à des fonts externes via `<link>`)
- **`<text>` SVG** : respecter les 3 règles de `ui-kit-creator` § "Texte dans les SVG assets" — **une seule** valeur de `font-family` (pas de fallback CSS `"A, B, serif"` qui casse `flutter_svg`), pas de `font-variation-settings` (utiliser `font-weight` / `font-style` standards), famille présente dans `ds/assets/fonts/<x>/`
- **Logo qui voyage** (App Store, README, marketing, presse) : générer en plus une variante `logo-outlined.svg` avec `<text>` converti en paths (text-to-outline) — indépendant des fonts enregistrées côté code, robuste partout
- **Couleurs** : ok d'utiliser `var(--primary)` dans les SVG inline d'un HTML qui charge le DS, MAIS pour les fichiers SVG autonomes copiables (designer / mobile / Flutter), utiliser **les hex en dur** (extraits de `tokens.css`) — `flutter_svg` ne résout pas les `var(--…)`
- `xmllint --noout` doit passer sur les 7 fichiers (caractères spéciaux échappés, namespace présent)
- Naming : kebab-case strict, lowercase

### Phase 7 — Intégration dans `UI Kit.html`

Une fois les 7 fichiers brand écrits, mettre à jour la **section Brand de `UI Kit.html`** pour qu'elle rende les nouveaux assets dans la galerie. Si la section n'existe pas (kit minimal), l'ajouter.

### Phase 8 — Cleanup

Supprimer le fichier temporaire `Brand explorer.html` à la racine du kit (à moins que l'utilisateur veuille le garder pour itérer plus tard — auquel cas le renommer `Brand explorer - <direction validée>.html`).

### Phase 9 — Validation visuelle finale

Re-ouvrir `UI Kit.html` via chrome-devtools, scroll jusqu'à la section Brand, screenshot pour confirmer que les 7 nouveaux assets s'affichent bien et que rien n'est cassé.

## 🎨 Limites du SVG génératif

Ce que le skill **fait bien** :
- Lettermark (initiales en typo + composition simple)
- Wordmark (texte stylisé, ligatures opentype)
- Géométrique simple (cercles, polygones, paths courbes)
- Combinaisons mark+wordmark
- Monogrammes (superpositions de lettres)
- Symboles métier abstraits (feuille stylisée, papillote, checkmark, etc.)

Ce que le skill **ne fait PAS bien** (signaler en `unknowns` + placeholder) :
- Mascottes / personnages illustrés (Duolingo owl, Mailchimp Freddie, etc.)
- Illustrations photoréalistes
- Effets complexes (glitch, gradient mesh sophistiqué, texture grain à la main)
- Logos à plus de 4-5 chemins courbes complexes

**Recommandation pour les cas non couverts** : produire un placeholder propre + note explicite "Direction validée mais finalisation graphique à confier à un designer humain. Le skill a posé un brouillon utilisable comme spec."

## 🚫 Anti-patterns

1. **Inventer une couleur hors `tokens.css`** — toujours puiser dans la palette du DS. Si une nuance manque (ex: une variante claire pour le splash), proposer d'étendre `tokens.css` plutôt que de hardcoder.
2. **Inventer une typo hors `tokens.css`** — utiliser `--font-display`, `--font-body`, `--font-handwritten` selon le contexte.
3. **Logo qui ne fonctionne pas en monochrome** — chaque direction doit avoir une variante monochrome lisible. Si la direction repose entièrement sur la couleur, elle est fragile.
4. **App-icon sans safe-area iOS** — l'icône d'app doit avoir un padding interne d'environ 10% pour respecter la safe-area du masque iOS rounded square.
5. **Favicon trop détaillé** — à 16×16, les détails fins disparaissent. Le favicon doit être une version **simplifiée** du mark.
6. **SVG avec caractères non échappés** (`&` dans aria-label/title) — `xmllint --noout` rejette, le browser refuse en `<img>`. Toujours `&amp;`, `&lt;`, `&gt;`.
7. **Splash trop dense** — le splash apparaît 1-2 secondes au lancement. Composition centrée minimale : logo + éventuellement tagline 1 ligne. Pas de paragraphes, pas de visuels secondaires.
8. **Présenter des directions cassées** à l'utilisateur — toujours screenshot + lire avant de demander de choisir.
9. **Dépasser 6 directions** dans la première proposition — au-delà, l'utilisateur sature et choisit mal. Si vraiment besoin, faire 6 directions par "batch" et itérer.
10. **Itérer plus de 3 rounds** — au-delà, c'est que le brief est mal posé ou que l'utilisateur a besoin d'un designer humain. Le skill signale et arrête.
11. **`font-family="A, B, serif"` (liste CSS) sur un `<text>` SVG** — `flutter_svg` ne sait pas parser une liste, il cherche la chaîne complète comme nom de famille → texte invisible côté Flutter. Toujours **une seule** famille.
12. **`font-variation-settings="…"` sur un `<text>` SVG** — non supporté par `flutter_svg`. Utiliser `font-weight` et `font-style` standards (les fonts variables interpolent automatiquement).
13. **Référencer une font dans un `<text>` SVG sans la déclarer dans `ds/assets/fonts/<x>/`** — rendu OK browser (font CDN à la volée) mais cassé Flutter. Toujours vérifier que chaque famille référencée a son `.ttf` dans `ds/assets/fonts/`.
14. **Livrer un seul `logo.svg` avec `<text>` pour tous les usages** — pour App Store / README / marketing / presse, fournir aussi `logo-outlined.svg` avec text-to-outline (paths au lieu de text), indépendant des fonts.

## 📤 Livrable

Mise à jour de `ds/assets/brand/` :
- `logo.svg`
- `logo-mark.svg`
- `logo-monochrome.svg`
- `logo-dark.svg`
- `app-icon.svg`
- `splash.svg` (dans `ds/assets/brand/splash/splash.svg` selon convention)
- `favicon.svg`

+ section Brand de `UI Kit.html` mise à jour avec les nouveaux rendus.

Le fichier `Brand explorer.html` est nettoyé (sauf si l'utilisateur a demandé à le garder).

Mention dans le retour final :
- Direction validée + courte description
- Liste des 7 fichiers écrits
- Screenshots de la galerie Brand de `UI Kit.html` post-modif
- Limitations éventuelles (cas non couverts par le SVG génératif → renvoyer vers designer humain)

## 🔄 Re-invocation idempotente

Re-invoquer le skill avec un kit qui a déjà des assets brand :
- Mode : refonte (l'utilisateur veut tout regénérer) → backup les anciens en `ds/assets/brand/.archive/<timestamp>/` puis génère les nouveaux
- Mode : itération (variantes fines de la direction actuelle) → 3-4 variantes au lieu de 6 directions

## 🎯 Workflow utilisateur typique

```
1. ToF crée un kit via ui-kit-creator → logo placeholder posé en ds/assets/brand/logo.svg
2. ToF invoque ui-kit-brand-explorer (sans argument)
3. Le skill lit ds/tokens.css + extrait le brief depuis README.md / PRD.md
4. Génère Brand explorer.html avec 6 directions × 4 variantes
5. Screenshot + montre à ToF
6. ToF dit "j'aime la direction 3 mais le bleu est trop foncé"
7. Le skill regénère uniquement la direction 3 avec une teinte primary plus claire
8. Re-screenshot
9. ToF valide la direction 3 ajustée
10. Le skill écrit les 7 fichiers brand finaux + met à jour UI Kit.html
11. Cleanup Brand explorer.html
12. ToF re-invoque ui-kit-prototype pour rafraîchir le prototype avec le nouveau logo
```
