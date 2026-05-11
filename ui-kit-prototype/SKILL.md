---
name: ui-kit-prototype
description: Génère un fichier `Prototype.html` autonome à la racine d'un UI kit HTML existant — vue interactive style Figma qui affiche un seul écran à la fois en grand, avec navigation cliquable entre cells via la convention `data-nav-target`, transitions temporisées via `data-auto-advance`, sidebar de tous les écrans, breadcrumb, prev/next, animations de transition. À invoquer pour transformer une grille canvas statique en prototype navigable de présentation/itération.
---

# UI Kit Prototype — vue interactive style Figma

Transforme un kit HTML statique (grille canvas avec phones côte-à-côte) en **prototype cliquable autonome**. Un seul écran s'affiche à la fois en grand, les CTA cliquables naviguent vers d'autres écrans selon la convention `data-nav-target`, et les écrans temporisés (splash, loading, processing) s'enchaînent automatiquement via `data-auto-advance`.

## 🎯 Quand utiliser ce skill

- Présentation user/stakeholder (parcourir le produit comme s'il existait)
- Validation de parcours (vérifier que tous les flows tiennent debout, pas d'écran orphelin)
- Itération design rapide (modifier un Flow → re-générer le Prototype → re-tester)
- Pré-handoff dev (le dev voit le parcours complet, pas juste les écrans isolés)

## 📋 Inputs requis

**Obligatoire** :
1. **Path du kit** — racine d'un kit produit par `ui-kit-creator` (avec `ds/`, `flows/`, `UI Kit.html`)

**Optionnel** :
2. **Écran d'entrée** (par défaut : premier flow lexicographique × première page × premier état)
3. **Mode d'affichage** : `desktop` (sidebar visible, par défaut) ou `presentation` (fullscreen, sidebar masquable via touche `S`)

## 📋 Lecture du PRD — recommandée (non bloquante pour ce skill)

Contrairement aux autres skills `ui-kit-*`, le prototype **génère pas de contenu produit** — il assemble juste les écrans existants en navigation cliquable. Donc la lecture du PRD n'est pas bloquante.

Cependant, si un `PRD.md` existe dans le kit (`<kit-root>/PRD.md` ou variants), le lire rapidement aide à :
- Choisir un screen d'entrée pertinent (ex: si le PRD dit "wow moment = écran X", démarrer le prototype sur X)
- Documenter dans le rapport de génération les flows que le PRD considère comme prioritaires
- Détecter des écrans manquants côté kit que le PRD mentionne

Si pas de PRD : continuer normalement, le prototype reflète l'état actuel des flows du kit.

## 🧭 Workflow

### Phase 1 — Reconnaissance

Avant de générer, lire le kit pour extraire :

1. **Liste des flows** : `ls flows/` → `flow_dir` (kebab avec préfixe `NN-`)
2. **Liste des pages** par flow : `ls flows/<flow_dir>/*.html`
3. **Liste des états** par page : grep `data-screen-label` dans chaque HTML
4. **Graphe de navigation** : grep `data-nav-target` dans tous les Flows
5. **Graphe d'auto-advance** : grep `data-auto-advance` (sur `[data-screen-label]`) dans tous les Flows
6. **Détection de liens cassés** : pour chaque `data-nav-target="<flow>/<page>[:<state>]"` ET chaque `data-auto-advance="..."`, vérifier que la cible existe. Si non → `unknowns` warning + le clic devient no-op (ou affiche un toast "écran non trouvé") / l'auto-advance est ignoré
7. **Détection des écrans orphelins** : un écran que personne ne cible via `data-nav-target` **ni via `data-auto-advance`** (sauf le screen d'entrée) → warning informatif. ⚠️ Inclure `data-auto-advance` dans le calcul, sinon faux positifs sur les écrans only-reachable-by-timer.

Inventaire produit :
```
SCREENS = {
  "onboarding/welcome:default": {
    html: "...",
    flow_id: "onboarding",
    page_id: "welcome",
    state_id: "default",
    auto_advance: null
  },
  "onboarding/magic:processing": {
    html: "...",
    flow_id: "onboarding",
    page_id: "magic",
    state_id: "processing",
    auto_advance: { target: "onboarding/magic:result", delay: 3500 }
  },
  ...
}
NAVIGATION = [
  { from: "auth/login:default",   to: "tasks/list:default", trigger: ".btn submit-login" },
  { from: "tasks/list:default",   to: "tasks/detail:default", trigger: ".task-row" },
  { from: "tasks/detail:default", to: "tasks/list:default", trigger: ".nav-action[back]" },
  ...
]
```

### Phase 2 — Génération du `Prototype.html`

Fichier autonome posé à la racine du kit. Structure :

```html
<!doctype html>
<html lang="fr">
<head>
  <link rel="stylesheet" href="ds/tokens.css">
  <link rel="stylesheet" href="ds/components.css">
  <style>
    /* 1. Flow-styles inlinés (extraits + strippés des règles globales — voir Phase 4) */
    /* 2. CSS local du Prototype.html — sidebar, frame phone centré, animations */
    /* 3. Scroll fallback (NON NÉGOCIABLE — voir section dédiée) */
  </style>
</head>
<body>
  <aside class="proto-sidebar">
    <header>[Nom produit] — Prototype</header>
    <nav>
      <!-- liste des screens groupés par flow, click → navigation -->
      <!-- badge "auto" var(--accent) sur les screens avec auto_advance -->
    </nav>
    <footer>
      <button id="proto-back" title="Retour (B)">←</button>
      <button id="proto-prev" title="Précédent (←)">prev</button>
      <button id="proto-next" title="Suivant (→)">next</button>
      <button id="proto-toggle-sidebar" title="Masquer la sidebar (S)">⇤</button>
    </footer>
  </aside>

  <main class="proto-stage">
    <header class="proto-breadcrumb">
      <span class="proto-flow">tasks</span> ›
      <span class="proto-page">list</span> ›
      <span class="proto-state">default</span>
    </header>
    <div class="proto-frame" id="proto-frame">
      <!-- la cell .phone__screen courante injectée ici -->
    </div>
    <div class="proto-auto-advance" id="proto-auto-advance" hidden>
      <span class="proto-auto-advance__label">auto → <span id="proto-auto-target">…</span> dans <span id="proto-auto-eta">0.0</span>s</span>
      <div class="proto-auto-advance__bar"><div class="proto-auto-advance__fill" id="proto-auto-fill"></div></div>
      <button id="proto-auto-stop">stop</button>
    </div>
    <footer class="proto-meta">
      <span class="proto-screen-id">tasks/list:default</span>
      <span class="proto-warnings">[# liens cassés]</span>
    </footer>
  </main>

  <!-- Inventaire de tous les écrans inliné en JS (autonome, file:// safe) -->
  <script>
    const SCREENS = { /* …extraction du skill… */ };
    const NAVIGATION = [ /* … */ ];
    const ENTRY_SCREEN = "onboarding/welcome:default"; // ou autre selon param
    /* runtime JS — voir architecture ci-dessous */
  </script>
</body>
</html>
```

### Scroll fallback — règle CSS NON NÉGOCIABLE

Les flows définissent souvent `.phone__screen { overflow: hidden }` et délèguent le scroll à un wrapper interne `.phone__content { overflow-y: auto }`. Mais ce wrapper est régulièrement absent du markup. Conséquence : sur le Prototype, le contenu déborde silencieusement (boutons coupés en bas, pages-indicators invisibles).

Le shell Prototype DOIT inclure ce CSS d'override **après** l'inclusion des flow-styles, pour rendre le scroll robuste sans dépendre du markup parfait :

```css
/* Scroll fallback — fait partie du contrat du Prototype */
.proto-frame .phone__screen {
  overflow-y: auto !important;
  overflow-x: hidden !important;
}
.proto-frame .phone__screen > .frame-status,
.proto-frame .phone__screen > [data-os-chrome="status-bar"] {
  position: sticky; top: 0; z-index: 5;
  background: var(--bg);
}
.proto-frame .phone__screen > .frame-home,
.proto-frame .phone__screen > [data-os-chrome="home-indicator"] {
  position: sticky; bottom: 0; z-index: 5;
}
.proto-frame .phone__screen::-webkit-scrollbar { width: 0; }
```

Cette règle co-existe sans conflit avec les sous-zones scroll natives (ex: `.result-card__body { overflow: auto }`) qui continuent à fonctionner indépendamment.

### Architecture du runtime JS du Prototype.html

**État** :
```js
let currentScreen = ENTRY_SCREEN;
let history = [];                          // pour back button
const SCREENS_LIST = Object.keys(SCREENS).sort(); // ordre canonique pour prev/next
let pendingNavigation = false;             // anti-race pour navs rapides successives
let autoAdvanceTimer = null;               // timeout courant, à clearTimeout
let autoAdvanceRaf = null;                 // requestAnimationFrame de l'ETA visuel
```

**Rendu d'un écran (avec anti-race)** :
```js
function renderScreen(screenId, { animation = 'none' } = {}) {
  // Cancel-and-replace : neutralise toute nav en cours
  if (pendingNavigation) {
    // L'ancien setTimeout d'animation finira mais ne devra rien faire
    // (clear via closure flag ou via id de transition)
  }
  cancelAutoAdvance(); // toute nouvelle nav annule l'auto-advance courant

  pendingNavigation = true;
  const screen = SCREENS[screenId];
  if (!screen) {
    showToast(`Écran '${screenId}' introuvable`);
    pendingNavigation = false;
    return;
  }

  const frame = document.getElementById('proto-frame');
  // 1. Animer la sortie de l'écran courant (si animation != 'none')
  // 2. Injecter screen.html dans frame
  // 3. Animer l'entrée
  // 4. Update breadcrumb, sidebar highlight, screen-id footer
  // 5. Intercepter les clics sur [data-nav-target] dans le nouvel écran
  // 6. Détecter les sous-zones scrollables → scroll-hint (voir plus bas)
  // 7. Si screen.auto_advance → scheduleAutoAdvance(screen.auto_advance)
  // 8. À la fin de l'animation : pendingNavigation = false; currentScreen = screenId
}
```

**Anti-race rapide successif** : si un nouveau `renderScreen` arrive avant la fin du précédent (ex: clic rapide sidebar), le précédent setTimeout doit être neutralisé via une closure flag (`let myTransitionId = ++transitionCounter; setTimeout(() => { if (myTransitionId !== transitionCounter) return; ... })`). En usage humain c'est invisible, mais ça gêne les tests automatisés et certains edge cases.

**Navigation par data-nav-target** :
```js
document.getElementById('proto-frame').addEventListener('click', (e) => {
  const target = e.target.closest('[data-nav-target]');
  if (!target) return;
  e.preventDefault();
  const navTo = target.getAttribute('data-nav-target');
  const fullTarget = navTo.includes(':') ? navTo : `${navTo}:default`;
  if (target.hasAttribute('data-nav-back') || navTo === 'back') {
    goBack();
    return;
  }
  pushAndNavigate(fullTarget);
});
```

**Auto-advance (transitions temporisées)** :
```js
function scheduleAutoAdvance({ target, delay }) {
  const indicator = document.getElementById('proto-auto-advance');
  const fill = document.getElementById('proto-auto-fill');
  const eta = document.getElementById('proto-auto-eta');
  document.getElementById('proto-auto-target').textContent = target;
  indicator.hidden = false;

  // Reset CSS animation pour qu'elle redémarre si on revient sur le même screen
  fill.style.animation = 'none';
  void fill.offsetWidth; // force reflow
  fill.style.animation = '';
  fill.style.animationDuration = `${delay}ms`;

  const startedAt = performance.now();
  const tick = () => {
    const elapsed = performance.now() - startedAt;
    const remaining = Math.max(0, delay - elapsed);
    eta.textContent = (remaining / 1000).toFixed(1);
    if (remaining > 0 && autoAdvanceTimer) {
      autoAdvanceRaf = requestAnimationFrame(tick);
    }
  };
  autoAdvanceRaf = requestAnimationFrame(tick);

  autoAdvanceTimer = setTimeout(() => {
    autoAdvanceTimer = null;
    indicator.hidden = true;
    if (target === 'back') goBack();
    else pushAndNavigate(target);
  }, delay);
}

function cancelAutoAdvance() {
  if (autoAdvanceTimer) { clearTimeout(autoAdvanceTimer); autoAdvanceTimer = null; }
  if (autoAdvanceRaf)   { cancelAnimationFrame(autoAdvanceRaf); autoAdvanceRaf = null; }
  document.getElementById('proto-auto-advance').hidden = true;
}

document.getElementById('proto-auto-stop').addEventListener('click', cancelAutoAdvance);
```

**L'auto-advance DOIT être annulé sur** :
- toute nav manuelle (data-nav-target click, sidebar click, back, prev/next, Esc, keyboard ←/→)
- click sur le bouton "stop" de l'indicator
- nouveau renderScreen (via `cancelAutoAdvance()` en début de fonction)

**CSS de la barre de progression** :
```css
.proto-auto-advance__fill {
  height: 100%;
  background: var(--accent);
  width: 100%;
  transform-origin: left;
  animation: proto-auto-fill linear forwards;
}
@keyframes proto-auto-fill {
  from { transform: scaleX(0); }
  to   { transform: scaleX(1); }
}
```

**Détection des sous-zones scrollables (scroll-hint)** :
```js
function markScrollHints(root) {
  const candidates = root.querySelectorAll('*');
  candidates.forEach(el => {
    const cs = getComputedStyle(el);
    const overflowY = cs.overflowY;
    if ((overflowY === 'auto' || overflowY === 'scroll') &&
        el.scrollHeight > el.clientHeight + 1) {
      // Forcer position: relative si static (sinon ::after se positionne mal)
      if (cs.position === 'static') el.style.position = 'relative';
      el.setAttribute('scroll-hint', 'true');
    }
  });
}
```

```css
.proto-frame [scroll-hint="true"]::after {
  content: ""; position: absolute; bottom: 0; left: 0; right: 0; height: 24px;
  background: linear-gradient(to bottom, transparent, var(--bg));
  pointer-events: none;
}
```

À appeler en fin de `renderScreen()` après injection du HTML.

**Warning au boot — détection des débordements** :
```js
function auditOverflows() {
  const overflowing = [];
  for (const screenId of SCREENS_LIST) {
    // Render hors-écran dans un détacher pour mesurer
    const probe = document.createElement('div');
    probe.style.cssText = 'position:fixed;left:-9999px;top:0;width:390px;height:844px;';
    probe.innerHTML = SCREENS[screenId].html;
    document.body.appendChild(probe);
    const phoneScreen = probe.querySelector('.phone__screen') || probe;
    if (phoneScreen.scrollHeight > phoneScreen.clientHeight + 4) {
      overflowing.push(screenId);
    }
    document.body.removeChild(probe);
  }
  if (overflowing.length) {
    console.info(
      `[Prototype] ${overflowing.length} screen(s) scrollable(s) — vérifier que les CTAs ne sont pas hors-vue :`,
      overflowing
    );
  }
}
```

**Validation auto au boot — règles globales strippées** :
```js
function auditGlobalCss() {
  const bodyDisplay = getComputedStyle(document.body).display;
  if (bodyDisplay !== 'flex') {
    console.warn(
      `[Prototype] body.display = "${bodyDisplay}" (attendu "flex"). ` +
      `Une règle globale d'un flow-style a peut-être survécu au stripping.`
    );
  }
}
```

**Choix de l'animation** :
```js
function chooseAnimation(from, to) {
  const [fromFlow, fromPageState] = from.split('/');
  const [fromPage] = fromPageState.split(':');
  const [toFlow, toPageState] = to.split('/');
  const [toPage] = toPageState.split(':');
  if (fromFlow !== toFlow)             return 'slide-flow';   // entre flows : slide marqué
  if (fromPage !== toPage)             return 'slide-page';   // entre pages d'un flow : slide léger
  return 'fade-state';                                         // entre états d'une page : fade
}
```

**Keyboard shortcuts** :
- `← →` : prev / next dans `SCREENS_LIST` (cancelAutoAdvance avant)
- `B` : back (history) (cancelAutoAdvance avant)
- `S` : toggle sidebar
- `Esc` : back to entry screen (cancelAutoAdvance avant)
- `1` … `9` : raccourcis rapides vers les 9 premiers écrans (optionnel)

**Sidebar** :
```js
function renderSidebar() {
  // Group par flow_id, sub-list par page_id, sub-sub-list par state_id
  // Highlight l'élément correspondant à currentScreen
  // Click sur un élément → renderScreen(screenId, { animation: 'fade' })
  // Petit indicateur visuel sur les screens orphelins (pas ciblés par data-nav-target ni data-auto-advance)
  // Badge "auto" couleur var(--accent) sur les screens avec auto_advance
}
```

### Phase 3 — Extraction des cells par flow

Pour chaque `flows/<NN-flow>/<page>.html`, le skill :

1. Parse le HTML, trouve toutes les `[data-screen-label]` (= les cells)
2. Pour chaque cell, lit aussi **les attributs de cell** (avant extraction du phone) :
   - `data-auto-advance` → `screen.auto_advance.target`
   - `data-auto-advance-delay` → `screen.auto_advance.delay` (défaut 3000)
   - ⚠️ Ces attributs sont sur la cell (`[data-screen-label]`), PAS dans le HTML extrait du phone — ne pas chercher dedans
3. Extrait `.phone__screen` (le viewport mobile, sans le chrome de cell)
4. Si pas de `.phone__screen` (web/desktop), **fallback** : extrait le contenu brut moins `.cell-head` et `.cell-desc`
5. **Réécrit les paths relatifs** : `Prototype.html` est à la racine du kit, mais les flows utilisent `../../ds/...`. Le skill doit faire :
   ```js
   html = html
     .replaceAll('../../ds/', 'ds/')
     .replaceAll('../ds/', 'ds/');
   ```
6. Sauvegarde le HTML extrait dans `SCREENS[<flow_id>/<page_id>:<state_id>]`

⚠️ **Préserver les éléments OS-chrome** dans le HTML extrait : `data-os-chrome="status-bar"`, `dynamic-island`, `home-indicator` font partie du rendu visuel du Prototype même s'ils sont skipés par `ingest-mockup`. Le Prototype montre le rendu complet pour le designer/stakeholder.

### Phase 4 — Stripping robuste des règles globales dans les flow-styles

Les `<style>` inline de chaque flow contiennent des règles globales qui cassent le shell Prototype : `body { display: flex }` du shell est écrasé par `body { display: grid }` du flow, etc.

**Sélecteurs à stripper** (liste exhaustive — à compléter au fil des kits) :
- `body`, `html`, `*`
- `.canvas`, `.cell`, `.cell-head`, `.cell-desc`, `.page-head`
- `@media (...) { body { ... } }` et autres wrappers `@media` autour de ces sélecteurs

**Méthode** :
1. Parser ligne par ligne avec regex naïve (suffisant pour la majorité des cas)
2. Pour les cas complexes (nested rules, pseudo-classes complexes, `@supports`) : si une règle ressemble à une règle globale mais a un sélecteur non listé, **logger un warning skill-side** plutôt que d'injecter le CSS sans validation
3. Validation runtime : `auditGlobalCss()` au boot du Prototype (cf. plus haut) vérifie que `body.display === "flex"` — sinon warning console

⚠️ **Ne JAMAIS injecter le CSS d'un flow sans stripping**. Une règle `.canvas { display: grid }` qui passe au travers casse silencieusement la mise en page du Prototype.

### Phase 5 — Génération du fichier final

Le skill assemble :
- Le shell HTML du Prototype (sidebar + stage + meta + auto-advance indicator)
- Les flow-styles extraits + strippés (Phase 4)
- Le CSS local (scopé `.proto-*` pour ne pas polluer le DS)
- Le **scroll fallback** (CSS obligatoire — non négociable)
- Le JS runtime (avec auto-advance, scroll-hint, anti-race, audit)
- Les `SCREENS` et `NAVIGATION` inlinés en JS

Pose à la racine `Prototype.html` (overwrite si existe).

## 🎨 Conventions de navigation (à respecter dans les Flows)

Trois conventions complémentaires permettent de décrire la navigation côté maquette :

### `data-nav-target` — navigation déclenchée par interaction

Pour qu'un élément soit navigable au clic, le poser dans le HTML du Flow :

```html
<!-- Bouton de connexion → liste des tâches -->
<button class="btn btn-primary" data-nav-target="tasks/list:default">
  Se connecter
</button>

<!-- Click sur une row → écran détail -->
<div class="task-row" data-nav-target="tasks/detail:default">
  ...
</div>

<!-- Bouton retour explicite -->
<button class="nav-action" data-nav-target="back" data-nav-back="true">
  <svg ...><use href="#icon-chevron-left"/></svg>
  Tâches
</button>

<!-- Lien vers un autre flow -->
<a class="link" data-nav-target="settings/profile:default">Mon profil</a>
```

**Format** :
- `<flow_id>/<page_id>` — navigue vers l'état `:default` de la page cible
- `<flow_id>/<page_id>:<state_id>` — navigue vers un état spécifique (ex: ouverture d'un modal, page en loading, page en error)
- `back` (avec `data-nav-back="true"`) — pop l'historique, équivalent à `Navigator.pop()` Flutter

### `data-auto-advance` + `data-auto-advance-delay` — navigation temporisée

Posés sur la **cell** `[data-screen-label]` (PAS dans le `.phone__screen` extrait). Sémantique : "après X ms d'affichage, navigue automatiquement vers Y".

```html
<!-- Splash → home après 1.5s -->
<div class="cell"
     data-screen-label="01 Splash default"
     data-auto-advance="onboarding/welcome:default"
     data-auto-advance-delay="1500">
  <div class="cell-head">…</div>
  <div class="phone">
    <div class="phone__screen">…</div>
  </div>
</div>

<!-- Processing → result après 3.5s -->
<div class="cell"
     data-screen-label="03 Magic processing"
     data-auto-advance="onboarding/magic:result"
     data-auto-advance-delay="3500">
  ...
</div>

<!-- Snackbar/toast qui se dismiss tout seul après 2.5s — auto-advance vers back -->
<div class="cell"
     data-screen-label="02 Capture saved"
     data-auto-advance="back"
     data-auto-advance-delay="2500">
  ...
</div>
```

**Sémantique** :
- `data-auto-advance` : screenId cible (même format que `data-nav-target`, accepte aussi `back`)
- `data-auto-advance-delay` : milliseconds avant déclenchement, **défaut 3000**

**Cas d'usage typiques** :
- Splash screen → home
- Loading / processing → result
- Toast / snackbar / confirmation éphémère → dismiss auto
- Tutorial step transitions
- Démo de transitions automatiques (typiquement pour une présentation stakeholder)

**Pourquoi DEUX attributs séparés et pas un format compact ?**
Le format `screen_id` est `<flow>/<page>:<state>` — il contient déjà un `:`. Donc `data-auto-advance="onboarding/magic:result:3500"` serait ambigu. Garder deux attributs distincts est plus lisible et plus parsable.

### Co-bénéfices des trois conventions

- Le designer décrit la navigation côté maquette, pas le dev par devinette côté code
- Les outils de code-gen peuvent consommer ces attributs pour générer `Navigator.pushNamed('/<flow>/<page>')` directement
- L'agent codeur n'a plus à inférer "ce bouton navigue probablement vers X" — c'est explicite
- Les transitions temporisées (splash, loading) sont jouables en démo sans intervention manuelle

## ✅ Validation visuelle (obligatoire)

Avant de dire "c'est livré" :

1. `mcp__chrome-devtools__new_page url: file:///path/to/Prototype.html`
   - Si MCP refuse (`browser already running` / profile occupé), fallback : `open -a "Google Chrome" /path/to/Prototype.html` et demander à l'utilisateur de regarder. **Ne pas bloquer la phase de validation pour autant** — documenter la limitation dans le rapport final.
2. `list_console_messages types: ["error", "warn"]`
   - 0 erreur (sinon SCREENS mal sérialisé / nav-target broken)
   - Warning autorisé : "X screens scrollables …" (info, pas erreur)
   - Warning interdit : "body.display = ..." (= règle globale a survécu au stripping)
3. `take_screenshot` du screen d'entrée → vérifier rendu
4. **Test scroll** : `evaluate_script` qui sélectionne 1-2 screens supposés scrollables, vérifie `phoneScreen.scrollHeight > phoneScreen.clientHeight` ET que le scroll fonctionne réellement (`phoneScreen.scrollTop = 100; assert(scrollTop > 0)`)
5. `evaluate_script` qui clique sur le premier `[data-nav-target]` du screen courant → vérifier navigation
6. **Test auto-advance** (si au moins 1 screen avec `auto_advance`) :
   - Naviguer vers ce screen
   - Vérifier que l'indicator `#proto-auto-advance` s'affiche
   - Attendre `delay + 200ms`
   - Vérifier que `currentScreen` est passé à la cible
7. **Test cancel auto-advance** :
   - Re-naviguer vers le screen avec auto_advance
   - Cliquer le bouton "stop" avant la fin du delay
   - Vérifier que `currentScreen` n'a pas changé
   - Vérifier que `autoAdvanceTimer === null`
8. Tester `keyboard ←` et `keyboard →` (cancelAutoAdvance attendu)
9. Tester back via bouton

**Tous les tests doivent passer avant de déclarer le skill terminé.**

Si un click `data-nav-target` mène à un toast "écran non trouvé" → liens cassés à corriger dans les Flows source (pas dans le Prototype).

## 🚫 Anti-patterns

1. **Modifier le `Prototype.html` à la main** — c'est un fichier généré, à régénérer après chaque modif Flow
2. **Mettre `data-nav-target` sur des éléments non interactifs** (texte statique, image décorative) — pose seulement sur les CTA / rows / liens vrais
3. **Cibler un écran qui n'existe pas** — le skill warning, et le clic devient no-op au runtime
4. **Cibler un état qui n'existe pas** dans une page existante (ex: `tasks/list:foo` quand il n'y a pas de cellule "foo") — idem
5. **Dupliquer les flows dans des iframes** — le Prototype inline tout en JS, pas d'iframe
6. **Servir via http-server** quand on peut servir en `file://` — le Prototype est conçu autonome
7. **Injecter les flow-styles sans stripping** — une règle `body { display: grid }` casse le layout du Prototype
8. **Omettre le scroll fallback CSS** — sans lui, les écrans avec `.phone__screen { overflow: hidden }` mais sans wrapper `.phone__content` débordent silencieusement
9. **Mettre `data-auto-advance` dans le `.phone__screen` au lieu de la cell** — l'attribut doit être sur `[data-screen-label]`, sinon il sera extrait dans le HTML et perdu côté skill
10. **Cibler un auto-advance qui crée une boucle infinie** sans état terminal (A → B → A → B …) — toujours prévoir une sortie manuelle (CTA `data-nav-target` quelque part)
11. **Skip la phase de stripping CSS** quand un flow a beaucoup de styles inline custom — toujours valider avec `auditGlobalCss()` au boot

## 🔄 Re-génération idempotente

`Prototype.html` est régénéré at-rest à chaque appel du skill. Si rien n'a changé dans `flows/`, le contenu est identique. Si une page a été ajoutée / un `data-nav-target` modifié / un `data-auto-advance` ajouté, le fichier reflète l'état actuel.

Pour suivre les évolutions : commit `Prototype.html` dans git, le diff montre exactement quels écrans ont changé.

## 📤 Livrable

- **Un seul fichier** : `Prototype.html` à la racine du kit, autonome, ouvrable directement en `file://`
- **Aucune nouvelle dépendance** au kit (pas de package.json, pas de build, pas de framework JS)
- **Mention dans le README/GUIDELINES** : "Prototype interactif disponible — ouvrir `Prototype.html` dans un browser. Régénérer après modif via `ui-kit-prototype`."

## 🎯 Cas limites & dégradés

### Pas de `data-nav-target` ni `data-auto-advance` dans le kit

Le Prototype fonctionne quand même : la sidebar permet de naviguer manuellement entre écrans, mais aucune interaction in-screen ne déclenche de navigation. Warning amical au boot : "0 lien data-nav-target ni data-auto-advance détecté — navigation manuelle uniquement via la sidebar."

### Mockup web/desktop (pas de `.phone__screen`)

Si la cell n'a pas de `.phone__screen`, extraire le contenu direct de la cell (en excluant `.cell-head` et `.cell-desc`) et l'afficher pleine largeur. Le frame `.proto-frame` devient un container flexible.

### Très grand nombre d'écrans (50+)

La sidebar passe en mode collapsed-by-default avec un search input en haut. Group expand/collapse au click sur le header de flow.

### Click sur un élément avec `data-nav-target` ET `href`

Priorité au `data-nav-target` (preventDefault sur le click). Le `href` reste utile en mode "open this Flow file directly" hors prototype.

### Auto-advance vers un screen inexistant

Le skill warning à la génération (`unknowns`). Au runtime, `scheduleAutoAdvance` log une erreur console et n'arme pas le timer.

### Chrome MCP refuse `new_page` (profile occupé)

Fallback : ouvrir via `open -a "Google Chrome" /path/to/Prototype.html`, demander à l'utilisateur de valider visuellement et reporter d'éventuels warnings console manuellement. Documenter dans le rapport que la validation auto a été partielle.

## 🐛 Points bloquants connus (à anticiper en construction)

- **Paths relatifs `../../ds/`** : les flows sont dans `flows/<NN-flow>/<page>.html` et utilisent `../../ds/` pour les assets. Le `Prototype.html` est à la racine — donc remplacer `../../ds/` → `ds/` (et `../ds/` → `ds/` au cas où) lors de l'extraction.
- **Cell-level attrs invisibles dans le HTML extrait** : `data-auto-advance` et `data-auto-advance-delay` sont sur la cell, hors du `.phone__screen`. Lire ces attrs AVANT d'extraire le phone, pas après.
- **Animation CSS qui ne redémarre pas** : si on revient sur le même screen avec auto_advance, l'animation de la barre de progression ne redémarre pas par défaut. Solution : `el.style.animation = "none"; void el.offsetWidth; el.style.animation = "";` pour forcer le reflow.
- **`grep -c` qui retourne 0 et casse `&&`** : ajouter `|| true` aux pipelines bash quand un grep peut ne rien matcher.
- **Régression silencieuse du scroll** : un écran qui scroll OK aujourd'hui peut casser demain si quelqu'un retire le wrapper `.phone__content`. Le scroll fallback CSS du shell rend ça non-bloquant, mais l'audit `auditOverflows()` au boot le signale au designer.

## 📖 Workflow utilisateur typique

```
1. ToF crée un kit via ui-kit-creator (5 flows, 20 pages, 60 états)
2. ToF édite quelques boutons dans flows/01-onboarding/01-welcome.html pour ajouter
   data-nav-target="onboarding/capture:default" sur le CTA principal
3. ToF ajoute data-auto-advance="onboarding/magic:result" + data-auto-advance-delay="3500"
   sur la cell "03 Magic processing"
4. ToF invoque ui-kit-prototype
5. Le skill génère Prototype.html, ouvre dans le browser
6. ToF présente à un stakeholder : on clique le CTA → on arrive sur l'écran capture,
   puis on déclenche le scan → écran processing s'affiche → 3.5s plus tard, transition
   automatique vers result (avec barre de progress visible + bouton stop si besoin)
7. Stakeholder dit "et après le swipe-to-delete, on va où ?"
8. ToF ajoute data-nav-target sur le swipe action, re-invoque ui-kit-prototype
9. Prototype.html régénéré, on re-démontre
```

Le Prototype devient l'objet partageable / présentable en réunion, plus seulement la grille canvas.
