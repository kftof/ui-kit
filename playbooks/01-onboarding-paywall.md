# Playbook 1 — Onboarding & Paywall stratégiques

Doctrine pour transformer un onboarding + paywall en **tunnel de vente** plutôt qu'en formalité technique. Un onboarding bien conçu, couplé à un paywall optimisé, peut générer jusqu'à 75 % des revenus d'une app. À consulter dès qu'un kit contient un flow `onboarding` ou un écran `paywall`.

Règles **actionables**, numérotées pour référence directe (§1.1, §4.2…). Pas de templates HTML — c'est le rôle du `ui-kit-creator` de matérialiser ces règles dans des écrans concrets adaptés au produit.

> ⚠️ **Hiérarchie de décision — le PRD prime sur le playbook**
>
> Les valeurs concrètes mentionnées ici (durée d'onboarding 10-15 min, trial 3 jours, ≤ 3 options de prix, etc.) sont des **recommandations par défaut** issues d'apps haute conversion. **Le PRD du projet en cours prime systématiquement** : si le brief dit "trial 7 jours" ou "pas de trial du tout", c'est le PRD qui gagne, pas le playbook.
>
> Idem côté audit : un critère qui contredit explicitement le PRD doit être marqué `n/a` avec mention "contre-indication PRD §X", **pas** `fail`. Le playbook est un guide, pas une contrainte juridique.

> 🎨 **Rappel — on fait des MAQUETTES UI, pas du templating code**
>
> Quand une règle évoque une donnée variable (prix, réponse user, durée), la **maquette HTML doit contenir une valeur concrète** (ex: "9,99 €/mois", "Réponse type : *gagner du temps*"). La nature variable est signalée aux agents de code-gen via `data-hint="..."` (qui ne s'affiche pas visuellement). **Aucun placeholder de templating type `{{xxx}}` ne doit apparaître dans le HTML visible** — c'est un bug d'agent qui produit une maquette inutilisable.

---

## 1. Onboarding — Introduction (cadrage psychologique)

### 1.1 Poser problème + solution dans les 3 premiers écrans
- Écran 1 = **problème** vécu par l'utilisateur, formulé en langage utilisateur (pas en jargon produit)
- Écran 2 = **solution** que l'app apporte, en une phrase
- Écran 3 = **preuve** que ça marche (chiffre, résultat attendu, témoignage)
- Pas de logo "welcome to MyApp" qui ne dit rien sur la valeur

### 1.2 Moment "Aha" en moins d'une minute
- Statistique choquante personnalisée OU prise de conscience immédiate
- Doit s'appuyer sur un fait vérifiable (pas une promesse marketing creuse)
- Exemples de tonalité : "Vous passerez 16 ans de votre vie sur votre téléphone" / "92 % des gens abandonnent leurs résolutions en 2 semaines"

### 1.3 Questions personnelles → auto-conviction
- 3 à 5 questions qui forcent l'utilisateur à **se convaincre lui-même** qu'il a besoin du produit
- Format strict : choix multiple, slider, échelle 1-5, toggle — **jamais** de texte libre (friction tueuse de conversion)
- Question type : "Combien de fois par jour ouvres-tu cette app concurrente ?" (le user formule le problème lui-même)

### 1.4 Effet miroir — réinjecter les réponses dans les écrans suivants
- Chaque réponse captée doit être **visible** dans au moins un écran ultérieur
- Format : "Nous préparons votre plan pour `<réponse user>`"
- Donne l'illusion d'une expérience personnalisée — coût zéro côté backend (juste du templating client-side)

### 1.5 Aversion à la perte — assumer un onboarding LONG (recommandation 10-15 min, à arbitrer selon PRD)
- Contre-intuitif mais vérifié : un onboarding long **triple à quintuple** le taux de conversion
- L'utilisateur qui a investi 12 minutes ne veut pas avoir "perdu son temps" — il essaie le produit, paye plus facilement
- Implication design : pas de "skip" visible sur les écrans intermédiaires (la friction est volontaire)
- **Si le PRD stipule un onboarding court** (ex: outil pro où la friction est interdite par la nature du produit), respecter le PRD — l'aversion à la perte n'est qu'un levier parmi d'autres

---

## 2. Onboarding — Climax (engagement émotionnel)

### 2.1 Essai de la fonctionnalité phare PENDANT l'onboarding
- L'utilisateur doit **tester** la valeur principale avant le paywall, pas seulement la voir promise
- Si l'app fait de la génération IA, déclencher une génération avec ses inputs réels
- Si l'app fait du tracking, créer la première entrée immédiatement
- Le wow-moment ne peut pas être différé au post-achat

### 2.2 Demande d'avis App Store au pic émotionnel
- À déclencher pile au moment où l'utilisateur est **le plus satisfait** (résultat IA réussi, premier streak, premier wow)
- Pas trop tôt (user pas convaincu) ni trop tard (émotion retombée)
- Critique pour ASO et preuve sociale — chaque review = visibilité gratuite

---

## 3. Onboarding — Conclusion (transition vers paywall)

### 3.1 Résumé du parcours utilisateur
- Avant le paywall, écran qui résume **ce que l'utilisateur a déclaré vouloir accomplir** (rebrousse-poil de §1.4)
- Lier à une **habitude concrète** mesurable ("Prends cette habitude en 30 jours", pas "améliore ta vie")

### 3.2 Ancrage de prix
- Comparer le coût mensuel à une **dépense banale familière** ("Le prix d'un café par mois")
- Pas une comparaison entre offres concurrentes (génère de la comparaison défavorable)
- Le bénéfice doit être **émotionnel** ("la paix spirituelle"), pas fonctionnel ("X features")

### 3.3 Engagement explicite avant paywall
- Question directe : "À quel point es-tu engagé à réussir ?"
- Échelle 1-5 ou choix multiple ("pas du tout / un peu / vraiment / extrêmement")
- L'utilisateur qui répond "extrêmement engagé" se conditionne lui-même à passer le paywall

---

## 4. Paywall — règles structurelles

### 4.1 Structure de prix Weekly + Annual (pas Lifetime)
- **Weekly** : lever la barrière d'entrée, tester la rétention, capter les sceptiques
- **Annual** : meilleure valeur perçue, monter le MRR, lock-in 12 mois
- **Abandonner les lifetime deals** : préfère la croissance du revenu récurrent (MRR) au cash one-shot
- Maximum 2-3 options visibles (paralysie de décision au-delà)

### 4.2 Trial gratuit (recommandation 3 jours, à arbitrer selon PRD)
- Réduit la peur de l'engagement ("j'essaie sans risque")
- 70 % des activations trial se produisent **juste après l'onboarding** — fenêtre étroite, ne pas la rater
- Durée recommandée par défaut : **3 jours** (convertit mieux en général). **7 jours** acceptable pour produits à courbe d'apprentissage longue (fitness, méditation, écriture)
- **Si le PRD dit "pas de trial"** ou stipule une durée précise (ex: 14 jours pour B2B) : respecter le PRD, ne pas surcharger avec la recommandation par défaut

### 4.3 Rappel "1 jour avant fin trial" (case toggle visible)
- Option proactive permettant à l'utilisateur de **recevoir un rappel** la veille de la facturation
- Levier de conversion majeur ET honnête (transparence pure)
- À cocher par défaut OU à proposer en bouton dédié — pas caché dans les settings

### 4.4 Preuve sociale embarquée
- Badges : "App #X dans la catégorie Y", "Choisi par Z utilisateurs"
- Avis clients courts : 1-3 max sur le paywall, sélectionnés pour adresser des objections spécifiques
- À placer **avant** le bouton CTA, pas en dessous

### 4.5 Simplicité visuelle et transparence
- **Une œuvre d'art**, pas un formulaire administratif — style visuel polished, parfois vidéo de fond ou animation
- Bouton de fermeture **visible** (jamais caché ou trop discret — érode la confiance)
- Prix **non-ambigus** : afficher le prix total annuel ("$59.99/an" pas seulement "$4.99/mois facturé annuellement" enterré en small text)
- Pas de dark patterns (false urgency timers, faux "sold out", etc.)

---

## 5. Patterns Paywall 2026 (mises à jour récentes)

Évolutions observées sur les apps à très haute conversion en 2025-2026. À considérer en complément des règles fondamentales §4 (qui restent valides).

### 5.1 Trial sur l'annual uniquement (pas sur le weekly)
- Les apps haute conversion 2026 déplacent le trial **exclusivement** sur le plan annual
- Supprimer le trial sur le weekly augmente l'ARPU : les utilisateurs prêts à payer en weekly n'ont pas besoin de trial pour s'engager
- Maintient le bénéfice "réduction de friction d'entrée" tout en privilégiant le revenu long-terme
- Adapter selon le produit : trial gratuit annuel + weekly direct **OU** trial gratuit weekly courte durée + annuel direct

### 5.2 Soft-then-Hard paywall (pattern à 2 étages)
- **Soft paywall** affiché pendant l'onboarding avec **"Skip" autorisé** — l'utilisateur entre dans l'app sans payer
- **Hard paywall** au premier usage de la fonctionnalité phare gated
- Capture les utilisateurs à 2 stades de trust différents — conversion combinée mesurée à 16.5% (vs 8-10% sur hard seul)
- Le "Skip" doit être **clairement visible** et fonctionnel — pas un dark pattern déguisé
- Implication design : il faut **deux écrans paywall** dans `flows/`, pas un seul

### 5.3 Tableau comparatif Free vs Pro **embedded** sur le paywall
- Devenu un must-have 2026 — apparaît sur les paywalls top-converting (fitness, education, productivity, IA, design)
- Format : 2 colonnes (Free / Pro), 5-8 lignes de features clés, ✓/✗ ou icônes
- Supprime l'objection "qu'est-ce que j'obtiens vraiment ?" sans forcer l'utilisateur à quitter le paywall pour vérifier
- À placer **après** la preuve sociale (§4.4), **avant** le bouton CTA primaire

### 5.4 Post-close welcome offer ciblée
- Quand l'utilisateur ferme le paywall sans convertir, déclencher un **second paywall** dans les 24h via push notification OU à la prochaine ouverture
- Format : discount limité (-20% / -30% sur l'annual) valide 24h, **uniquement** pour les non-converters
- Capture +10-15% d'ARPU sans dévaluer le plein tarif pour tous (pas broadcast)
- Implication design : prévoir un écran paywall variant "welcome-offer" séparé, avec badge discount visible

### 5.5 Personalized plan loader entre onboarding et paywall
- Écran intermédiaire "Analyzing your preferences..." (ou équivalent) avec animation de progression (2-4 secondes)
- Liste les inputs collectés ("Analyse de tes réponses... Calcul du plan optimal...") + social proof inline ("Comme 8 234 utilisateurs cette semaine")
- Rend la personnalisation **perçue** (sans nécessairement le faire côté backend)
- Réduit la friction d'achat en faisant percevoir l'effort fourni par l'app
- Implication kit : ajouter une cell "loading-personalization" entre la dernière question d'onboarding et le paywall

### 5.6 Localization / pricing PPP-adjusted
- Pricing ajusté à la parité de pouvoir d'achat (Purchasing Power Parity) selon le pays — un abonnement à $9.99/mois US ne se vend pas à $9.99 en Turquie ou Argentine
- Apps en croissance 2026 : Europe (+18% YoY), Japon, Mexique, Turquie
- Implication kit : afficher un **prix concret** dans la maquette HTML (ex: "9,99 €/mois" pour le marché FR) ET poser `data-hint="price-PPP-adjusted-per-market"` sur la cell du prix. C'est cet attribut (non visible) qui signale à l'agent de code-gen de brancher le pricing sur le SDK paywall (RevenueCat / Adapty / StoreKit2). **Surtout pas** de placeholder `{{...}}` visible dans la maquette — c'est un bug d'agent.
- Localiser **aussi** les wordings (pas seulement le prix) selon le marché

### 5.7 AI-powered personalization du paywall
- Trial length adapté selon engagement détecté pendant l'onboarding (user très engagé → trial court 3j, user hésitant → trial 7j)
- Messaging du paywall adapté selon les features explorées pendant l'onboarding (user a interagi avec feature X → paywall met X en avant)
- Pricing dynamique optionnel selon segment user (à manier avec éthique — la transparence reste prioritaire)
- Implication kit : poser `data-hint` métier sur le paywall pour signaler que le messaging est dynamique (ex: `data-hint="messaging personnalisé selon top-feature explorée onboarding"`) — pas une variante statique

---

## 6. Honnêteté & transparence — patterns interdits

Les patterns persuasion ci-dessus (effet miroir, aversion à la perte, ancrage prix, social proof, etc.) sont **éthiques tant qu'ils s'appuient sur du réel**. Dès qu'on bascule dans l'invention ou la manipulation émotionnelle grossière, ça se voit, ça fait fuir, et ça nuit à la crédibilité du produit. Liste des patterns **interdits** par construction — à ne JAMAIS générer dans un onboarding ou un paywall.

### 6.1 Statistiques fake / citations d'institutions non-traçables
- ❌ "Selon une étude de l'INRAé, 73 % des Français…"
- ❌ "Des chercheurs de Stanford ont prouvé que…"
- ❌ "92 % des utilisateurs ont vu une amélioration en 7 jours" (chiffre inventé pour faire crédible)

**Règle** : si tu cites une stat, elle doit être **vérifiable** (lien source dans le kit ou GUIDELINES.md). Si tu ne peux pas la sourcer, **reformule sans chiffre** :
- ✅ "Beaucoup de gens passent trop de temps sur leur téléphone" (vague mais honnête)
- ✅ "Tu n'es pas seul à vouloir reprendre le contrôle" (universel, pas chiffré)

Citer une institution scientifique fictivement = perte de crédibilité + risque légal selon la juridiction.

### 6.2 Emojis émotionnels manipulateurs (💔 😢 🚨 🔥)
- ❌ Emoji 💔 sur un écran de cadrage "Tu ressens X ?"
- ❌ Emoji 😢 dans une question d'onboarding ("Tu te sens triste parfois ?")
- ❌ Emoji 🚨 ou 🔥 sur un titre de paywall ("URGENT — Offre limitée")

**Règle** : pas d'emojis émotionnels sur les écrans de cadrage / conversion. Les emojis acceptables sont **décoratifs neutres** :
- ✅ Icons SVG dédiés (avec `data-asset`) pour les illustrations
- ✅ Emojis très contextuels et neutres (✓, →, ★) en accent typo si vraiment nécessaire

Les emojis émotionnels sont une manipulation grossière qui se voit. Préférer une illustration SVG soignée (cf. `ds/assets/illustrations/`) pour matérialiser le ressenti.

### 6.3 Échelles d'engagement grandiloquentes
- ❌ "À quel point es-tu engagé ?" → choix : ["Pas du tout", "Un peu", "Beaucoup", "Plus que tout", "À 200%"]
- ❌ "Vraiment ?" / "Absolument ?" / "Tu en es sûr ?" en boucle

**Règle** : échelle factuelle, sobre, en 3 ou 5 niveaux **mesurables** :
- ✅ "Peu engagé / Moyennement engagé / Très engagé" (3 niveaux)
- ✅ Échelle numérique 1-5 ou 1-10 sans wording grandiloquent

L'utilisateur se met en retrait quand il sent la formulation ridicule — ce qui ruine le bénéfice de la §3.3 (engagement avant paywall).

### 6.4 Countdown 24h / 1h / 10min sur le paywall (false urgency)
- ❌ Timer "Cette offre expire dans 23h 47m 12s" qui reset à chaque ouverture
- ❌ Badge "FLASH SALE — last 5 minutes!" qui ne change jamais
- ❌ "Plus que 3 places disponibles !" sur un produit SaaS infini

**Règle** : pas de countdown sur le paywall **sauf** si l'offre est **réellement limitée** (ex: code promo Black Friday avec vraie deadline, post-close offer 24h §5.4 qui expire vraiment). Si tu poses un timer, il doit refléter une vraie limite, pas une pression artificielle.

Le pattern 5.4 (post-close welcome offer) **est** une exception légitime au "pas de countdown" — mais l'offre doit vraiment expirer côté backend.

### 6.5 Social proof inventé
- ❌ "Rejoins 12 458 utilisateurs cette semaine" sans avoir 12 458 users
- ❌ "★★★★★ — Marie, Paris" en faux testimonial
- ❌ "4.9 ⭐ sur 8 932 avis" sans avoir 8 932 avis réels
- ❌ "Comme vu dans Le Monde / TechCrunch / Forbes" sans avoir été publié dedans

**Règle** : tout social proof posé sur un écran doit être **vérifiable**. Sources acceptables :
- ✅ Reviews App Store / Play Store (capture screenshot d'un vrai avis, ou intégration via API)
- ✅ Press mentions réelles avec lien
- ✅ User counts approximatifs sourcés ("Plus de 10 000 téléchargements depuis le lancement" si vrai)

Si tu n'as pas encore de social proof : ne pas en inventer. Préférer un message direct ("Nouveau venu — sois parmi les premiers à essayer") ou retirer la cell entièrement.

### Pourquoi ces 5 patterns sont interdits

Les utilisateurs mobile en 2026 reconnaissent ces patterns à 10 mètres — ils sont devenus tellement omniprésents (apps IA génériques, dropshipping, scammy fitness apps) que leur présence **signale** à l'utilisateur "cette app est suspecte / probablement low quality / probablement IA-gen". Effet inverse du but recherché : au lieu de convaincre, ils repoussent les utilisateurs avertis (ceux qui paient le plus).

L'honnêteté éthique du produit **est** un argument de conversion. Les apps qui assument leur jeunesse ("Nouveau venu — pas encore de testimonial, on construit avec toi") convertissent mieux que celles qui inventent.

---

## 7. Critères d'audit machine-readable

Liste binaire utilisée par `ui-kit-audit` pour évaluer un kit. Chaque critère = `pass` / `fail` / `n/a`.

| ID | Critère | Où chercher |
|---|---|---|
| OB-1 | Problème + solution exprimés dans les 3 premiers écrans onboarding | `flows/*onboarding*/*.html` cells 1-3 |
| OB-2 | Moment Aha (stat / prise de conscience) présent en début d'onboarding | grep texte choc / chiffre dans cells 1-3 |
| OB-3 | Au moins 3 questions personnelles (choix multiple / échelle) | cells avec form / radio / range dans onboarding |
| OB-4 | Effet miroir : réponses user réinjectées dans écrans suivants | la cell qui réutilise une réponse contient une **valeur d'exemple concrète** (ex: phrase type "*gagner du temps*" reprise de la question §1.4) ET porte `data-hint="reuse:answer-N"` qui signale aux agents de code-gen quelle réponse réinjecter. **Aucun `{{answer_N}}` visible** dans la maquette. |
| OB-5 | Pas de skip visible sur les écrans intermédiaires de l'onboarding | grep "skip" / "passer" / "ignorer" dans onboarding |
| OB-6 | Essai de la fonctionnalité phare AVANT paywall | cell onboarding avec interaction réelle (pas démo statique) |
| OB-7 | Demande d'avis App Store au pic émotionnel (post-wow) | cell avec `data-uses="native:request-review"` ou équivalent |
| OB-8 | Écran de résumé qui réutilise les réponses utilisateur | cell avant paywall qui templating les answers |
| OB-9 | Ancrage de prix à une dépense banale ("café", "abonnement journal") | texte paywall |
| OB-10 | Question d'engagement explicite avant paywall | cell "à quel point engagé" avec échelle |
| OB-11 | Personalized plan loader entre dernière question onboarding et paywall ("Analyzing your preferences" + social proof inline) | cell intermédiaire avec progression animée + listing inputs collectés |
| PW-1 | Structure de prix Weekly + Annual (≤ 3 options visibles) | paywall cell — count des options |
| PW-2 | Aucun lifetime deal | grep "lifetime" / "à vie" |
| PW-3 | Trial gratuit proposé sur le paywall **OU** absence justifiée par le PRD (ex: produit B2B sans trial, freemium pur, etc.) | grep "trial" / "essai gratuit" sur paywall ; si absent, vérifier le PRD pour justification — `n/a` si PRD contre-indique |
| PW-4 | Toggle "rappel 1 jour avant fin trial" visible | cell paywall avec checkbox / switch dédié |
| PW-5 | Preuve sociale (badges / avis) AVANT le CTA | placement avis/badges en haut du paywall |
| PW-6 | Bouton de fermeture visible (pas caché) | grep close button / X icon avec taille tap ≥ 44px |
| PW-7 | Prix total affiché clairement (pas enterré en small text) | font-size du prix annuel ≥ font-size body |
| PW-8 | Aucun dark pattern (false urgency, faux sold out) | absence de timers / countdown / "X places restantes" |
| PW-9 | Trial sur l'annual uniquement (pas en doublon sur weekly + annual) | cell paywall : option trial gratuit affichée sur l'annual seul, weekly sans trial OU weekly avec trial court + annual direct |
| PW-10 | Soft-then-Hard paywall implémenté (2 écrans paywall : soft skippable pendant onboarding + hard sur usage feature) | présence de 2 cells paywall : `paywall-soft` (avec "Skip" visible) + `paywall-hard` |
| PW-11 | Tableau comparatif Free vs Pro embedded sur le paywall (2 colonnes, 5-8 features ✓/✗) | cell paywall contient `.feature-table` ou `<table>` avec colonnes Free/Pro |
| PW-12 | Paywall variant "welcome-offer" (post-close discount 24h ciblé) | cell `paywall-welcome-offer` avec badge discount + countdown 24h |
| PW-13 | Pricing signalé comme **paramétrable** via `data-hint="price-PPP-adjusted-per-market"` sur la cell du prix (le HTML lui-même affiche un prix concret pour le marché principal — ex: "9,99 €/mois") | grep `data-hint=".*price.*PPP.*"` dans la cell paywall ; le prix affiché doit être un nombre concret (pas `{{...}}`) |
| PW-14 | `data-hint` signalant un messaging dynamique / AI-personalized sur le paywall (si concept produit le justifie) | grep `data-hint=".*personnalisé.*"` ou `data-hint=".*dynamic.*"` sur le paywall |
| HON-1 | **Aucune stat fake** non traçable (ex: "Selon l'INRAé, l'IFOP, Stanford…" sans source réelle) sur les écrans onboarding/paywall | `grep -rE '(INRAé\|IFOP\|Stanford\|Harvard\|Oxford\|MIT)' flows/*onboarding*/ flows/*paywall*/ "UI Kit.html"` → 0 match (ou présence d'une source vérifiée dans GUIDELINES.md) ; grep également des `%` non sourcés dans le texte |
| HON-2 | **Aucun emoji émotionnel manipulateur** (💔 😢 🚨 🔥 😱) sur les écrans onboarding/paywall | `grep -rE '(💔\|😢\|🚨\|🔥\|😱\|😭)' flows/*onboarding*/ flows/*paywall*/` → 0 match |
| HON-3 | **Échelle d'engagement sobre** (3-5 niveaux mesurables, pas de wording grandiloquent type "Plus que tout" / "À 200%") | inspection des options de l'écran §3.3 — aucun choix avec superlatif absolu |
| HON-4 | **Pas de countdown** sur le paywall principal (sauf welcome-offer §5.4 qui expire vraiment côté backend) | `grep -rE '(countdown\|expires\|expire-in\|hh:mm:ss)' flows/*paywall*/` → 0 match SAUF dans la cell `paywall-welcome-offer` qui a un `data-hint="real-deadline"` |
| HON-5 | **Aucun social proof inventé** (user counts sans source, faux testimonials, faux press mentions) sur les écrans onboarding/paywall | inspection des cells avec `.testimonial` / `.user-count` / `.press-mention` — chaque proof doit avoir un `data-hint="source:..."` qui pointe vers une source vérifiable (App Store review, lien press réel, etc.) OU être retiré |
