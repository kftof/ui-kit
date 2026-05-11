# Playbook 2 — Engagement & Viralité

Doctrine pour transformer l'usage post-onboarding en **boucle addictive** qui maintient l'utilisateur, génère de la viralité gratuite et favorise la rétention long-terme. À consulter dès qu'un kit contient les flows d'usage principal (post-onboarding, post-paywall).

Une "breakout app" se construit en protégeant le flux à tout prix : moins de friction, plus de moments de satisfaction (Aha moments) qui incitent au partage et à la notation.

Règles **actionables**, numérotées pour référence directe.

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

## 5. Rétention & Maintenance "boring"

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

## 6. Critères d'audit machine-readable

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
