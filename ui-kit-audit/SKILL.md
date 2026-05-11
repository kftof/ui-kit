---
name: ui-kit-audit
description: Audite un UI kit HTML existant contre les 3 playbooks de stratégie (onboarding/paywall, engagement/viralité, psychologie/conversion) ET contre les règles d'accessibilité mobile WCAG 2.1 AA, puis produit un rapport `Audit.md` à la racine du kit avec les critères respectés / non / N/A + justification + recommandations prioritaires ordonnées. À invoquer pour valider qu'un onboarding respecte les principes psychologiques, qu'un paywall maximise le MRR (avec patterns 2026 : trial annual-only, soft-then-hard, free vs pro table embedded, post-close offer, plan loader, PPP-adjusted pricing, AI personalization), qu'une boucle d'habitude Hook Model est implémentée, ou que le design est accessible. Ne modifie pas le kit — pour corriger, enchaîner avec `ui-kit-editor`.
---

# UI Kit Audit — vérification stratégique d'un kit

Évalue un kit existant contre **3 playbooks stratégiques** (`playbooks/01-onboarding-paywall.md`, `playbooks/02-engagement-virality.md`, `playbooks/03-conversion-psychology.md`) **+ une famille de critères accessibilité mobile WCAG 2.1 AA** (référencée depuis `ui-kit-creator/SKILL.md` § Accessibilité mobile). Produit un rapport `Audit.md` à la racine du kit.

**Total de critères** : **76 critères stratégiques** (25 playbook 1 + 23 playbook 2 + 28 playbook 3) + **7 critères a11y WCAG mobile** = **83 critères** maximum si on lance l'audit complet.

**Skill diagnostic, pas correctif** : il identifie les problèmes, ne les fixe pas. Pour appliquer les corrections, enchaîner avec `ui-kit-editor`.

## 🎯 Quand utiliser ce skill

- Validation pré-release : "Mon onboarding maximise-t-il la conversion ?"
- Audit d'un kit hérité : "Ce kit respecte-t-il les bonnes pratiques ?"
- Pré-paywall : "Mon paywall a-t-il les leviers de conversion attendus ?"
- Audit ASO : "Mes captures d'écran App Store vendent-elles des bénéfices ou des features ?"
- Audit de différenciation : "Mon design sort-il du AI slope générique de la niche ?"

## 📋 Inputs requis

**Obligatoire** :
1. **Path du kit** — racine d'un kit produit par `ui-kit-creator` (avec `ds/`, `flows/`, `UI Kit.html`)

**Optionnel** :
2. **Liste de playbooks à appliquer** — par défaut, les 3. Possibilités : `["01"]` (onboarding/paywall seul), `["02"]` (engagement seul), `["03"]` (psycho seul), ou combinaisons.
3. **Niche déclarée** — pour le mapping §4.1 du playbook 3 (méditation / fitness / fintech / etc.). Si absente, le skill tente de l'inférer depuis `README.md` / `PRD.md` du kit.

## 🧭 Workflow

### Phase 1 — Reconnaissance du kit

1. **Lire la structure du kit** :
   - `ls flows/` → liste des flows présents
   - `ls flows/*onboarding*/` → présence d'un flow onboarding (sinon les critères OB-* du playbook 1 sont marqués `n/a` au lieu de `fail`)
   - `grep -r paywall flows/` → présence d'un écran paywall
   - `ls flows/*settings*/` → présence d'un flow settings
   - `ls flows/*history*/ flows/*collection*/` → présence d'un flow collection

2. **Inférer la niche** (pour le playbook 3) :
   - Lire `README.md` + `PRD.md` (si présent) — chercher des mots-clés (méditation, fitness, productivité, finance, dating, etc.)
   - Si ambigu → demander à l'utilisateur en une question avant de lancer l'audit

3. **Charger les playbooks demandés** depuis `<chemin du repo ui-kit>/playbooks/0X-*.md`. Si le repo n'est pas accessible, demander à l'utilisateur de le pointer.

### Phase 2 — Évaluation critère par critère

Pour chaque critère de chaque playbook demandé (table "Critères d'audit machine-readable" à la fin du playbook) :

1. Identifier la zone du kit à inspecter (colonne "Où chercher")
2. Exécuter le grep / la lecture du fichier concerné
3. Évaluer : `pass` / `fail` / `n/a` (si la zone n'existe pas — ex: pas de paywall dans un kit free-only)
4. Pour les `fail` : produire une **justification courte** (1-2 lignes) qui dit POURQUOI le critère échoue
5. Pour les `pass` : pas besoin de justification (le critère est rempli)

**Règle d'or** : ne pas inventer de critères ou en oublier. Le set est figé dans les playbooks pour reproductibilité.

### Phase 3 — Validation visuelle (cas particuliers)

Certains critères ne sont pas grep-ables — ils nécessitent un screenshot et une inspection visuelle :

- `CV-3` (icône app contrastée light + dark) → screenshot de la galerie Brand de `UI Kit.html`
- `CV-4` (premier bénéfice lisible à 200px) → screenshot zoomé du visuel App Store + check de la lisibilité de la typo
- `SH-2` (branding visible sur sharable moment) → screenshot de la cell sharable + check présence logo
- `DIFF-3` (palette "AI slope" ?) → screenshot de la galerie Couleurs de `UI Kit.html` + comparaison avec niche cible

Pour ces critères : utiliser `mcp__chrome-devtools__new_page` + `take_screenshot` + lecture du PNG, puis évaluer.

### Phase 4 — Production du rapport `Audit.md`

À la racine du kit, écrire `Audit.md` (overwrite si existe) avec :

```markdown
# Audit du kit — [Nom du produit]

Date : YYYY-MM-DD
Playbooks appliqués : 01 (onboarding/paywall), 02 (engagement/viralité), 03 (psychologie/conversion)
Niche inférée : [méditation / fitness / etc.]

## Score global

| Playbook / Famille | Pass | Fail | N/A | Score |
|---|---|---|---|---|
| 01 — Onboarding/Paywall (25 critères : OB-* + PW-*) | X | Y | Z | X / (X+Y) = NN% |
| 02 — Engagement/Viralité (23 critères : CL-* SH-* RV-* PR-* RM-* HK-*) | … | … | … | … |
| 03 — Psychologie/Conversion (28 critères : OB-V* CV-* DIFF-* PA-* PE-* STYLE-* AP-*) | … | … | … | … |
| WCAG 2.1 AA — Accessibilité mobile (7 critères : WC-*) | … | … | … | … |
| **TOTAL** | … | … | … | … |

## Détail par critère

### Playbook 1 — Onboarding/Paywall

| ID | Critère | Statut | Justification |
|---|---|---|---|
| OB-1 | Problème + solution dans les 3 premiers écrans | ✅ pass | — |
| OB-2 | Moment Aha en début d'onboarding | ❌ fail | Le premier écran montre un logo de bienvenue sans contenu valeur. Aucune stat ou prise de conscience visible avant l'écran 4. |
| OB-3 | ≥ 3 questions personnelles | ⏭️ n/a | Pas de flow onboarding détecté dans ce kit |
| … | … | … | … |

### Playbook 2 — Engagement/Viralité
…

### Playbook 3 — Psychologie/Conversion
…

## Recommandations prioritaires

Liste **ordonnée** des fails les plus critiques à corriger, avec proposition d'action concrète :

1. **OB-2 — Ajouter un moment Aha en écran 1-3** : remplacer l'écran "Bienvenue dans MyApp" par une statistique choquante personnelle au problème adressé. Ex: "92 % des gens abandonnent leurs résolutions en 2 semaines". À implémenter via `ui-kit-editor` recette "Refondre les premiers écrans d'onboarding".
2. **PW-3 — Activer un trial gratuit 3 jours** : actuellement le paywall propose un achat direct sans trial. Ajouter une option trial réduit la friction et active 70 % des trials post-onboarding. À implémenter via `ui-kit-editor`.
…

## Critères non auditables automatiquement

Liste les critères qui n'ont pas pu être évalués par grep / inspection visuelle et qui nécessitent un jugement humain :

- `DIFF-3` (palette "AI slope" ?) : à valider manuellement par comparaison avec les 5 apps leaders de la niche
- `CV-2` (Custom Product Pages prévues) : nécessite de regarder le déploiement App Store, pas le kit lui-même
- `RM-3` (empty states actionables) : à vérifier qu'aucun empty state n'a été oublié — le skill ne sait que ce qui est dans le kit, pas ce qui manque
```

### Phase 4.5 — Critères additionnels Accessibilité (WCAG 2.1 AA)

Famille de critères a11y mobile, basée sur la section "♿ Accessibilité mobile — WCAG 2.1 AA" du `ui-kit-creator`. À auditer **par défaut** sur tout kit mobile (sauf si l'utilisateur l'exclut explicitement). Inclure dans le rapport `Audit.md` comme une 4e section après les 3 playbooks.

| ID | Critère | Où chercher |
|---|---|---|
| WC-1 | Variable `--touch-target-min` (44px iOS / 48px Android) définie dans `tokens.css` | `grep -E '^\s*--touch-target-min' ds/tokens.css` |
| WC-2 | Tous les éléments interactifs (boutons, rows, icon-buttons, tabs, links) ont `min-height` ≥ touch-target-min | inspection visuelle des cells + grep `min-height` dans `components.css` |
| WC-3 | Paires de couleurs critiques respectent les ratios WCAG (texte normal 4.5:1, texte large 3:1, UI components 3:1) | calcul mécanique sur `--text` × `--bg`, `--text-muted` × `--bg`, `--text-on-primary` × `--primary`, mode dark inclus |
| WC-4 | Icon-buttons sans texte visible ont un `data-a11y-label="..."` | `grep -rE '<button[^>]*class="[^"]*\b(icon-btn\|nav-action)\b' flows/ \| grep -v 'data-a11y-label'` → 0 match |
| WC-5 | Images décoratives ont `data-a11y-hidden="true"` (et `alt=""`) | `grep -rE '<img[^>]+>' flows/ \| grep -v 'alt=' \| grep -v 'data-a11y-hidden'` → 0 match |
| WC-6 | Toasts / snackbars ont `data-a11y-live="polite"` (ou `assertive` si critique) | `grep -rE 'class="[^"]*\b(toast\|snackbar\|alert)\b' flows/ \| grep -v 'data-a11y-live'` → 0 match |
| WC-7 | Pas d'information transmise par la couleur seule (un dot rouge/vert pour status doit avoir aussi un icon ou un label) | inspection visuelle des status indicators |

**Note** : WC-3 nécessite un calcul de contraste — si chrome-devtools MCP est disponible, utiliser `evaluate_script` pour extraire les `computedStyle` et calculer. Sinon proposer à l'utilisateur de lancer manuellement le check via WebAIM Contrast Checker, et marquer `WC-3` comme "non auditable automatiquement".

### Phase 5 — Cas particuliers

**Si aucun flow onboarding** : tous les critères `OB-*` et `OB-V*` sont `n/a`, le rapport le mentionne explicitement et propose d'invoquer `ui-kit-editor` pour créer un onboarding (ou de poser la question : "Est-ce une app free-only sans onboarding ?").

**Si aucun paywall** : critères `PW-*` en `n/a`, rapport mentionne que l'app n'a pas de monétisation visible — proposer d'invoquer `ui-kit-editor` pour ajouter un paywall si le produit est censé monétiser.

**Si la niche n'est pas identifiable** : laisser `PA-1` en `n/a` avec note "Niche non déclarée, mapping palette/typo non vérifiable".

## 🚫 Anti-patterns

1. **Modifier le kit pendant l'audit** — ce skill est purement diagnostic. Pour corriger, l'utilisateur enchaîne avec `ui-kit-editor` après lecture du rapport.
2. **Inventer des critères absents des playbooks** — le set est figé pour reproductibilité. Si tu penses qu'un critère manque, le proposer en ajoutant au playbook concerné (via PR), pas le glisser dans l'audit.
3. **Skipper la phase de validation visuelle** — certains critères ne sont pas grep-ables (contraste icône, lisibilité capture). Sans screenshot + lecture, ces critères doivent être marqués "non auditable automatiquement", pas évalués au pif.
4. **Surreporter** — ne pas écrire un rapport de 200 lignes si 5 critères seulement échouent. Le rapport est consommé par un humain ; brièveté > exhaustivité performative.
5. **Faire passer des `fail` en `n/a`** parce que le critère "ne s'applique pas vraiment" — si le critère existe et la zone du kit existe, c'est `pass` ou `fail`. `n/a` est strictement réservé aux cas où la zone n'existe pas (pas de paywall, pas d'onboarding).

## ✅ Validation finale

Avant de livrer le rapport :
- Vérifier que chaque critère du / des playbook(s) demandé(s) a un statut
- Le score global est cohérent (somme pass + fail + n/a = total des critères)
- Les recommandations prioritaires sont **ordonnées** par impact (les pires fails d'abord)
- Le rapport ne dépasse pas ~3 viewports à l'écran — sinon condenser
- `Audit.md` est écrit à la racine du kit, ouvrable directement

## 📤 Livrable

`Audit.md` à la racine du kit, qui contient :
- Score global pour chaque playbook appliqué
- Détail critère par critère (table)
- Recommandations prioritaires ordonnées
- Critères non auditables automatiquement (transparency)

L'utilisateur peut ensuite :
- Invoquer `ui-kit-editor` pour appliquer les fixes recommandés
- Re-invoquer `ui-kit-audit` après modifs pour vérifier la progression
- Commit le `Audit.md` dans git pour tracer l'évolution du score entre versions

## 🔄 Re-invocation idempotente

Re-invoquer le skill sur le même kit avec les mêmes playbooks → produit un `Audit.md` identique. Si le kit a évolué (ajout d'écrans, modif de l'onboarding), le nouveau rapport reflète l'état actuel. Diff git du `Audit.md` montre exactement quels critères sont passés `fail` → `pass` (ou inversement).

## 🎯 Workflow utilisateur typique

```
1. ToF a créé un kit via ui-kit-creator (onboarding + paywall + core flows)
2. ToF invoque ui-kit-audit avec les 3 playbooks
3. Le skill lit le kit + les 3 playbooks
4. Évalue les 49 critères (18 + 17 + 14)
5. Écrit Audit.md à la racine du kit
6. ToF lit le rapport : score 31/49, 7 fails identifiés
7. ToF invoque ui-kit-editor pour fixer les 3 fails les plus critiques
8. ToF re-invoque ui-kit-audit → score 38/49
9. Itération jusqu'à un score acceptable (typiquement 80%+ sur les playbooks pertinents pour le produit)
```

Le `Audit.md` devient un baromètre de qualité stratégique du kit, mesurable entre versions.
