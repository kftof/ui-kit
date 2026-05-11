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

## 5. Critères d'audit machine-readable

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
| PW-1 | Structure de prix Weekly + Annual (≤ 3 options visibles) | paywall cell — count des options |
| PW-2 | Aucun lifetime deal | grep "lifetime" / "à vie" |
| PW-3 | Trial gratuit proposé (3 ou 7 jours) | grep "trial" / "essai gratuit" sur paywall |
| PW-4 | Toggle "rappel 1 jour avant fin trial" visible | cell paywall avec checkbox / switch dédié |
| PW-5 | Preuve sociale (badges / avis) AVANT le CTA | placement avis/badges en haut du paywall |
| PW-6 | Bouton de fermeture visible (pas caché) | grep close button / X icon avec taille tap ≥ 44px |
| PW-7 | Prix total affiché clairement (pas enterré en small text) | font-size du prix annuel ≥ font-size body |
| PW-8 | Aucun dark pattern (false urgency, faux sold out) | absence de timers / countdown / "X places restantes" |
