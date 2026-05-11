# Playbook 3 — Psychologie & Design à haute conversion

Doctrine pour intégrer **psychologie comportementale + stratégie visuelle marketing** dans le système de design, au-delà de la pure esthétique technique. À consulter quand on conçoit l'onboarding, le paywall, les écrans sharable et les assets App Store (captures d'écran promotionnelles).

Un design beau ne suffit pas — il doit être **persuasif**. Les patterns ci-dessous sont distillés de l'analyse d'apps à haute conversion (Cal AI, Lapse, Praying Mantis, etc.) et applicables à toute niche.

Règles **actionables**, numérotées pour référence directe.

---

## 1. Design de l'onboarding comme tunnel de vente

### 1.1 Aversion à la perte intégrée au design
- Concevoir un onboarding **long et engageant** (10-15 minutes) — chaque étape investit l'utilisateur émotionnellement
- Pattern visuel : barre de progression visible mais qui avance lentement, donnant l'impression d'un parcours significatif
- Le user qui voit "Étape 8/12" après 5 min ne lâche pas — il veut compléter

### 1.2 Personnalisation visuelle (effet miroir)
- Système design capable de **réafficher dynamiquement** les réponses du user dans les écrans suivants
- Composant `data-hint="reuse:answer-N"` pour signaler à l'agent codeur qu'un placeholder doit afficher une réponse passée
- Exemple : user dit "gagner du temps" → écran suivant affiche en bandeau "Nous préparons votre plan pour vous faire gagner du temps" (typo display large, accent saturé)

### 1.3 Moment "Aha" visuel dans la première minute
- Élément visuel **percutant** dans l'écran 1-3 : statistique géante en typo display, illustration choc, animation hero qui matérialise le problème
- La taille de la statistique compte : `font-size: var(--fs-display-2xl)` minimum, dominante de l'écran
- Pas de body text long — la stat doit pouvoir être lue en 2 secondes

### 1.4 Boutons CTA conformes (pas custom-ish)
- Bouton primaire = `.btn--primary` avec hauteur ≥ 56px, full-width, label action-verb ("Continuer", pas "Suivant" qui est passif)
- Pas de bouton fantaisiste qui distrait de l'action — l'utilisateur cherche **un seul** chemin évident
- Couleur du bouton = `var(--primary)` saturée, contraste max avec le background

---

## 2. Stratégie visuelle App Store — captures d'écran comme publicités

### 2.1 Design axé sur le bénéfice (show, don't tell)
- Chaque capture d'écran promotionnelle doit montrer un **résultat utilisateur**, pas une feature technique
- Mauvais titre : "IA de génération de tâches" → bon titre : "Reprenez enfin le contrôle de votre journée"
- Le visuel doit prouver le bénéfice (avant/après, screenshot d'un résultat réel, transformation visible)

### 2.2 Analyse des leaders de la niche avant de figer le design
- Avant le freeze brand, étudier les 5 apps les plus téléchargées de la niche cible
- Identifier les patterns visuels qui convertissent dans cette catégorie : couleurs dominantes, quantité de texte sur les screenshots, type d'iconographie (3D / flat / hand-drawn), style des CTA
- **Ne pas copier aveuglément** — extraire les invariants (ex: les apps fitness utilisent toutes des verts/oranges saturés) et différencier sur le reste

### 2.3 Custom Product Pages (CPP) — variantes par mot-clé
- Prévoir des **variantes design** des captures d'écran selon le mot-clé qui amène l'utilisateur
- Si keyword "perte de poids" → visuels de sport / silhouette
- Si keyword "musculation" → visuels d'haltères / progression
- Même app, framing visuel différent selon l'intent du chercheur
- Système design doit pouvoir générer plusieurs CPP variants sans recoder

### 2.4 Contraste icône d'app + clarté du premier bénéfice
- Icône d'app testée sur fond clair ET fond sombre des résultats de recherche App Store
- Le **premier bénéfice** affiché en titre de capture d'écran doit être lisible **à la taille de prévisualisation** (≈ 200px de large)
- Pas de texte multi-ligne dans la capture #1 — une phrase max, large typo, contraste fort

---

## 3. Différenciation visuelle — sortir du "AI slope"

### 3.1 Identité visuelle avec une "âme"
- Le risque de l'IA design = production d'interfaces génériques interchangeables ("AI slope")
- Solution : **un détail distinctif** dans le DS qui rend l'app reconnaissable au premier coup d'œil — peut être :
  - Une typo serif vivante (Fraunces, Playfair) à contre-courant des sans-serif partout
  - Un style "hand-drawn sketch" pour les illustrations / icônes custom
  - Une palette terreuse (terracotta, olive, ivoire) quand la niche utilise du bleu corporate
  - Une animation signature (logo qui pulse, transition de page particulière)
- **Un seul** signe distinctif fort — pas dix superposés

### 3.2 Hand-drawn sketch style (option)
- Pattern à considérer pour apps de niche émotionnelle (méditation, écriture créative, journal intime)
- Plus authentique et mémorable qu'une interface standard SaaS bleue
- Implémentation : illustrations SVG type "rough" (lib `rough.js` ou tracé à la main), pas du flat design propret
- Adam Lyttle's recommandation : sort des sentiers battus quand la catégorie est saturée

### 3.3 Cohérence brand → sharable moments → App Store
- L'identité visuelle doit être **identique** entre l'app (UI Kit), les sharable moments (cf. Playbook 2 §2) et les captures d'écran App Store
- Si user partage un screenshot, qu'il convertisse → la capture App Store doit visuellement matcher ce que le user voit dans l'app
- Pas de "couleurs marketing" différentes des couleurs in-app

---

## 4. Palettes & typos psychologiques par niche

### 4.1 Mapping niche → palette type
Indicatif — toujours adapter au brief produit, mais utiles comme point de départ :

| Niche | Palette type | Typo type |
|---|---|---|
| Méditation / mindfulness | Pastels apaisants (lavande, sauge, sable) | Serif douce ou sans-serif arrondie |
| Productivité / focus | Bleu profond + accent saturé (orange / corail) | Sans-serif géométrique (Inter, Geist) |
| Fitness / sport | Vert saturé / orange / rouge | Sans-serif condensée (Inter Bold, Bebas) |
| Finance / fintech | Bleu corporate + vert neutre OU noir + accent vif | Sans-serif neutre (IBM Plex, Inter) |
| Jeu / kids | Couleurs vives saturées (rouge, jaune, bleu primaire) | Display ronde, fun |
| Éditorial / lecture | Sépia / ivoire / terracotta | Serif vivante (Fraunces, Playfair) |
| Dating / social | Rose / corail / accent vif | Sans-serif moderne arrondie |
| Dev tool / CLI | Noir + accent saturé (vert terminal / cyan / magenta) | Mono (JetBrains Mono, Geist Mono) |

### 4.2 Test du contraste sur l'icône d'app
- Icône en couleur primary saturée sur 1024×1024
- Vérifier rendu sur fond clair (page d'accueil iOS) ET fond sombre (résultats search App Store)
- Si l'icône disparaît sur un des deux fonds → ajouter une bordure ou un cadre de contraste

### 4.3 Itération rapide via IA (palettes + variantes)
- Générer **plusieurs variantes** de palette pour la même niche via prompt IA
- Tester en visualisation côte-à-côte (cf. `ui-kit-brand-explorer` qui matérialise ça)
- Choisir la variante qui maximise différenciation **dans la niche** (pas qui plaît au designer dans l'absolu)

---

## 5. Critères d'audit machine-readable

Liste binaire utilisée par `ui-kit-audit` pour évaluer un kit. Chaque critère = `pass` / `fail` / `n/a`.

| ID | Critère | Où chercher |
|---|---|---|
| OB-V1 | Onboarding contient ≥ 8 étapes / écrans (assume aversion à la perte) | comptage cells dans `flows/*onboarding*` |
| OB-V2 | Barre de progression visible pendant l'onboarding | grep progress / step indicator |
| OB-V3 | Effet miroir : placeholders / `data-hint="reuse:..."` présents | grep placeholders dans cells post-questions |
| OB-V4 | Au moins un écran avec stat ou prise de conscience en typo display | font-size ≥ display-xl + texte court |
| OB-V5 | CTA primaires homogènes (`.btn--primary` full-width, ≥ 56px) | inspection cells onboarding |
| CV-1 | Captures App Store axées bénéfice (pas feature) — vérifiable sur les wordings | section App Store assets du kit (si présente) |
| CV-2 | Système prévoit ≥ 2 variantes de captures (Custom Product Pages) | présence `flows/*app-store*` avec plusieurs variants |
| CV-3 | Icône app testée light + dark (deux rendus) | `ds/assets/brand/app-icon-light.svg` + `app-icon-dark.svg` ou équivalent |
| CV-4 | Premier bénéfice de la 1ère capture lisible à 200px (typo display, 1 ligne) | inspection asset capture #1 |
| DIFF-1 | Au moins un signe visuel distinctif identifié (typo, palette, style illustration, animation signature) | `data-hint="signature-style:..."` ou section dédiée GUIDELINES.md |
| DIFF-2 | Cohérence brand entre UI app + sharable moments + captures App Store (mêmes couleurs / typo) | comparaison cross-flows |
| DIFF-3 | Pas de palette "AI slope" générique (bleu saas / gris flat / corail random sans rationale) | revue palette tokens.css vs niche cible |
| PA-1 | Palette du DS cohérente avec la niche (mapping §4.1 — indicatif) | comparison tokens.css × niche déclarée dans README/PRD |
| PA-2 | Icône app a un contraste suffisant sur fond clair ET fond sombre | inspection visuelle `ds/assets/brand/app-icon.svg` |
