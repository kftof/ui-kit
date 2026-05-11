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

### 1.5 Peak-End Rule (loi du pic-fin)
> "L'utilisateur retient deux moments d'une expérience : le **pic** (le plus intense) et la **fin** (la dernière impression). Le reste est moyenné mentalement et largement oublié."

Implication design directe : ne pas saupoudrer l'effort visuel uniformément sur tous les écrans. Identifier les deux moments à designer **à fond** dans chaque flow.

**Designer le PIC** :
- Moment où l'utilisateur ressent la valeur la plus forte — résultat IA généré, niveau franchi, première création réussie, streak validé, paywall passé
- Investir le maximum de polish sur ce screen : micro-animation célébratoire, son optionnel (mute-friendly), confettis légers, typo display large pour le score / résultat, badge visible
- C'est sur ce screen qu'on déclenche la demande d'avis (cf. Playbook 1 §2.2) et qu'on rend le screen "sharable" (cf. Playbook 2 §2)

**Designer la FIN** :
- Dernier écran d'un parcours : fin d'onboarding, fin de session, fin de niveau, dernier écran avant fermeture
- Soin particulier à la dernière transition : pas un crash vers le hub, mais une affirmation de progrès + un nudge vers la prochaine session
- Pattern type : carte résumée ("Tu as réalisé X aujourd'hui") + CTA secondaire pour revenir ("Reviens demain pour Y")
- L'écran de "Logout / Bye" ou "Subscription expired" comptent comme des fins — ne pas les négliger

⚠️ **Erreur fréquente** : effort distribué uniformément → user retient une expérience "moyenne". Concentrer sur les pics et fins → user retient une expérience "exceptionnelle" avec exactement le même nombre d'écrans soignés.

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

## 5. Catalogue de styles UI nommés (vocabulaire de référence)

Plutôt que de décrire "un design moderne épuré avec des touches de couleur", désigner explicitement le **style nommé** facilite l'alignement entre toi, le designer, et l'agent codeur. Liste indicative (les plus pertinents mobile en 2026) — à utiliser au moment du choix de direction visuelle (`ui-kit-creator` étape 2 / `ui-kit-brand-explorer`).

| Style | Description rapide | Quand l'utiliser |
|---|---|---|
| **Liquid Glass / Apple Visual Design** | Effets de verre translucide avec blur, reflets dynamiques, layers superposés | App premium iOS, alignée avec les patterns Apple récents (iOS 26+) |
| **Glassmorphism** | Cards translucides avec backdrop-filter blur, bordures subtiles, ombres douces | Apps fintech, productivité premium, dashboards |
| **Claymorphism** | Formes 3D arrondies, ombres internes + externes, palette tendre, sensation tactile | Apps kids, jeux casual, apps de bien-être / méditation |
| **Neumorphism** | Soft UI : ombres internes + externes simulant relief subtil, monochrome | Apps niche, audio control, calculatrices, weather (usage limité — accessibilité difficile) |
| **Skeuomorphism** | Imitation d'objets réels (carnet, vinyle, calendrier papier) | Apps de notes, journal intime, lecture, cuisine — quand authenticité > minimalisme |
| **Brutalism** | Typo XXL, contraste max, layouts asymétriques, hover/tap effects bruts | Apps statement, presse alternative, mode, art — différenciation forte |
| **Swiss Minimalism / International Style** | Helvetica-like, grille stricte, espace blanc dominant, accent rare | Apps pro, finance, news premium |
| **Editorial / Magazine** | Mise en page de magazine print : grands titres serif, photos pleine largeur, espaces aérés | Apps lecture, news, food, voyage |
| **Cyberpunk / Synthwave** | Néons saturés, glow effects, mono techy, palettes magenta/cyan, scanlines | Jeux, apps dev tools, niche techno |
| **Hand-drawn Sketch** | Tracés "à la main" pour illustrations / icônes, typo serif vivante, palette terreuse | Méditation, journal intime, écriture créative, parenting |
| **Material Expressive** | Material You / Material 3 enrichi : couleurs dynamiques (dérivées du wallpaper), formes morphables | Apps Android natives modernes, gen Z target |
| **Dark Mode First** | Conçu primary pour fond sombre, couleurs saturées sur background `#0A0A0F` ou similaire, accent vif | Dev tools, crypto, dating, gaming, apps usage nocturne |
| **Y2K / Retro Tech** | Couleurs saturées, dégradés chrome, typo display rétro, glow, transparences | Niche jeune, fashion, music apps, social |
| **Solarized / Warm Neutral** | Palette terreuse (ivoire / sepia / terracotta / olive), serif vivante, papier-friendly | Apps lecture, food, slow living, journaling |
| **Bento Grid Layout** | Composition modulaire de cards de tailles variées, polish moderne | Dashboards, profils, portfolios, home screens d'apps données |

**Règle d'usage** : choisir **UN seul** style dominant par produit (pas un mélange). Le style choisi influence le DS entier (`tokens.css`, `components.css`, illustrations, brand). Documenter le style choisi dans `GUIDELINES.md` du kit — utile pour l'agent codeur et pour les itérations futures.

---

## 6. Anti-patterns mobile spécifiques

Erreurs fréquentes qui cassent la conversion / l'usabilité sur mobile spécifiquement (ne pas confondre avec les anti-patterns web). À éviter par construction dans le kit :

1. **Gradient / blur excessif** — perd la hiérarchie, alourdit le rendu, casse la lisibilité. Réserver aux accents (1-2 endroits max par écran).
2. **Plus de 4 tailles ou 3 poids typographiques par écran** — fragmente l'attention. Maintenir une échelle stricte définie dans `tokens.css`.
3. **Espacement aléatoire** — doit suivre la grille 4pt / 8pt. Pas de `padding: 13px`. Tous les spacings dans `var(--space-N)`.
4. **Contenu clé caché derrière bannières / taps additionnels** — l'utilisateur mobile abandonne après 2 taps cherchant l'info. Le contenu primaire doit être visible **above the fold** sans scroll.
5. **CTA en dehors de la thumb zone** — sur iPhone 6.7" / Android XL, le coin haut-droit est inatteignable au pouce. CTA primaires en bas (tiers inférieur), icônes secondaires en haut.
6. **Empty states génériques sans guidance** — un écran vide doit toujours pointer vers la prochaine action concrète (cf. Playbook 2 §6.3 RM-3).
7. **Sliders pour des données fréquentes ou précises** — sur mobile, un slider est peu précis et lent. Préférer steppers (-/+) pour les valeurs précises, ou input numérique avec keyboard `decimalPad`.
8. **Hiérarchie visuelle absente** — tout égal = rien hiérarchisé. Chaque écran a UN élément focal (titre principal / CTA primaire / résultat) plus contrasté que le reste.
9. **Labels surpondérés vs valeurs** — afficher "Prix : $9.99" en taille égale est faux. La **valeur** ($9.99) doit dominer, le label ("Prix") est secondaire / muet (`var(--text-muted)`).
10. **Ombres gris/noir pur sur fond coloré** — produit un effet sale et brunâtre. Les ombres sur fond coloré utilisent une teinte assombrie de la couleur du fond (ex: `rgba(40, 20, 80, 0.15)` pas `rgba(0, 0, 0, 0.15)` sur un fond violet clair).
11. **Notifications système overlay sur du contenu critique** — un toast / snackbar qui couvre le bouton primaire bloque l'usage. Toujours réserver une zone safe en bas pour les CTAs, posée au-dessus du toast.
12. **Animations qui bloquent l'interaction** — une transition d'1.5s entre 2 écrans force l'attente. Maximum 300ms pour les transitions, 600ms pour les animations célébratoires (peak moments).
13. **Image lazy-loaded sans skeleton** — l'écran "saute" quand l'image charge. Toujours réserver l'espace (aspect-ratio CSS) + skeleton coloré.
14. **Status bar masquant le contenu** — `data-os-chrome="status-bar"` doit être respecté côté kit, et l'agent codeur wrap dans SafeArea (cf. UI Kit Contract §data-os-chrome).

---

## 7. Critères d'audit machine-readable

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
| PE-1 | Au moins un écran PIC identifié (résultat IA, niveau franchi, paywall passé, première création) avec polish max (micro-anim, badge, typo display) | `data-hint="peak-moment"` ou cell dédiée avec animation célébratoire |
| PE-2 | Au moins un écran FIN soigné par flow (résumé + nudge prochaine session, pas un crash vers le hub) | cells de fin de flow avec carte résumé + CTA secondaire "Reviens" |
| PE-3 | Effort visuel concentré sur PIC + FIN, pas saupoudré uniformément sur tous les écrans | inspection visuelle de la grille canvas du flow |
| STYLE-1 | Un seul style UI dominant choisi parmi le catalogue §5 (pas un mélange) et documenté dans GUIDELINES.md | grep dans `GUIDELINES.md` du kit pour le style nommé |
| STYLE-2 | Le style choisi est cohérent avec la niche (cf. mapping §4.1 + §5) — pas un style fashion / dating sur une app fintech | review niche × style choisi |
| AP-1 | Pas de gradient / blur excessif (max 1-2 endroits par écran) | comptage gradients dans `components.css` + grep `backdrop-filter` |
| AP-2 | Maximum 4 tailles + 3 poids typographiques par écran | grep `font-size` / `font-weight` distincts dans chaque cell |
| AP-3 | Espacement strict sur grille 4pt ou 8pt (pas de valeurs aléatoires type 13px / 27px) | grep `padding` / `margin` hors `var(--space-*)` |
| AP-4 | CTA primaires en zone thumb (tiers inférieur, pas en haut à droite) | inspection visuelle position des `.btn--primary` |
| AP-5 | Pas de slider pour des données précises (préférer steppers / input numérique) | grep `<input type="range">` ou `data-uses="ui:slider"` × contexte de la valeur |
| AP-6 | Valeurs prédominent sur leurs labels (font-size valeur ≥ font-size label) | inspection visuelle key/value display |
| AP-7 | Ombres tintées (pas gris/noir pur sur fond coloré) | grep `box-shadow.*rgba\(0` sur des cells avec fond non blanc |
| AP-8 | Transitions ≤ 300ms (cell → cell, sauf peak animations célébratoires ≤ 600ms) | grep `transition-duration` / `animation-duration` |
| AP-9 | Images avec espace réservé (aspect-ratio CSS) + skeleton avant chargement | grep `aspect-ratio` + `data-uses="ui:skeleton"` sur les images dynamiques |
