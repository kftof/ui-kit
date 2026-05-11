# Playbook 2 — Engagement & Viralité

Doctrine pour transformer l'usage post-onboarding en **boucle addictive** qui maintient l'utilisateur, génère de la viralité gratuite et favorise la rétention long-terme. À consulter dès qu'un kit contient les flows d'usage principal (post-onboarding, post-paywall).

Une "breakout app" se construit en protégeant le flux à tout prix : moins de friction, plus de moments de satisfaction (Aha moments) qui incitent au partage et à la notation.

Règles **actionables**, numérotées pour référence directe.

> ⚠️ **Hiérarchie de décision — le PRD prime sur le playbook**
>
> Les recommandations ci-dessous (streaks, push notifs, demande d'avis au pic émotionnel, etc.) sont des **patterns par défaut** d'apps haute rétention. **Si le PRD du projet en cours en contre-indique** (ex: produit B2B sans gamification, app éphémère sans rétention long terme), le PRD prime. Côté audit, un critère qui contredit le PRD est marqué `n/a` avec mention "contre-indication PRD §X" — **pas** `fail`.

> 🎨 **Rappel — on fait des MAQUETTES UI, pas du templating code**
>
> Aucun placeholder `{{...}}` visible dans le HTML. Valeurs variables (compteurs, contenus dynamiques) affichées concrètement + `data-hint="..."` pour signaler la nature paramétrable.

---

## 1. Core Loop — éliminer la friction d'accès à la valeur

### 1.1 Accès direct à la fonctionnalité principale
- Après l'onboarding (et/ou paywall), l'utilisateur est **projeté directement** dans le cas d'usage
- Pas d'écran "Bienvenue dans MyApp" entre l'onboarding et l'usage
- Pas de menu hub d'accueil si l'app a UNE fonctionnalité phare évidente

### 1.2 Élimination des menus et boutons intermédiaires
- Le moins de taps possible entre l'ouverture de l'app et l'action principale
- Si l'app est un jeu : pas d'écran titre, l'utilisateur joue immédiatement (pattern type "Swipe the Cat")
- Si l'app est un outil créatif : l'éditeur s'ouvre directement, prêt à recevoir un input
- Si l'app est un tracker : l'écran principal montre le tracker en cours, pas un dashboard de stats

### 1.3 Réduction des interruptions entre sessions
- Minimiser les pauses, transitions, écrans de "chargement plein écran" entre deux sessions d'usage
- Skeleton loaders au lieu de spinners full-screen
- Cache local pour que l'app paraisse instantanée même en sortie de background

### 1.4 Pas de modals bloquants au lancement
- Pas de "Notez l'app" / "Rejoignez Discord" / "Nouvelle feature !" en cold start
- Si quelque chose doit être communiqué, le faire en bannière dismissable, pas en modal qui interrompt le core loop

---

## 2. Sharable Moments — viralité intégrée

### 2.1 Identifier l'écran de fierté utilisateur
- Le moment où l'utilisateur est **le plus fier** de son résultat — score élevé, création réussie, statistique flatteuse, streak important
- Ce moment est unique par produit : à identifier explicitement avant de coder le kit
- **Une seule** screen par flow doit porter ce rôle ; le tagger en `data-hint="sharable-moment"` pour signaler aux outils de code-gen

### 2.2 Rendre l'écran "screenshot worthy"
- Style visuel **distinctif** (typographie large, composition centrée, palette accent saturée)
- Background ou cadre qui survit au crop d'iOS Screenshots / Instagram Stories
- Pas de chrome OS visible (status-bar masquée ou intégrée dans le design)

### 2.3 Branding explicite sur le sharable moment
- Logo + nom de l'app **visibles** sur l'écran de fierté (typiquement en watermark coin bas-droit ou en signature header)
- Quand l'utilisateur partage l'écran, le logo voyage avec — publicité gratuite
- Le branding ne doit pas dominer le contenu, juste être présent et lisible

### 2.4 Bouton "Partager" natif dans le sharable moment
- Bouton dédié au partage avec `data-uses="native:share-sheet"`
- Pré-rempli avec un texte + l'image de l'écran + un deep link App Store
- Pas dans un sub-menu ou caché — bouton primaire visible

---

## 3. Review & Aha Moment — timing de la demande d'avis

### 3.1 Demande de notation TOUJOURS au pic émotionnel
- Jamais en cold start, jamais sur un écran de paramètres, jamais après un crash
- Toujours après un succès **confirmé** côté utilisateur : streak validé, première création réussie, premier objectif atteint
- Implémenter via l'API native (`StoreKit.SKStoreReviewController` iOS / `In-App Review API` Android) — pas un faux dialog custom

### 3.2 Limiter à 1 demande par session, max 3 par an
- L'OS limite à 3 demandes par an de toute façon — ne pas gaspiller les opportunités
- Ne demander qu'après le 2e ou 3e événement positif (pas le premier — risque d'agacement si l'utilisateur n'est pas encore convaincu)

### 3.3 Pas de pré-screening manuel
- Pas de dialog "Vous aimez l'app ? [Oui → review] [Non → form de feedback]" — interdit par les guidelines App Store récentes
- Demander directement la review native ; pour le feedback critique, lien dédié dans Settings

---

## 4. Progression & Récompense — transformer l'usage en jeu

### 4.1 Écran de débloquage / récompense après usage
- Immédiatement après l'usage de la fonctionnalité principale, écran qui matérialise un **gain** (points, badge débloqué, niveau progressé)
- Animation rapide (1-2s max) pour ne pas casser le core loop
- Le gain doit être **mesurable** et cumulable (pas "Bravo !" sans contexte)

### 4.2 Écran de collection / historique
- Pour les outils créatifs : galerie des créations passées
- Pour les jeux : collection de items débloqués
- Pour les trackers : historique de progression sur N jours/semaines
- Affiche la **valeur accumulée** pour que l'utilisateur visualise son investissement (et hésite à churn)

### 4.3 Streaks visuels
- Compteur de jours/sessions consécutifs visible sur l'écran principal
- Notification dédiée si streak en danger (pattern Duolingo, Snapchat)
- L'aversion à perdre un streak est un puissant moteur de rétention

### 4.4 Variabilité du reward
- Pattern type "variable reward schedule" : ne pas donner exactement la même récompense à chaque fois
- L'incertitude (petite ou grosse récompense ?) génère l'engagement (dopamine schedule)
- À utiliser avec parcimonie — ne pas tomber dans le casino design

---

## 5. Hook Model — boucle d'habitude complète

Cadre théorique structurant pour designer une boucle d'engagement répétée qui transforme l'usage occasionnel en habitude. Synthèse du modèle de Nir Eyal ("Hooked"), enrichie du Fogg Behavior Model. Les §1-4 ci-dessus traitent des composants ; cette section les **articule en boucle**.

Une boucle complète = 4 phases enchaînées. Si une phase manque ou est faible, la boucle ne se forme pas → l'utilisateur churn après quelques sessions.

### 5.1 Phase 1 — Trigger (déclencheur)
- **Triggers externes** : éléments qui ramènent l'utilisateur dans l'app depuis l'extérieur
  - Push notification (la plus directe — à manier avec parcimonie pour éviter le ratio de désinstallation)
  - Email transactionnel (résumé hebdo, milestone atteint)
  - Badge sur l'icône d'app
  - Widget home screen iOS / Android
  - Live Activity (iOS Dynamic Island) pour les opérations en cours
- **Triggers internes** : émotion / état mental qui pousse l'utilisateur à ouvrir l'app **sans rappel externe**
  - Émotion source typique : ennui (TikTok), anxiété (méditation), FOMO (réseaux sociaux), curiosité (news), accomplissement (fitness tracker), connexion (chat)
  - L'objectif design : **mapper l'émotion source du user** et structurer le core loop pour la résoudre rapidement
  - Une app qui n'a que des triggers externes ne devient jamais une habitude
- **Transition externe → interne** : c'est le but à 3-6 mois d'usage. Si l'utilisateur n'ouvre l'app que sur notification → trigger interne raté, churn imminent

### 5.2 Phase 2 — Action (comportement attendu)
- L'action est **le comportement le plus simple** que l'utilisateur fait en anticipation d'un reward
- Fogg Behavior Model : `Behavior = Motivation × Ability × Trigger`
  - Les 3 doivent converger **au même moment** pour qu'une action se produise
  - Si motivation faible → augmenter ability (rendre l'action plus simple)
  - Si ability bloquée → augmenter motivation (rappeler le bénéfice)
  - Si trigger absent → installer un trigger (notification, widget, badge)
- Exemples d'actions canoniques :
  - Swipe (Tinder / TikTok / Snap)
  - Tap pour révéler (Inbox / scratch ticket)
  - Pull-to-refresh (Instagram / Twitter)
  - Open app + glance (Météo, Bourse)
  - Snap photo (BeReal, Snapchat)
- **Règle design** : 1 action principale par session, accessible en ≤ 2 taps depuis cold start (cf. §1.2)

### 5.3 Phase 3 — Variable Reward (récompense variable)
- L'incertitude sur la nature ou la magnitude de la récompense **active le système dopaminergique** (anticipation > satisfaction)
- 3 types de variable reward (Nir Eyal) :
  - **Reward of the tribe** : validation sociale (likes, commentaires, streak partagé, leaderboard)
  - **Reward of the hunt** : matériel / information (nouvelle recette, nouvelle photo dans le feed, nouveau score, nouvelle stat)
  - **Reward of the self** : maîtrise / accomplissement (passer un niveau, finir une session, voir sa progression)
- **Reinforcement schedule** : pas la même récompense à chaque fois. Patterns :
  - Variable ratio (la majorité des sessions = petit reward, occasionnellement gros reward) — pattern le plus addictif
  - Variable interval (le timing du reward varie — pull-to-refresh ramène parfois rien, parfois 10 items)
- ⚠️ Limite éthique : la variable reward peut basculer en design manipulatoire. **Aligner le reward avec un bénéfice utilisateur réel** (pas juste un endorphin hit creux)

### 5.4 Phase 4 — Investment (investissement utilisateur)
- L'utilisateur investit quelque chose dans l'app **après** le reward : temps, data, effort, social capital, money
- Cet investissement améliore l'expérience à la prochaine session ET augmente le coût de switch
- Exemples canoniques :
  - **Data** : ajouter ses préférences, importer ses contacts, configurer son profil
  - **Content** : créer une recette, poster une photo, enregistrer un favori
  - **Social capital** : suivre des utilisateurs, partager un score, inviter un ami
  - **Reputation** : score, level, badges, streaks (perdre un streak = perte d'investissement réel)
  - **Money** : abonnement payant (le paiement lui-même = investissement, l'utilisateur défend sa décision)
- **L'investissement charge le prochain trigger** : ajouter une recette à mes favoris → notification "Nouvelle recette ajoutée à ta collection cette semaine" → re-trigger

### 5.5 Boucle complète vs hook brisé
- **Boucle complète** : trigger interne (ennui) → action simple (open + swipe) → reward variable (vidéo qui résonne) → investment (like, follow, partage) → next trigger chargé (notification "X nouveaux contenus comme celui-ci")
- **Hook brisé** typique : trigger fort (push agressif) → action simple → **pas de reward variable** (toujours le même contenu) → **pas d'investment** (rien à perdre, rien à construire) → désinstallation après 3 sessions
- Audit récurrent : pour chaque flow récurrent du produit, écrire les 4 phases et identifier laquelle est faible ou manquante

---

## 6. Rétention & Maintenance "boring"

### 5.1 Écran Settings standardisé
- Modèle de paramètres réutilisable d'une app à l'autre — gain de temps + cohérence
- Sections canoniques : Compte, Abonnement, Notifications, Apparence, Confidentialité, Support, À propos, Déconnexion
- Pas d'inventivité visuelle dans les Settings — l'utilisateur s'attend à un pattern connu

### 5.2 Onglet "Abonnement" dédié
- Permet à l'utilisateur de voir son plan actuel, date de renouvellement, lien vers la gestion App Store
- Bouton "Restore Purchases" visible (obligatoire pour passer la review App Store)
- Cancel intégré ou redirection claire vers Apple/Google Subscriptions

### 5.3 Empty states actionnables
- Écran vide (pas encore de création / pas encore de friends / pas encore de history) → toujours un CTA pour produire la première entrée
- Pas un texte triste ("Pas de données") — un guide vers la prochaine action

### 5.4 Mises à jour régulières — petites + fréquentes
- Pas de "grosse release tous les 6 mois" — préfère N petites améliorations par semaine basées sur le feedback utilisateur
- Patch notes courts visibles dans l'app (pas seulement dans App Store)
- Adresse-toi au feedback utilisateur explicite ("Vous nous avez demandé X, voici X")

---

## 7. Critères d'audit machine-readable

Liste binaire utilisée par `ui-kit-audit` pour évaluer un kit. Chaque critère = `pass` / `fail` / `n/a`.

| ID | Critère | Où chercher |
|---|---|---|
| CL-1 | Le post-onboarding ouvre directement sur la fonctionnalité principale (pas de hub intermédiaire) | premier écran post-onboarding dans flows |
| CL-2 | Le core loop fait ≤ 2 taps depuis cold start jusqu'à l'action principale | comptage taps depuis splash → premier usage |
| CL-3 | Pas de modal bloquant au lancement (rate, news, paywall répété) | grep modals sur splash / home |
| CL-4 | Skeleton loaders utilisés au lieu de spinners full-screen | data-uses="ui:skeleton" sur loading states |
| SH-1 | Au moins un écran tagué "sharable-moment" identifié | grep `data-hint="sharable-moment"` ou équivalent |
| SH-2 | L'écran sharable porte le branding (logo + nom de l'app visibles) | présence logo asset dans sharable cell |
| SH-3 | Bouton partage natif (`data-uses="native:share-sheet"`) présent sur le sharable | grep dans la cell concernée |
| SH-4 | Pas de status-bar visible sur l'écran sharable (composition propre au crop) | data-os-chrome status-bar absent/masqué |
| RV-1 | Demande de review native (`data-uses="native:request-review"`) après un événement positif | grep dans flows post-success |
| RV-2 | La demande de review n'est PAS placée sur splash / home / settings | absence dans ces écrans |
| RV-3 | Pas de dialog custom de pré-screening "vous aimez ?" avant la review native | grep absence de ce pattern |
| PR-1 | Écran de récompense / débloquage après l'action principale | cell post-action avec animation gain |
| PR-2 | Écran de collection / historique présent (visualise l'investissement utilisateur) | flow dédié `history` ou `collection` |
| PR-3 | Si app à usage récurrent : streak / compteur de progression visible sur écran principal | grep streak / day counter |
| RM-1 | Écran Settings respectant le modèle standard (Compte/Abo/Notifs/Apparence/Confidentialité/Support/About) | structure de `settings.html` |
| RM-2 | Section "Abonnement" avec "Restore Purchases" visible | grep restore purchases / restaurer achats |
| RM-3 | Empty states avec CTA d'action (pas juste un texte vide) | cells "empty" contiennent un bouton |
| HK-1 | Trigger interne identifié (émotion source) documenté dans le kit ou via `data-hint` sur l'écran d'entrée | grep `data-hint=".*trigger.*"` / mention émotion source dans GUIDELINES.md ou README.md du kit |
| HK-2 | Au moins un trigger externe configuré (push notification, badge icon, widget, live activity) | grep `data-uses="native:push-notification"` ou `data-uses="native:widget"` / `data-uses="native:live-activity"` dans le kit |
| HK-3 | Action principale du core loop atteignable en ≤ 2 taps depuis cold start (cf. CL-2) | comptage taps du flow entrée → core action |
| HK-4 | Variable reward implémenté sur l'action principale (pas le même résultat à chaque fois) | présence d'au moins 2 variantes / states dans la cell post-action (jackpot, neutre, gros gain…) |
| HK-5 | Au moins un mécanisme d'investment user (favoris, profile data, social capital, streaks…) présent dans le kit | grep `data-uses="ui:favorite"` / cells de profil avec inputs / flow `streaks` ou `collection` |
| HK-6 | L'investment charge le prochain trigger (ex: ajout favori → notification "Nouveau X dans tes favoris") — documenté dans GUIDELINES.md | grep dans GUIDELINES.md des phrases "déclenche" / "trigger" en lien avec les actions d'investment |
