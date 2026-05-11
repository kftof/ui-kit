# Playbook 1 — Onboarding & Paywall stratégiques

Doctrine pour transformer un onboarding + paywall en **tunnel de vente** plutôt qu'en formalité technique. Un onboarding bien conçu, couplé à un paywall optimisé, peut générer jusqu'à 75 % des revenus d'une app. À consulter dès qu'un kit contient un flow `onboarding` ou un écran `paywall`.

Règles **actionables**, numérotées pour référence directe (§1.1, §4.2…). Pas de templates HTML — c'est le rôle du `ui-kit-creator` de matérialiser ces règles dans des écrans concrets adaptés au produit.

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

### 1.5 Aversion à la perte — assumer un onboarding LONG (10-15 min)
- Contre-intuitif mais vérifié : un onboarding long **triple à quintuple** le taux de conversion
- L'utilisateur qui a investi 12 minutes ne veut pas avoir "perdu son temps" — il essaie le produit, paye plus facilement
- Implication design : pas de "skip" visible sur les écrans intermédiaires (la friction est volontaire)

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

### 4.2 Trial 3 jours par défaut
- Réduit la peur de l'engagement ("j'essaie sans risque")
- 70 % des activations trial se produisent **juste après l'onboarding** — fenêtre étroite, ne pas la rater
- Trial 7 jours acceptable pour des produits à courbe d'apprentissage longue (apps de fitness, méditation), mais 3 jours convertit mieux en général

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
- Implication kit : prévoir une variable `{{price_localized}}` dans le texte du paywall plutôt qu'un prix en dur — l'agent de code-gen branchera sur le SDK paywall (RevenueCat / Adapty / StoreKit2)
- Localiser **aussi** les wordings (pas seulement le prix) selon le marché

### 5.7 AI-powered personalization du paywall
- Trial length adapté selon engagement détecté pendant l'onboarding (user très engagé → trial court 3j, user hésitant → trial 7j)
- Messaging du paywall adapté selon les features explorées pendant l'onboarding (user a interagi avec feature X → paywall met X en avant)
- Pricing dynamique optionnel selon segment user (à manier avec éthique — la transparence reste prioritaire)
- Implication kit : poser `data-hint` métier sur le paywall pour signaler que le messaging est dynamique (ex: `data-hint="messaging personnalisé selon top-feature explorée onboarding"`) — pas une variante statique

---

## 6. Critères d'audit machine-readable

Liste binaire utilisée par `ui-kit-audit` pour évaluer un kit. Chaque critère = `pass` / `fail` / `n/a`.

| ID | Critère | Où chercher |
|---|---|---|
| OB-1 | Problème + solution exprimés dans les 3 premiers écrans onboarding | `flows/*onboarding*/*.html` cells 1-3 |
| OB-2 | Moment Aha (stat / prise de conscience) présent en début d'onboarding | grep texte choc / chiffre dans cells 1-3 |
| OB-3 | Au moins 3 questions personnelles (choix multiple / échelle) | cells avec form / radio / range dans onboarding |
| OB-4 | Effet miroir : réponses user réinjectées dans écrans suivants | présence de placeholders `{{answer_N}}` ou `data-hint="reuse:answer-N"` |
| OB-5 | Pas de skip visible sur les écrans intermédiaires de l'onboarding | grep "skip" / "passer" / "ignorer" dans onboarding |
| OB-6 | Essai de la fonctionnalité phare AVANT paywall | cell onboarding avec interaction réelle (pas démo statique) |
| OB-7 | Demande d'avis App Store au pic émotionnel (post-wow) | cell avec `data-uses="native:request-review"` ou équivalent |
| OB-8 | Écran de résumé qui réutilise les réponses utilisateur | cell avant paywall qui templating les answers |
| OB-9 | Ancrage de prix à une dépense banale ("café", "abonnement journal") | texte paywall |
| OB-10 | Question d'engagement explicite avant paywall | cell "à quel point engagé" avec échelle |
| OB-11 | Personalized plan loader entre dernière question onboarding et paywall ("Analyzing your preferences" + social proof inline) | cell intermédiaire avec progression animée + listing inputs collectés |
| PW-1 | Structure de prix Weekly + Annual (≤ 3 options visibles) | paywall cell — count des options |
| PW-2 | Aucun lifetime deal | grep "lifetime" / "à vie" |
| PW-3 | Trial gratuit proposé (3 ou 7 jours) | grep "trial" / "essai gratuit" sur paywall |
| PW-4 | Toggle "rappel 1 jour avant fin trial" visible | cell paywall avec checkbox / switch dédié |
| PW-5 | Preuve sociale (badges / avis) AVANT le CTA | placement avis/badges en haut du paywall |
| PW-6 | Bouton de fermeture visible (pas caché) | grep close button / X icon avec taille tap ≥ 44px |
| PW-7 | Prix total affiché clairement (pas enterré en small text) | font-size du prix annuel ≥ font-size body |
| PW-8 | Aucun dark pattern (false urgency, faux sold out) | absence de timers / countdown / "X places restantes" |
| PW-9 | Trial sur l'annual uniquement (pas en doublon sur weekly + annual) | cell paywall : option trial gratuit affichée sur l'annual seul, weekly sans trial OU weekly avec trial court + annual direct |
| PW-10 | Soft-then-Hard paywall implémenté (2 écrans paywall : soft skippable pendant onboarding + hard sur usage feature) | présence de 2 cells paywall : `paywall-soft` (avec "Skip" visible) + `paywall-hard` |
| PW-11 | Tableau comparatif Free vs Pro embedded sur le paywall (2 colonnes, 5-8 features ✓/✗) | cell paywall contient `.feature-table` ou `<table>` avec colonnes Free/Pro |
| PW-12 | Paywall variant "welcome-offer" (post-close discount 24h ciblé) | cell `paywall-welcome-offer` avec badge discount + countdown 24h |
| PW-13 | Prix paramétré (variable / placeholder), pas en dur — permet pricing PPP-adjusted | grep dans cell paywall : `{{price_localized}}` ou `data-hint="price localized PPP"` au lieu de "$9.99/mo" |
| PW-14 | `data-hint` signalant un messaging dynamique / AI-personalized sur le paywall (si concept produit le justifie) | grep `data-hint=".*personnalisé.*"` ou `data-hint=".*dynamic.*"` sur le paywall |
