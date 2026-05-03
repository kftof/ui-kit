---
name: ui-kit-to-code
description: Transforme un écran HTML hi-fi (de n'importe quel UI kit) en code fonctionnel dans n'importe quel framework (React, Vue, Flutter, SwiftUI, Jetpack Compose, HTML/CSS vanilla…). À invoquer pour un handoff design→dev, implémenter un écran mocké, ou migrer une maquette vers un framework spécifique.
---

# UI Kit to Code — Handoff universel

Transforme une maquette HTML hi-fi en code production-ready dans le framework cible de l'utilisateur, en respectant les conventions du codebase existant.

## 📋 Contexte à charger

**Obligatoire** :
1. Le fichier HTML de la maquette cible — situé dans `flows/NN-<flow_id>/<page_id>.html`. Convention de dérivation des IDs :
   - Dossier `flows/01-tasks/` → `flow_id = "tasks"` (strip `/^\d+-/`)
   - Fichier `list.html` (ou `NN-list.html`) → `page_id = "list"`
   - Cellule `data-screen-label="01 List default"` → `state_id = "default"` (dernier mot, lower)
   - Identifiant global stable : `<flow_id>/<page_id>[:<state_id>]` → ex. `tasks/list:loading`
   - Cette convention permet le mapping route mobile : 1 page HTML = 1 écran/route, états = variants visuels du même écran
2. Les **tokens** du DS (`ds/tokens.css`, `variables.css`…)
3. Le **catalogue de composants partagés** (`ds/components.css` + section "Composants" de `UI Kit.html`) — c'est la source de vérité de l'API des widgets. Le lire **avant** la maquette : ça donne le contrat à mapper.
4. La **librairie d'icônes** — convention `data-asset` :
   - Source : `ds/assets/icons/<name>.svg` (un fichier par icône, contenu Lucide canonique)
   - Référence consolidée : `ds/assets/icons/icons.svg` (sprite source, vue d'ensemble)
   - **Dans les HTML, chaque icône est inlinée en plein SVG** avec un attribut `data-asset="ds/assets/icons/<name>.svg"`. **Cet attribut est la clé de matching** entre une icône visible dans la maquette et son fichier source — c'est ce qui permet à un outil de code-gen (scripts custom, agents IA) de savoir quel asset importer dans le projet mobile.
5. Le framework cible + sa stack (React+TS, Vue, Flutter, SwiftUI, Compose…)

**Fortement recommandé** :
6. Le code existant du projet (au moins un exemple de composant/écran)
7. Le système de thème/tokens du framework (ex: `theme.ts`, `Colors.kt`, `app_theme.dart`)
8. Les composants partagés/réutilisables du codebase
9. **Screenshot du Flow cible** via chrome-devtools MCP — **viewport-by-viewport** (PAS `fullPage:true`, le PNG serait trop grand pour être lu). Méthode : récupérer la liste des cellules via `evaluate_script` (sélecteur `.cell`), scroller à chaque cellule avec `behavior: 'instant'` puis `take_screenshot` au viewport. Référence visuelle pour vérifier la fidélité du code généré.

**Sans accès au codebase** : fonctionner en mode "code générique idiomatique" + signaler clairement les endroits à adapter.

**Si le kit a une organisation obsolète** (Components.html / Icons.html séparés, sprite externe `<use href="ds/assets/icons/icons.svg#…"/>`, SVG inline sans `data-asset`, assets directement sous `ds/icons/` au lieu de `ds/assets/icons/`, flows à la racine au lieu de `flows/<flow>/<page>.html`, paths SVG approximés) : signaler à l'utilisateur et proposer une consolidation préalable via le skill `ui-kit-editor` (recette "Consolider un kit non-DRY"). Le mapping sera moins fiable sinon — sans `data-asset`, il faut tenter un matching approximatif sur les paths SVG, peu fiable.

## 🧭 Règles d'or

1. **Ne jamais hardcoder** une valeur de design → toujours token/variable
2. **Réutiliser les composants existants** du projet avant d'en créer
3. **Respecter l'architecture** du projet (MVC, Clean Arch, feature-first, etc.)
4. **Préserver les hooks existants** : store, routing, i18n, auth
5. **Tout texte → système d'i18n** si le projet en a un
6. **Accessibilité minimale** : labels, roles, contrastes, tailles
7. **Pixel-fidélité au hi-fi** ; tout écart justifié

## 🔄 Workflow 6 étapes

### 1. Reconnaissance du HTML source

Faire dans cet ordre — la section Composants de `UI Kit.html` et le dossier `ds/assets/icons/` donnent l'API ; le Flow ne sert qu'à la composition.

1. **Ouvrir `UI Kit.html`** : aller à la section Composants. Extraire le contrat de chaque widget partagé (classes, modifiers, variantes, états). Faire la liste des familles utilisées par le kit (`.btn`, `.btn-primary`, `.card`, `.task-row`, `.dialog`, `.sheet`, etc.). Aussi consulter la section Iconographie pour la liste des icônes documentées.
2. **Lister `ds/assets/icons/*.svg`** : ce sont les fichiers sources des icônes (un par icône, Lucide canonique). Ces fichiers sont **directement copiables** vers le projet mobile : Flutter → `assets/icons/`, iOS → asset catalog, Android → `res/drawable/`. Noter les noms canoniques (Lucide : `ellipsis-vertical`, `trash-2`, `triangle-alert`…).
3. **Ouvrir le Flow cible** (`flows/NN-<flow_id>/<page_id>.html`) : identifier la structure d'écran (header/body/footer, sheets, modals), les widgets consommés (par classes CSS), les **icônes consommées via `data-asset`**, et les **états montrés via `data-screen-label`** (default, loading, empty, error). Extraire `flow_id` / `page_id` / `state_id` selon les règles ci-dessus pour tracer le mapping vers le code mobile.
4. **Extraire les `data-asset`** : `grep -oE 'data-asset="[^"]+"'` sur le Flow → liste exacte des assets icônes à importer dans le projet mobile.
5. **Extraire les autres assets référencés** : `grep -oE 'src="[^"]*ds/assets/[^"]+"'` → liste des `<img>` brand/illustrations/images à copier.
6. **Extraire les conventions sémantiques** (cf. section dédiée plus bas) :
   - `grep -oE 'data-os-chrome="[^"]+"'` → éléments à NE PAS coder (SafeArea s'en charge)
   - `grep -oE 'data-nav-target="[^"]+"'` → routes à brancher au router
   - `grep -oE 'data-uses="(native|ui):[^"]+"'` → packages natifs ou primitives framework requis
   - `grep -oE 'data-auto-advance="[^"]+"'` + `data-auto-advance-delay` → transitions temporisées (Timer + cancel-on-dispose)
   - `grep -oE 'data-hint="[^"]+"'` → sémantique métier non-visible (à traiter au cas-par-cas)
7. **Screenshot du Flow** via chrome-devtools (viewport-by-viewport, une cellule à la fois) — référence visuelle de fidélité.
8. **Repérer les conventions** du DS (nommage, échelle, dark mode).

Si le Flow contient du `<style>` local définissant des widgets, ou des `<svg>` inline **sans `data-asset`**, ou des `<use href="ds/...">` (sprite externe) : noter chacun comme "non couvert" et le traiter comme à créer côté code (ou suggérer une consolidation préalable du kit).

### 2. Reconnaissance du codebase cible

- Lister les composants partagés dispos
- Identifier les conventions de nommage (PascalCase, snake_case, kebab…)
- Trouver le fichier de tokens/theme du projet
- Noter le pattern de state management (Redux, Zustand, BLoC, StateFlow…)
- Noter le pattern de styling (CSS modules, Styled, Tailwind, StyleSheet, Compose modifiers…)

### 3. Cartographie HTML → framework

Produire un tableau **avant** d'écrire du code. La colonne "DS source" pointe vers la section Composants de `UI Kit.html` ou le fichier d'asset — c'est ce qui garantit la fidélité.

| Élément HTML (Flow) | DS source | Composant cible | Code source |
|---|---|---|---|
| `<header class="appbar">` | UI Kit.html § Composants → App bar | `<AppBar>` | exists in `@/components` |
| `<button class="btn btn-primary">` | UI Kit.html § Composants → Boutons (variant primary) | `<Button variant="primary">` | exists |
| `<div class="task-row">` | UI Kit.html § Composants → Task rows | `<TaskRow>` | à créer (mapping 1:1) |
| `<div class="dialog">` ou `<div class="sheet">` | UI Kit.html § Composants → Bottom sheet / Dialog | modal API native du framework | — |
| `<svg ... data-asset="ds/assets/icons/plus.svg">…</svg>` | `ds/assets/icons/plus.svg` (extraite via `data-asset`) | `<Plus />` (lucide-react) / `LucideIcons.plus` (Flutter) / `"plus"` (SF Symbols) / asset import | voir convention |
| `<img src="ds/assets/illustrations/empty-tasks.svg">` | fichier `ds/assets/illustrations/empty-tasks.svg` | `Image.asset('assets/illustrations/empty-tasks.svg')` (Flutter) / `Image("empty-tasks")` (iOS bundle) / `<img src="/illustrations/empty-tasks.svg">` (web) | copier l'asset dans le projet |

**Règle d'or** : tout widget consommé dans le Flow **doit** avoir une entrée dans la section Composants de `UI Kit.html` ; tout `<svg data-asset="…">` doit pointer vers un fichier existant dans `ds/assets/icons/` ; tout `<img src="ds/assets/...">` doit pointer vers un fichier existant dans le sous-dossier correspondant. Si ce n'est pas le cas, marquer l'élément `[hors catalogue — à créer]` et alerter.

### 4. Cartographie tokens CSS → code

| Token CSS | Équivalent cible |
|---|---|
| `var(--primary)` | `theme.colors.primary` / `AppColors.primary` / `Color(0xFFxxx)` |
| `var(--space-16)` | `theme.spacing.md` / `16.dp` / `AppSpacing.s16` |
| `var(--radius-card)` | `theme.radii.card` / `12.dp` / `BorderRadius.circular(12)` |
| `var(--fs-body)` | `theme.fonts.body` / `AppTextStyle.body` |

**Si un token manque côté code** : signaler explicitement, ne pas hardcoder silencieusement.

### 5. Implémentation

Suivre les conventions du projet. Templates par framework ci-dessous :

#### React (TS + styled-components ou CSS modules)

```tsx
import { Button } from '@/components/ui/button';
import { theme } from '@/theme';
import { useTranslation } from 'react-i18next';

export function GroupDetailPage({ groupId }: { groupId: string }) {
  const { t } = useTranslation();
  const { data, isLoading, error } = useGroup(groupId);

  if (isLoading) return <LoadingView />;
  if (error) return <ErrorView error={error} />;
  if (!data) return null;

  return (
    <Screen>
      <AppBar title={data.name} onBack={...} />
      <Content>
        <Header group={data} />
        <MemberList members={data.members} />
      </Content>
    </Screen>
  );
}
```

#### Vue 3 (Composition API + TS)

```vue
<script setup lang="ts">
import { useGroup } from '@/composables/useGroup';
const props = defineProps<{ groupId: string }>();
const { data, isLoading, error } = useGroup(props.groupId);
</script>

<template>
  <Screen>
    <AppBar :title="data?.name" @back="..." />
    <Content v-if="data">
      <GroupHeader :group="data" />
      <MemberList :members="data.members" />
    </Content>
    <LoadingView v-else-if="isLoading" />
    <ErrorView v-else-if="error" :error="error" />
  </Screen>
</template>
```

#### Flutter (Dart)

```dart
class GroupDetailPage extends StatelessWidget {
  const GroupDetailPage({super.key, required this.groupId});
  final String groupId;

  @override
  Widget build(BuildContext context) {
    final l10n = AppLocalizations.of(context)!;
    return Scaffold(
      backgroundColor: AppColors.bgLight,
      appBar: AppBar(title: Text(l10n.group), ...),
      body: BlocBuilder<GroupCubit, GroupState>(
        builder: (context, state) => switch (state) {
          GroupLoading() => const LoadingView(),
          GroupError() => ErrorView(error: state.error),
          GroupLoaded() => _Content(group: state.group),
          _ => const SizedBox.shrink(),
        },
      ),
    );
  }
}
```

#### SwiftUI

```swift
struct GroupDetailView: View {
  @StateObject var viewModel: GroupDetailViewModel

  var body: some View {
    ScreenContainer {
      switch viewModel.state {
      case .loading: LoadingView()
      case .error(let e): ErrorView(error: e)
      case .loaded(let group):
        VStack(spacing: Spacing.md) {
          GroupHeader(group: group)
          MemberList(members: group.members)
        }
      }
    }
    .navigationTitle(L10n.group)
  }
}
```

#### Jetpack Compose (Kotlin)

```kotlin
@Composable
fun GroupDetailScreen(
  groupId: String,
  viewModel: GroupDetailViewModel = hiltViewModel()
) {
  val state by viewModel.state.collectAsState()
  Scaffold(
    topBar = { AppBar(title = stringResource(R.string.group)) }
  ) { padding ->
    when (val s = state) {
      is GroupState.Loading -> LoadingView(Modifier.padding(padding))
      is GroupState.Error -> ErrorView(s.error)
      is GroupState.Loaded -> Content(s.group, Modifier.padding(padding))
    }
  }
}
```

#### HTML/CSS vanilla (si le framework cible est "web statique")

Prendre le HTML du kit, le nettoyer des métadonnées (`data-screen-label`, `.cell`, phone frame…), extraire le CSS spécifique, intégrer aux styles globaux du projet.

### 6. États manquants

Le hi-fi ne montre souvent que le happy path. Toujours ajouter :

| État | À coder |
|---|---|
| **Loading** | Skeleton / spinner cohérent avec le DS |
| **Empty** | Composant vide + CTA |
| **Error** | Message + bouton retry |
| **Offline** | Banner si le projet le gère |
| **Permission denied** | Dialog + lien settings |

Si un état manque dans le hi-fi, le signaler et proposer un design avant de coder.

## 🖼️ Assets non-icônes — mapping vers le projet cible

Le DS produit aussi des dossiers `ds/assets/brand/`, `ds/assets/illustrations/`, `ds/assets/images/`. Ces assets sont **directement copiables** vers le projet, contrairement aux icônes qui passent par une librairie. Mapping :

| Famille DS | Flutter | iOS (SwiftUI / UIKit) | Android (Compose / View) | React / web |
|---|---|---|---|---|
| `ds/assets/brand/logo.svg` | `assets/brand/logo.svg` + `flutter_svg` | Asset Catalog (vector PDF) ou bundle SVG via `SVGKit` | `res/drawable/logo.xml` (VectorDrawable) ou bundle SVG via `androidsvg` | `public/brand/logo.svg` + `<img>` |
| `ds/assets/brand/app-icon.svg` | export multi-tailles via `flutter_launcher_icons` | export tailles via Asset Catalog AppIcon | export via Image Asset Studio (`mipmap-*`) | `<link rel="icon">` |
| `ds/assets/brand/splash/splash.svg` | `flutter_native_splash` config | Launch Storyboard / Asset Catalog | Splash screen API (Android 12+) ou drawable | n/a |
| `ds/assets/illustrations/empty-tasks.svg` | `assets/illustrations/empty-tasks.svg` + `SvgPicture.asset(...)` | bundle SVG via `SVGKit` ou export PDF dans Asset Catalog | `res/drawable/empty_tasks.xml` (si VectorDrawable compatible) ou bundle SVG | `<img src="/illustrations/empty-tasks.svg">` |
| `ds/assets/images/hero.jpg` | `assets/images/hero.jpg` + `Image.asset(...)` | Asset Catalog avec @1x/@2x/@3x | `res/drawable-{hdpi,xhdpi,xxhdpi}/hero.jpg` | `<img src="/images/hero.jpg">` |

**Règles** :
- **Préserver l'arborescence** : recopier la structure `ds/assets/brand/`, `ds/assets/illustrations/`, `ds/assets/images/` telle quelle dans le projet (sous `assets/`, `Resources/`, ou équivalent). Évite de tout mélanger dans un seul dossier flat.
- **Préférer SVG aux exports raster** quand la stack le supporte (économie de poids, scaling crisp). Pour iOS récent + Android moderne + Flutter : SVG fonctionne partout via lib dédiée.
- **App icon + splash** : ces deux-là demandent des configurations spécifiques par framework — utiliser les outils dédiés (`flutter_launcher_icons`, Asset Catalog AppIcon, `flutter_native_splash`) plutôt que copier les SVG bruts.
- **Snake_case côté code mobile** : `empty-tasks.svg` → `empty_tasks.svg` quand on copie dans `res/drawable/` Android (les noms de drawable n'acceptent pas les tirets).

## 🔤 Fonts — mapping vers le projet cible

Le DS livre les fonts dans `ds/assets/fonts/<font>/` en **deux formats** : `.woff2` (web) et `.ttf` (Flutter / iOS / Android natif). Selon la cible, copier le format adapté.

| Framework | Format à copier | Destination dans le projet | Déclaration |
|---|---|---|---|
| **Flutter** | `.ttf` | `assets/fonts/<font>/<font>-variable.ttf` | `pubspec.yaml` → `flutter: fonts: - family: <Font> fonts: - asset: assets/fonts/<font>/<font>-variable.ttf` |
| **iOS (SwiftUI / UIKit)** | `.ttf` ou `.otf` | Bundle (drag-and-drop dans Xcode, target sélectionné) | `Info.plist` → `UIAppFonts: [<font>-variable.ttf]` ; usage `Font.custom("<Font>", size: 16)` |
| **Android (Compose / Views)** | `.ttf` | `app/src/main/res/font/<font_snake>.ttf` (snake_case obligatoire) | XML font family ou `FontFamily(Font(R.font.<font_snake>))` en Compose |
| **Web (React, Vue, etc.)** | `.woff2` (priorité) + `.ttf` (fallback) | `public/assets/fonts/<font>/` ou via bundler (Vite, webpack) | `@font-face` en CSS, identique au mockup |

**Règles** :
- **Snake_case obligatoire pour Android** : `inter-variable.ttf` → `inter_variable.ttf` quand on copie dans `res/font/` (les noms de drawable/font Android n'acceptent pas les tirets)
- **Variable fonts côté Flutter** : utiliser `fontVariations: [FontVariation('wght', 600)]` plutôt que `fontWeight: FontWeight.w600` pour exploiter l'axe variable
- **iOS Info.plist** : le nom déclaré dans `UIAppFonts` doit matcher exactement le nom de fichier ; le nom utilisé dans `Font.custom(...)` est le **PostScript name** de la font (souvent différent du nom de famille — vérifier via `Font Book` ou `otfinfo -p font.ttf`)
- **Fallback stack** : reproduire la cascade du `tokens.css` côté code (si la font custom ne charge pas, retomber sur la font système locale)

## 🎯 Icônes — stratégie multi-framework

Si le kit utilise Lucide (ou autre famille), les noms de fichiers dans `ds/assets/icons/` sont **les noms canoniques** de la famille (ex. Lucide : `ellipsis-vertical.svg`, `trash-2.svg`, `triangle-alert.svg`, `clipboard-check.svg`). Le mapping est mécanique : `data-asset="ds/assets/icons/<name>.svg"` → composant `Name` de la librairie ou import asset depuis le fichier.

**Bonus mobile** : les fichiers `ds/assets/icons/<name>.svg` (un par icône, contenu Lucide canonique) sont directement copiables vers le projet — pas besoin de re-fetcher.

| Framework | Stratégie | Mapping depuis `data-asset="ds/assets/icons/<name>.svg"` |
|---|---|---|
| React | `lucide-react` package | `ellipsis-vertical` → `<EllipsisVertical />` (PascalCase) |
| Vue | `lucide-vue-next` | `ellipsis-vertical` → `<EllipsisVertical />` |
| Flutter | `flutter_lucide` package OU copier `ds/assets/icons/*.svg` dans `assets/icons/` + `flutter_svg` | `plus` → `LucideIcons.plus` ou `SvgPicture.asset('assets/icons/plus.svg')` |
| SwiftUI | SF Symbols si nom équivalent existe (`plus`, `trash`, `flag`…), sinon copier `ds/assets/icons/*.svg` dans le bundle | `plus` → `Image(systemName: "plus")` ou `Image("plus")` (asset) |
| Compose | Lucide Compose port OU copier `ds/assets/icons/*.svg` dans `res/drawable/` | `plus` → `Icons.Filled.Add` ou `painterResource(R.drawable.ic_plus)` |
| Web (vanilla) | Réutiliser le SVG inline du kit (avec ou sans data-asset) | `<svg ...><path .../></svg>` ou `<img src="/assets/icons/plus.svg">` |

**Choix de famille côté code = même famille que `ds/assets/icons/`**. Si le kit est en Lucide, ne pas mapper vers Material Icons côté Flutter — utiliser `flutter_lucide` ou copier les SVG depuis `ds/assets/icons/`.

Si `ds/assets/icons/` contient des icônes custom (status bar iOS, badges spéciaux) non-couvertes par la librairie cible : copier les SVG individuels depuis `ds/assets/icons/` vers les assets du projet et les rendre via le pipeline SVG natif du framework.

Toujours spécifier `size` + `color` explicitement, jamais inherit implicite.

## 🏷️ Conventions sémantiques — mapping multi-framework

Le kit pose des attributs HTML qui décrivent la **sémantique** d'un élément (pas juste son visuel). Un outil de code-gen consciencieux les lit AVANT de générer du code, pour produire une version idiomatique au lieu d'un mockup mort.

### `data-os-chrome` — éléments rendus par l'OS (à NE PAS coder)

Les éléments porteurs de `data-os-chrome="<type>"` (status-bar, dynamic-island, home-indicator, keyboard, gesture-bar) sont rendus par le système d'exploitation. **Le code mobile NE DOIT PAS générer de widget pour ces éléments** — c'est le job de la SafeArea du framework.

⚠️ **Héritage parent** : tous les descendants d'un nœud `data-os-chrome` sont skippés automatiquement. Ne pas suivre les `<svg>` enfants pour générer des assets fantômes (icônes signal/wifi/battery sont du chrome OS).

| Framework | Comment "skipper" |
|---|---|
| Flutter | Wrapper le body dans `SafeArea(child: ...)` (top + bottom) ; ne pas coder de status bar custom |
| SwiftUI | Pas besoin — `View` honore `safeAreaInsets` par défaut. Utiliser `.ignoresSafeArea()` seulement quand le design veut explicitement passer dessous |
| Compose | `Modifier.statusBarsPadding()` + `navigationBarsPadding()` ou utiliser un `Scaffold` qui les gère |
| React (web) | Pas applicable (pas de status bar) — skip total |

### `data-nav-target` / `data-nav-back` — routes à brancher

Format : `<flow_id>/<page_id>[:<state_id>]` ou `back`. Mapping direct vers la lib de routing du projet.

| Framework | Push vers `tasks/list:default` | Pop (back) |
|---|---|---|
| Flutter (go_router) | `context.push('/tasks/list')` (state passé en query si nécessaire) | `context.pop()` |
| Flutter (Navigator) | `Navigator.pushNamed(context, '/tasks/list')` | `Navigator.pop(context)` |
| SwiftUI | `path.append(.tasksList)` (NavigationStack) | `dismiss()` ou `path.removeLast()` |
| Compose (Navigation) | `navController.navigate("tasks/list")` | `navController.popBackStack()` |
| React (react-router) | `navigate('/tasks/list')` | `navigate(-1)` |
| Vue (vue-router) | `router.push('/tasks/list')` | `router.back()` |

**Règle** : un bouton sans `data-nav-target` est considéré non-navigant — ne PAS deviner sa destination. Si la maquette est ambiguë, alerter le designer pour qu'il pose l'attribut côté kit.

### `data-uses="native:<feature>"` — fonctions device

Sémantique pure : "cet élément déclenche une fonction native". L'agent codeur priorise un widget partagé du projet (ex: `MediaPicker()` shared), fallback sur le package natif. **Pas prescriptif** d'une lib particulière.

| Valeur | Flutter (priorité widget partagé > package) | SwiftUI | Compose | React Native |
|---|---|---|---|---|
| `native:image-picker` | shared `MediaPicker()` > `image_picker` | `PhotosPicker` | `ActivityResultContracts.PickVisualMedia` | shared > `expo-image-picker` |
| `native:camera` | shared `CameraView()` > `camera` | `Camera` / `AVCaptureSession` | `CameraX` | `expo-camera` |
| `native:date-picker` | `showDatePicker(...)` natif | `DatePicker` | `DatePicker` | `@react-native-community/datetimepicker` |
| `native:biometric-auth` | `local_auth` | `LocalAuthentication.LAContext` | `BiometricPrompt` | `expo-local-authentication` |
| `native:share-sheet` | `share_plus` | `ShareLink` / `UIActivityViewController` | `Intent.ACTION_SEND` | `Share.share(...)` |
| `native:location` | `geolocator` | `CLLocationManager` | `FusedLocationProviderClient` | `expo-location` |
| `native:contacts` | `flutter_contacts` | `CNContactStore` | `ContactsContract` | `expo-contacts` |

### `data-uses="ui:<pattern>"` — primitives UI framework

Sémantique : "cet élément suit un pattern UI au comportement non-évident". Sans cet attribut, l'agent recode en custom au lieu d'utiliser la primitive framework.

| Valeur | Flutter | SwiftUI | Compose | React |
|---|---|---|---|---|
| `ui:carousel` | `PageView` | `TabView(.page)` | `HorizontalPager` | shared `<Carousel>` ou `embla-carousel-react` |
| `ui:dialog` | `showDialog(...)` + `AlertDialog` | `.alert(...)` ou `.sheet(...)` | `AlertDialog` | `<Dialog>` (Radix, headless-ui) |
| `ui:bottom-sheet` | `showModalBottomSheet(...)` | `.sheet(.presentationDetents)` | `ModalBottomSheet` | `<Drawer>` mobile-first |
| `ui:popup-menu` | `PopupMenuButton` | `Menu { … }` | `DropdownMenu` | `<DropdownMenu>` |
| `ui:dot-indicator` | `SmoothPageIndicator` | manual `HStack(circles)` | manual `Row(circles)` | manual indicators |
| `ui:swipe-actions` | `Slidable` (`flutter_slidable`) | `.swipeActions { … }` | `SwipeToDismiss` | `react-swipeable-list` |
| `ui:pull-to-refresh` | `RefreshIndicator` | `.refreshable { … }` | `PullRefreshIndicator` | `RefreshControl` (RN) |
| `ui:tab-bar` | `TabBar` + `TabBarView` | `TabView` | `TabRow` | `<Tabs>` (Radix) |
| `ui:bottom-nav` | `BottomNavigationBar` | `TabView` (style: page) | `NavigationBar` | shared `<TabBar>` (mobile) |
| `ui:appbar` | `AppBar` (Material) / `CupertinoNavigationBar` | `NavigationStack` toolbar | `TopAppBar` | shared `<AppBar>` |
| `ui:navbar` | `BottomNavigationBar` / `NavigationBar` | `TabView` | `NavigationBar` | shared `<NavBar>` |

### `data-uses-context` — paramétrage de `data-uses`

Texte libre qui paramètre le `data-uses`. L'agent transmet ces params à l'API native :

```html
<button data-uses="native:image-picker"
        data-uses-context="multi-select, max:5, gallery only">
```

- Flutter : `ImagePicker().pickMultiImage(maxImages: 5, source: ImageSource.gallery)`
- SwiftUI : `PhotosPicker(maxSelectionCount: 5, matching: .images)`
- Compose : `PickMultipleVisualMedia(maxItems = 5, type = ImageOnly)`

### `data-hint` — sémantique métier libre

Texte libre qui décrit le comportement non visible côté visuel. **Pas de mapping mécanique** — à traiter au cas-par-cas comme annotation pour la génération.

```html
<!-- Hint : préférer la table BDD au pellicule device -->
<div class="gallery" data-hint="lit la table photos de la BDD utilisateur, pas la pellicule device">

<!-- Hint : comportement de swipe spécifique -->
<div class="task-row" data-hint="swipe gauche = archive, swipe droit = supprimer (avec confirm dialog)">

<!-- Hint : texte rotatif -->
<p data-hint="cycle entre 3 phrases toutes les 1.2s : 'Je lis…', 'J'extrais…', 'Je rédige…'">
```

L'agent privilégie un repository BDD au lieu d'un image_picker, code une logique de swipe différenciée, ou crée un cubit `RotatingTextCubit`. Ces hints comptent pour la qualité du code, pas pour le rendu visuel.

### `data-auto-advance` + `data-auto-advance-delay` — transitions temporisées

Posés sur la cell `[data-screen-label]` (PAS dans le `.phone__screen` extrait). Sémantique : "après X ms d'affichage, navigue automatiquement vers Y". Cas d'usage : splash → home, processing → result, toast auto-dismiss, tutorial steps.

| Framework | Implémentation |
|---|---|
| Flutter | `Timer(Duration(milliseconds: delay), () { if (mounted) context.push('/<target>'); })` dans `initState()` ; `_timer?.cancel()` dans `dispose()` |
| SwiftUI | `.task { try? await Task.sleep(for: .milliseconds(delay)); if !Task.isCancelled { path.append(target) } }` (cancel-on-disappear automatique) |
| Compose | `LaunchedEffect(Unit) { delay(delayMs); navController.navigate(target) }` (cancel-on-disposal automatique) |
| React | `useEffect(() => { const t = setTimeout(() => navigate(target), delay); return () => clearTimeout(t); }, [])` |

⚠️ **Toujours cancel le timer si le user navigue manuellement avant la fin** — sinon double navigation. Le pattern framework natif (LaunchedEffect, .task, useEffect cleanup, dispose) gère ça automatiquement quand le widget se démonte.

### Exemple complet — écran "magic processing" avec auto-advance + os-chrome + hint

**HTML source** :
```html
<div class="cell"
     data-screen-label="03 Magic processing"
     data-auto-advance="onboarding/magic:result"
     data-auto-advance-delay="3500">
  <div class="phone">
    <div class="phone__screen">
      <div data-os-chrome="status-bar">…</div>
      <div class="processing-content">
        <img src="ds/assets/illustrations/processing.svg">
        <h1>Analyse en cours…</h1>
        <p data-hint="cycle entre 3 phrases toutes les 1.2s">Je lis votre image…</p>
      </div>
      <div data-os-chrome="home-indicator"></div>
    </div>
  </div>
</div>
```

**Flutter (idiomatique avec wrappers du projet)** :
```dart
class OnboardingMagicProcessingPage extends StatefulWidget {
  const OnboardingMagicProcessingPage({super.key});
  @override State<OnboardingMagicProcessingPage> createState() => _State();
}

class _State extends State<OnboardingMagicProcessingPage> {
  Timer? _autoAdvance;
  late final RotatingTextCubit _rotating = RotatingTextCubit(
    phrases: const [
      'Je lis votre image',
      "J'extrais les ingrédients",
      'Je rédige la recette',
    ],
    interval: const Duration(milliseconds: 1200),
  );

  @override
  void initState() {
    super.initState();
    _autoAdvance = Timer(const Duration(milliseconds: 3500), () {
      if (mounted) context.push('/onboarding/magic/result');
    });
  }

  @override
  void dispose() {
    _autoAdvance?.cancel();
    _rotating.close();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final l10n = AppLocalizations.of(context)!;
    return Scaffold(
      backgroundColor: AppColors.bgLight,
      body: SafeArea( // ← gère data-os-chrome status-bar + home-indicator
        child: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              ImageView(imagePath: AppIllustrations.processing, width: 200),
              const SizedBox(height: 24),
              Text(l10n.onboardingProcessingTitle, style: AppTypography.h1),
              const SizedBox(height: 12),
              BlocBuilder<RotatingTextCubit, String>(
                bloc: _rotating,
                builder: (_, text) => Text(text, style: AppTypography.body),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

Ce qui a été lu côté HTML et traduit côté Dart :
- `data-os-chrome` × 2 → `SafeArea` (pas de widget custom pour status-bar / home-indicator)
- `data-auto-advance` + `data-auto-advance-delay` → `Timer` dans `initState` + `cancel` dans `dispose`
- `data-hint="cycle entre 3 phrases toutes les 1.2s"` → cubit dédié `RotatingTextCubit` (sémantique métier non visible côté CSS)
- `<img src="ds/assets/illustrations/processing.svg">` → wrapper projet `ImageView(imagePath: AppIllustrations.processing)` (pas `Image.asset` brut)

Sans la lecture des conventions sémantiques, l'agent aurait : codé une status bar custom morte, oublié l'auto-advance (écran statique), affiché une seule phrase fixe, utilisé `Image.asset` au lieu du wrapper. Quatre bugs invisibles avant intégration.

## 🧪 Exemple complet — List row

**HTML source** :
```html
<div class="member-row" data-nav-target="members/detail:default">
  <div class="member-row__avatar" style="background:var(--accent-warm)">
    M<span class="member-row__status member-row__status--online"></span>
  </div>
  <div class="member-row__body">
    <div class="member-row__line">
      <span class="member-row__name">Marie</span>
      <span class="member-row__role">(Admin)</span>
    </div>
    <div class="member-row__meta">Paris · il y a 2 min</div>
  </div>
  <svg class="member-row__chevron" data-asset="ds/assets/icons/chevron-right.svg"
       width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor"
       stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
    <path d="m9 18 6-6-6-6"/>
  </svg>
</div>
```

**React** :
```tsx
export function MemberRow({ member, onClick }: Props) {
  const { t } = useTranslation();
  return (
    <button className={styles.row} onClick={onClick}>
      <Avatar color={member.color} initial={member.initial} online={member.online} size={44} />
      <div className={styles.body}>
        <div className={styles.line1}>
          <span className={styles.name}>{member.name}</span>
          {member.isAdmin && <span className={styles.role}>({t('admin')})</span>}
        </div>
        <div className={styles.line2}>{member.location} · {formatRelative(member.lastSeen)}</div>
      </div>
      <ChevronRight size={20} color={theme.colors.gray} />
    </button>
  );
}
```

**Flutter** :
```dart
class MemberRow extends StatelessWidget {
  const MemberRow({super.key, required this.member, required this.onTap});
  final Member member;
  final VoidCallback onTap;

  @override
  Widget build(BuildContext context) {
    final l10n = AppLocalizations.of(context)!;
    return InkWell(
      onTap: onTap,
      child: Padding(
        padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 12),
        child: Row(
          children: [
            Avatar(color: member.color, initial: member.initial, online: member.online, size: 44),
            const SizedBox(width: 12),
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Row(children: [
                    Text(member.name, style: AppTypography.bodyBold),
                    if (member.isAdmin) ...[
                      const SizedBox(width: 4),
                      Text('(${l10n.admin})', style: AppTypography.caption.copyWith(color: AppColors.darkGray)),
                    ],
                  ]),
                  Text('${member.location} · ${member.lastSeen.humanize()}', style: AppTypography.small),
                ],
              ),
            ),
            Icon(LucideIcons.chevronRight, size: 20, color: AppColors.gray),
          ],
        ),
      ),
    );
  }
}
```

**Compose** :
```kotlin
@Composable
fun MemberRow(member: Member, onClick: () -> Unit) {
  Row(
    Modifier.fillMaxWidth().clickable(onClick = onClick)
      .padding(horizontal = 16.dp, vertical = 12.dp),
    verticalAlignment = Alignment.CenterVertically,
  ) {
    Avatar(color = member.color, initial = member.initial, online = member.online, size = 44.dp)
    Spacer(Modifier.width(12.dp))
    Column(Modifier.weight(1f)) {
      Row {
        Text(member.name, style = AppTypography.bodyBold)
        if (member.isAdmin) {
          Spacer(Modifier.width(4.dp))
          Text("(${stringResource(R.string.admin)})", style = AppTypography.caption, color = AppColors.darkGray)
        }
      }
      Text("${member.location} · ${member.lastSeen.humanize()}", style = AppTypography.small)
    }
    Icon(Icons.ChevronRight, null, tint = AppColors.gray)
  }
}
```

## 🚫 Pièges fréquents

1. **Copier des px HTML directement** → passer par l'échelle de spacing
2. **Créer un TextStyle inline** → utiliser le theme + `.copyWith`
3. **Oublier i18n** → tout texte via système
4. **Reproduire un gradient** qui n'existe pas dans le DS
5. **Shadow custom** → utiliser les presets
6. **Container/View partout** → préférer des primitives plus légères
7. **Ignorer dark mode** si le framework le gère
8. **Icon-only sans accessibilité** — toujours label/tooltip
9. **Dupliquer un composant existant** sous un autre nom
10. **Hardcoder du routing** → utiliser le routeur du projet
11. **Reverse-engineer un widget depuis le Flow** alors qu'il existe dans la section Composants de `UI Kit.html` — toujours partir du catalogue, pas de la composition
12. **Inliner un SVG path** côté code alors que `data-asset` pointe vers un fichier dispo dans `ds/assets/icons/` — passer par la librairie d'icônes ou par l'asset exporté
13. **Mapper les icônes vers une autre famille** que celle de `ds/assets/icons/` (ex. fichier Lucide → import Material) — créer une incohérence visuelle
14. **Ignorer l'attribut `data-asset`** lors du parsing de la maquette — c'est la clé qui dit explicitement quel fichier importer
15. **Coder des widgets pour le chrome OS** (status-bar, dynamic-island, home-indicator, keyboard) — utiliser SafeArea/safeAreaInsets, ne pas générer de Container custom
16. **Suivre les `<svg>` enfants d'un `data-os-chrome`** pour générer des assets — ces icônes (signal, wifi, battery) sont du chrome système, pas des assets app
17. **Recoder un pattern UI custom** quand `data-uses="ui:..."` désigne une primitive native — utiliser la primitive framework (PageView, BottomSheet, etc.)
18. **Forcer un package précis** quand `data-uses="native:..."` est posé — d'abord chercher un widget partagé dans le projet, le package natif est un fallback
19. **Deviner la destination d'un bouton** sans `data-nav-target` — un bouton non annoté est non-navigant, alerter le designer si la maquette est ambiguë
20. **Oublier l'auto-advance** sur un écran de splash/loading/processing — lire `data-auto-advance` sur la cell et installer un Timer/LaunchedEffect/.task avec cancel-on-dispose
21. **Ignorer les `data-hint`** qui décrivent une logique métier non-visible (rotation de texte, source BDD vs device, swipe différencié) — ces hints comptent pour la qualité du code généré

## ✅ Checklist finale

- [ ] Zéro hardcode de couleur/spacing/radius
- [ ] Tous les textes via i18n
- [ ] Composants existants réutilisés
- [ ] Chaque widget de la maquette mappé à une entrée de la section Composants de `UI Kit.html` (ou explicitement `[hors catalogue]`)
- [ ] Chaque icône mappée via `data-asset` vers un fichier `ds/assets/icons/<name>.svg` (ou explicitement `[hors catalogue]`)
- [ ] Famille d'icônes côté code = famille de `ds/assets/icons/` (pas de cross-mapping silencieux)
- [ ] Si copie d'assets SVG vers le projet : noms canoniques préservés (`ellipsis_vertical.svg` pas `more_vertical.svg`)
- [ ] State management branché correctement
- [ ] Navigation / routing intégrée
- [ ] États non-happy couverts (loading/empty/error)
- [ ] Accessibilité : labels, roles, hit targets ≥ 44px
- [ ] Dark mode géré si applicable
- [ ] Conventions de nommage du projet respectées
- [ ] Code idiomatique du framework (pas du JS déguisé en Dart)
- [ ] **`data-os-chrome`** géré via SafeArea / safeAreaInsets — aucun widget custom pour status-bar / home-indicator / keyboard / etc.
- [ ] **`data-nav-target`** branché au router du projet (push) ; **`data-nav-back`** → pop ; aucune destination devinée à la main
- [ ] **`data-uses="native:*"`** → widget partagé du projet en priorité, package natif en fallback (jamais une UI custom)
- [ ] **`data-uses="ui:*"`** → primitive framework (PageView, BottomSheet, etc.) plutôt que recodage custom
- [ ] **`data-uses-context`** transmis aux paramètres de l'API native quand présent
- [ ] **`data-auto-advance`** → Timer/LaunchedEffect/.task avec cancel-on-dispose ; tester que le timer s'annule en cas de nav manuelle
- [ ] **`data-hint`** → annotation prise en compte (texte rotatif, source de données, comportement de swipe…) ; pas ignorée silencieusement
- [ ] **Comparaison visuelle** : screenshot du Flow source côte-à-côte avec rendu du code généré (si possible)

## 📤 Format de livraison

```markdown
# Implémentation : [Nom écran] → [framework]

## Source
- HTML : `[fichier.html]` · cellule `[data-screen-label]`

## Fichiers créés / modifiés
- `[chemin/fichier.ext]` — [rôle]

## Composants
- ➕ Nouveaux : [liste]
- ♻️ Réutilisés : [liste]
- ❌ Supprimés : [liste]

## Tokens utilisés
[tableau CSS → code]

## États gérés
- ✅ Default / Loading / Error / [autres]
- ⚠️ [état non couvert + justification]

## Dépendances ajoutées
- `[package]` — [raison]

## Code
[diff ou fichiers complets]

## Checklist de review
[cocher ou noter les points à vérifier]

## Notes pour le dev
- [alertes, TODO, points d'attention]
```

## 🎓 Adaptation contextuelle

Ce skill est générique par design. Si le projet a un skill dédié plus spécifique (ex: un pipeline custom orienté Flutter ou un agent maison qui connaît les conventions exactes du repo), **le préférer** — il connaît les noms de fichiers et l'arborescence précise du codebase.

Ce skill sert quand :
- Le projet n'a pas de skill dédié
- Le framework cible est inhabituel
- On fait un prototype one-shot
