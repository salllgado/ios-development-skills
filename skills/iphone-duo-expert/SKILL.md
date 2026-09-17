---
name: "iphone-duo-expert"
description: "Desenvolvimento e adaptacao de apps iOS para iPhone Duo foldable. Invoke ao adaptar layout, usar APIs foldable (ArrangementView, onHingeChange, Reserved Regions, CameraCaptureAccessory), migrar NavigationSplitView, corrigir safe areas, testar poses ou rodar auditoria Duo."
---

# iPhone Duo Expert — SwiftUI + UIKit (iOS 27.1+)

Autoridade tecnica para desenvolvimento de apps voltados ao **iPhone Duo** — primeiro foldable da Apple, anunciado 09/Set/2026, lancamento 23/Out/2026 rodando **iOS 27.1**.
Incorpora as 6 Tech Talks oficiais Apple (111461 a 111466), Session WWDC26 269 (What is New in SwiftUI) e os padroes do `swiftui-expert-skill`.

---

## Modo de Uso Rapido

Sempre que invocar esta skill, siga ESTA ordem:

1. **Stage 0 (Auditoria Obrigatoria)** — Rode o checklist abaixo ANTES de qualquer outra mudanca.
2. **Stage 1 (Quick Wins)** — Correcoes de baixo esforco / alto impacto.
3. **Stage 2 (Layout Adaptativo)** — Migre para containers adaptativos.
4. **Stage 3 (APIs Foldable Nativas)** — Adote APIs especificas do Duo quando houver ganho.
5. **Stage 4 (Validacao)** — Teste nas 4 poses + transicoes.

---

## Stage 0 — Auditoria Obrigatoria (grep no projeto inteiro)

Antes de escrever 1 linha, busque e reporte estes padroes:

```bash
# Padroes PROIBIDOS no iPhone Duo
rg "UIScreen\.main"                 --type swift   # AMBIGUO (2 displays)
rg "UIDevice\.current\.orientation" --type swift   # MENTIROSO (display interno ignora orientation lock)
rg "\.interfaceOrientation"         --type swift
rg "safeAreaInsets\.left\s*\*\s*2"  --type swift   # ASSIMETRICO QUEBRADO
rg "safeAreaInsets\.right\s*\*\s*2" --type swift
rg "supportedInterfaceOrientations" --type swift
rg "\.cornerRadius\(\s*1[0-9]\.?[0-9]*" --type swift  # Cantos fixos
rg "CGSize\(width:\s*3[5-9][0-9]"   --type swift   # Width fixo estilo iPhone
```

Regra de ouro: se encontrar `UIScreen.main` ou `UIDevice.current.orientation`, marque como **P0 (bug garantido)** e substitua no Stage 1.

### Tabela de Comportamento por SDK (sem rebuild)

| Build com | Comportamento display interno |
|---|---|
| SDK antigo (< 27.0) | Roda, mas LETTERBOXED (barras pretas) |
| iOS 27.0 SDK | Estende esquerda da status bar, evita area da camera |
| **iOS 27.1 SDK** | **Full edge-to-edge, barras verticais automaticas** |

A primeira acao e SEMPRE confirmar build com **Xcode 27.1** + SDK **iOS 27.1**.

---

## Stage 1 — Quick Wins (baixo esforco, alto impacto)

### 1.1 Substitua UIScreen.main e Orientation

```swift
// ❌ ANTES (QUEBRA no Duo)
let scale  = UIScreen.main.scale
let bounds = UIScreen.main.bounds
if UIDevice.current.orientation == .landscape { WideLayout() }

// ✅ DEPOIS

// SwiftUI
@Environment(\.horizontalSizeClass) private var hSize
@Environment(\.verticalSizeClass)   private var vSize
@Environment(\.displayScale)        private var scale

Group {
    if hSize == .regular { WideLayout() }
    else                 { CompactLayout() }
}

// UIKit
override func traitCollectionDidChange(_ previous: UITraitCollection?) {
    super.traitCollectionDidChange(previous)
    let scale  = traitCollection.displayScale
    let screen = view.window?.windowScene?.screen
    let isWide = traitCollection.horizontalSizeClass == .regular
}
```

### 1.2 Corrija Safe Areas Assimetricas

No Duo, insets esquerdo e direito QUASE NUNCA sao iguais (camera + Split View + dobradica).

```swift
// ❌ ANTES (QUEBRA)
let usableWidth = view.bounds.width - view.safeAreaInsets.left * 2

// ✅ DEPOIS
let usableWidth = view.bounds.inset(by: view.safeAreaInsets).width
```

Regra Apple (Tech Talk 111461 @ 6:06):
- **Conteudo interativo foreground** -> PERMANECE dentro do safe area.
- **Backgrounds, artes, gradientes** -> `.ignoresSafeArea()` (extrapole).

```swift
// ✅ Padrao correto (SwiftUI)
ZStack {
    LinearGradient(colors: [.blue, .purple], startPoint: .top, endPoint: .bottom)
        .ignoresSafeArea()          // background: sangra

    VStack {
        Button("Comprar") {}
            .buttonStyle(.borderedProminent)
    }
}
```

### 1.3 Cantos Corretos com ConcentricRectangle (iOS 26+)

Use em vez de `.cornerRadius(12)` fixo. Os cantos fisicos do Duo nao sao do mesmo tamanho do iPhone 18.

```swift
// SwiftUI
ConcentricRectangle()
    .fill(.green)
    .padding(8)
    .ignoresSafeArea()

// UIKit
let cfg = UICornerConfiguration(curve: .circular, radius: view.layer.cornerRadius)
```

### 1.4 ForEach com Identidade Estavel (swiftui-expert-skill)

```swift
// ❌ ANTES — reorder nativo .reorderable() quebra com indices
ForEach(items.indices) { i in Text(items[i]) }

// ✅ DEPOIS — id estavel (sobrevive a drag and drop reorder)
ForEach(items, id: \.stableId) { item in Text(item.title) }
```

---

## Stage 2 — Layout Adaptativo (4 poses)

### 2.1 O Modelo Mental: 4 Poses, Nao 2 Orientacoes

| Pose | Descricao | horizontalSizeClass | verticalSizeClass |
|---|---|---|---|
| Fechado Portrait | Display externo, padrao iPhone | `compact` | `regular` |
| Fechado Landscape | Externo deitado | `compact` | `compact` |
| **Aberto Vertical** | Interno na vertical (quase quadrado) | **regular** | **regular** |
| **Aberto Horizontal** | Interno deitado (largo) | **regular** | **regular** |

Fato novo: iPhone Duo reporta `regular x regular` num iPHONE. Todo codigo que branchava em `userInterfaceIdiom == .pad` para ativar sidebar agora deve branchar em `horizontalSizeClass == .regular`.

### 2.2 NavigationSplitView — Maior ROI unico

`NavigationSplitView` (iOS 16+) adapta-se SOZINHO. No aberto = sidebar + detalhe; no fechado = push/pop normal.

```swift
@State private var columnVisibility: NavigationSplitViewVisibility = .automatic
@State private var selected: Tagging?
@State private var path = NavigationPath()

NavigationSplitView(columnVisibility: $columnVisibility) {
    TaggingListView(selected: $selected)
        .navigationTitle("Meus Cards")
        .navigationSplitViewColumnWidth(min: 280, ideal: 320)
} detail: {
    NavigationStack(path: $path) {
        if let s = selected {
            TaggingDetail(tagging: s)
                .navigationDestination(for: Route.self) { route in
                    route.view()
                }
        } else {
            ContentUnavailableView(
                "Selecione um card",
                systemImage: "creditcard",
                description: Text("Abra o iPhone Duo para ver mais detalhes.")
            )
        }
    }
}
.navigationSplitViewStyle(.adaptiveSidebar)   // ✅ iOS 27.1+
```

Restricao: se aninhar `NavigationSplitView` DENTRO de um `TabView`, embrulhe o `detail` com `NavigationStack`.

### 2.3 TabView com Sidebar Automatico

```swift
TabView(selection: $tab) {
    HomeView()      .tabItem { Label("Inicio",   systemImage: "house") }
    ContactsView()  .tabItem { Label("Contatos", systemImage: "person.2") }
    SearchView()    .tabItem { Label("Buscar",   systemImage: "magnifyingglass") }
    ProfileView()   .tabItem { Label("Perfil",   systemImage: "person.crop.circle") }
}
// ✅ Duo aberto => sidebar esquerda; fechado => TabBar inferior normal
.defaultTabBarPlacement(.sidebar)
```

---

## Stage 3 — APIs Foldable Nativas (iOS 27.1+)

Todas as APIs abaixo exigem `#available(iOS 27.1, *)` + fallback para versoes anteriores.

### 3.1 ArrangementView — Dividir Atraves da Dobradica

Container primitivo novo. Recebe `primary` + `secondary`; organiza automaticamente.

```swift
@available(iOS 27.1, *)
struct TaggingArrangementView: View {
    var body: some View {
        NavigationStack {
            ArrangementView {
                // Primary: lado A — NUNCA e obscurecido
                TaggingListView()
            } secondary: {
                // Secondary: lado B
                TaggingPreviewView()
            }
            // .split   => HStack/VStack automatico (nenhum view sobrepoe)
            // .overlay => ZStack (secondary atras do primary; parcialmente aceitavel obscuracao)
            .arrangementViewStyle(.split)
            .navigationTitle("Cartao")
        }
    }
}

// Fallback para iOS < 27.1
struct ArrangementFallbackView: View {
    var body: some View {
        if #available(iOS 27.1, *) {
            TaggingArrangementView()
        } else {
            HStack(spacing: 0) {
                TaggingListView()     .frame(maxWidth: .infinity)
                Divider()
                TaggingPreviewView()  .frame(maxWidth: .infinity)
            }
        }
    }
}
```

🚫 **2 Restricoes de Ouro (Tech Talk 111463):**
1. NAO aninhe `NavigationSplitView` dentro de `ArrangementView`.
2. NAO coloque `ArrangementView` dentro de `List`, `ScrollView` ou qualquer container rolavel.

### 3.2 Reserved Regions API — Evite Dobradica e Camera

Dois tipos de regiao reservada:
- `.division`  => a DOBRADICA (width > 0 apenas parcialmente fechado; zero quando plano)
- `.occlusion` => CAMERA under-display + furos

```swift
@available(iOS 27.1, *)
struct SafeCTALayout: View {
    var body: some View {
        GeometryReader { proxy in
            let folds   = proxy.reservedRegions(kind: .division)
            let cameras = proxy.reservedRegions(kind: .occlusion)

            let foldBottom = (folds.first?.frame.maxY ?? 0)
            let camWidth   = (cameras.first?.frame.width ?? 0)

            VStack {
                Spacer()
                Button("Compartilhar Cartao") { }
                    .buttonStyle(.borderedProminent)
                    .controlSize(.large)
                    .padding(.bottom, foldBottom + 24)
            }
            .padding(.horizontal, camWidth + 16)
            .frame(maxWidth: .infinity, maxHeight: .infinity)
        }
    }
}
```

Regra: **deslocamento, nao centralizacao**. Mova CTAs e botoes longe das regioes reservadas — NAO centralize tudo no meio da tela.

### 3.3 onHingeChange — Entrada Analogica

Use para EFEITOS, NAO para layout decisorio (para layout: size classes + reserved regions).

```swift
@available(iOS 27.1, *)
struct ParallaxCardView: View {
    @State private var parallax: CGFloat = 0

    var body: some View {
        TaggingCard3DView()
            .offset(x: parallax)
            .onHingeChange { _, context in
                // hinge e nil em dispositivos SEM dobradica
                guard let hinge = context.hinge else {
                    withAnimation(.interactiveSpring) { parallax = 0 }
                    return
                }

                switch hinge.status {
                case .partiallyOpen:
                    // Normalize 0..120 graus => -20..+20 pts offset
                    let t = min(max(hinge.angle.degrees / 120.0, 0), 1)
                    parallax = (t - 0.5) * 40
                case .closed, .open:
                    withAnimation(.interactiveSpring) { parallax = 0 }
                @unknown default:
                    parallax = 0
                }
            }
    }
}
```

Ideias de uso:
- Cartao de visita com parallax / 3D
- Controles finos (exposure, pitch bend musical)
- Efeito de page curl parcial

### 3.4 Scene Accessories + CameraCaptureAccessory — Dois Displays, Dois Conteudos

Revolucionario. Enquanto seu app roda FULL SCREEN no display INTERNO, voce envia um conteudo DIFERENTE para o EXTERNO.

Caso canonico YOOU (cartao):

```swift
@available(iOS 27.1, *)
struct ShareCardRootView: View {
    @StateObject private var model = ShareCardModel()
    @State private var showExternal = false

    var body: some View {
        // INTERNO: editor, formularios, QR, controles
        ShareCardEditor(model: model)
            .sceneAccessory {
                // EXTERNO: visual limpo do cartao para o cliente
                CameraCaptureAccessory(isActive: showExternal) {
                    ExternalCardPreview(model: model)
                        .font(.system(size: 64, weight: .semibold))
                        .preferredColorScheme(.light)
                        .statusBarHidden()
                }
            }
            .toolbar {
                Toggle("Preview externo", isOn: $showExternal)
                    .toggleStyle(.switch)
            }
    }
}
```

Casos de uso de alto valor para apps empresariais:
- Cartao de visita: editor interno + preview elegante externo
- Tagging PRO: formulario interno + "Aguardando aprovacao..." externo
- Checkout / POS: total + QR pagamento externo; itens internos
- Palestras/cursos: notas interno; slide limpo externo

---

## Stage 4 — Toolbars e Barras Verticais

No display largo do Duo, Navigation/Tab/Toolbars MUDAM AUTOMATICAMENTE para laterais (barras verticais).

### 4.1 visibilityPriority — Controle de Sobrevivencia

```swift
Toolbar {
    // Permanente (sobrevive a barra cheia)
    ToolbarItem(placement: .topBarPinnedTrailing) {
        ShareLink(item: cardUrl) {
            Label("Compartilhar", systemImage: "square.and.arrow.up")
        }
        .visibilityPriority(.high)
    }

    ToolbarItem {
        Button(action: favorite) { Label("Favoritar", systemImage: "star") }
        .visibilityPriority(.high)
    }

    ToolbarItem {
        Button(action: edit) { Label("Editar", systemImage: "pencil") }
        .visibilityPriority(.medium)
    }

    // Baixa prioridade -> cai primeiro no overflow
    ToolbarItem { Button(action: archive) { Label("Arquivar", systemImage: "archivebox") } }
    ToolbarItem { Button(action: report)  { Label("Reportar",  systemImage: "flag") } }

    // ✅ Todo item que nao coube aparece automaticamente aqui
    ToolbarOverflowMenu {
        Button("Exportar PDF") { }
        Button("Imprimir")   { UIPrintInteractionController.shared.present(...) }
    }

    // ✅ Minimiza automaticamente no scroll
    ToolbarItem(id: "search") { SearchField(text: $query) }
}
.toolbarMinimizeBehavior(.automatic, for: [.search, .title])
```

### 4.2 Sempre Forneca TITULO + SF Symbol

O sistema escolhe automaticamente a representacao baseada no espaco. Barra vertical exibe so o icone.

```swift
// ✅ Correto
.tabItem { Label("Contatos", systemImage: "person.2") }
// ❌ Errado
.tabItem { Text("Contatos") }   // fica VAZIO na barra vertical
```

---

## Stage 5 — Validacao: Checklist de Testes

Execute SEMPRE estas 8 checagens no Xcode 27.1 Device Hub:

| # | Teste | O que validar |
|---|---|---|
| T1 | Fechado Portrait | Igual a qualquer iPhone. Se quebrar aqui, ja estava quebrado. |
| T2 | Fechado Landscape | Barras ok, conteudo sem corte. |
| T3 | Aberto Vertical (regular x regular) | Sidebar aparece; CTAs fora da area da camera. |
| T4 | Aberto Horizontal | Mesmo que T3; largura extra bem aproveitada. |
| T5 | Transicao Fechado -> Aberto AO VIVO | Estado PRESERVADO (texto digitado, scroll, selecao, form). |
| T6 | Split View multitasking (interno) | App na ESQUERDA e na DIREITA (safe areas invertem). |
| T7 | Fold parcial 30/60/90/120 graus | Nenhum CTA cai na dobradica; onHingeChange com resposta fluida. |
| T8 | Barra vertical | Todos itens tem icone; nenhum depende so de texto. |

### Como usar o Simulador iPhone Duo

1. Xcode 27.1 -> Window -> Device and Simulators -> Simulators -> + -> iPhone Duo
2. Toolbar do simulador tem um **botao de dobradica** (toggle Fold/Unfold)
3. Para angulos finos: **Device Hub** -> Posture -> arraste o slider
4. View Debugger -> ative **Fold State Overlay** (pinta regioes quebradas em cor solida)

---

## Stage 6 — UIKit Bridge

| SwiftUI | UIKit Equivalente (iOS 27.1+) |
|---|---|
| `ArrangementView` | `UIArrangementViewController(primary:secondary:)` |
| `onHingeChange` | `UIHingeInteraction(delegate:)` com `hingeDidChange(_:)` |
| `GeometryReader.reservedRegions(kind:)` | `UIView.reservedRegions(kind:)` -> `[UIReservedRegion]` |
| `CameraCaptureAccessory` | `UIWindowScene` + `UISceneAccessory` + `UISCameraCaptureAccessory` |
| `.defaultTabBarPlacement(.sidebar)` | `UITabBarController.sidebar.preferredPlacement = .sidebar` |
| `ConcentricRectangle` | `UICornerConfiguration` em `CALayer` |

Mesmo em UIKit: SEMPRE use `traitCollection` e `view.window?.windowScene?.screen`. NUNCA `UIScreen.main`.

---

## Hard Constraints (Inquebraveis)

1. ❌ Nao use `UIScreen.main` — ambíguo em 2 displays.
2. ❌ Nao use `UIDevice.current.orientation` ou `.interfaceOrientation` — display interno ignora orientation lock.
3. ❌ Nao presuma safe areas simetricas.
4. ❌ Nao aninhe `NavigationSplitView` dentro de `ArrangementView`.
5. ❌ Nao coloque `ArrangementView` dentro de `List`/`ScrollView`.
6. ✅ Sempre forneca TITULO + SF SYMBOL em itens de toolbar/tab.
7. ✅ Sempre gire APIs 27.1+ com `#available(iOS 27.1, *)` + fallback.
8. ✅ Use **size classes**, nao `userInterfaceIdiom == .pad`, para ativar sidebar.

---

## Correlacao com swiftui-expert-skill

Aplique estas regras tambem (tornam-se mais criticas no Duo):

| Topico skill irmao | Impacto no Duo |
|---|---|
| Identidade ForEach estavel (list-patterns) | Reorder nativo `.reorderable()` depende de IDs estaveis |
| View Composition (view-structure) | Extraia subviews cedo — cada pose gera diff grande |
| State + @Observable (state-management) | Preservar estado em T5 (transicao fold/unfold) exige estado correto |
| Modifier identity (modifier-patterns) | Modifiers repetidos causam hitch nas transicoes fold/unfold |
| Performance patterns (performance-patterns) | Transicao fold/unfold e animacao cara — hot paths devem ser limpos |
| Toolbar patterns (toolbar-patterns) | Barras verticais sao novo padrao |
| Navigation patterns (sheet-navigation-patterns) | NavigationSplitView e a base |
| Accessibility patterns (accessibility-patterns) | VoiceOver focus nao deve pular para reserved regions |

---

## Referencias Oficiais Apple (citaveis)

| Sessao | Codigo | Titulo |
|---|---|---|
| Tech Talk | 111461 | Prepare your app for iPhone Duo |
| Tech Talk | 111462 | Raise the bar with iPhone Duo (barras verticais) |
| Tech Talk | 111463 | Strike a pose with adaptive layouts on iPhone Duo |
| Tech Talk | 111464 | Leverage multiple displays and scenes on iPhone Duo |
| Tech Talk | 111465 | Build a great camera experience for iPhone Duo |
| Tech Talk | 111466 | Design for iPhone Duo |
| WWDC26  | 269   | What's new in SwiftUI (reorder, docs, toolbars, error binding) |
| WWDC26  | 278   | Modernize your UIKit app |
| Portal  | —     | developer.apple.com/iphone-duo |

---

## Modo Auditoria Expressa (formato de resposta padrao)

Quando o usuario apenas disser "verifique meu app pro iPhone Duo", retorne SEMPRE nesta estrutura:

```
### Auditoria iPhone Duo — Resultado

#### P0 — BUGS GARANTIDOS (corrigir ja)
- (X) ocorrencias de `UIScreen.main` em [Arquivo1](), [Arquivo2]()
- (Y) ocorrencias de `UIDevice.current.orientation`
- (Z) safe areas simetricas (`* 2`) quebradas

#### P1 — Layout Subotimo (1 dia de trabalho)
- [ ] Migrar NavigationStack -> NavigationSplitView no root (arquivo)
- [ ] Toolbar itens sem SF Symbol (lista)
- [ ] ForEach com .indices / .offset (arquivos)

#### P2 — Otimizacoes Opcionais (baixa prioridade)
- Adotar ArrangementView na tela X
- onHingeChange para efeito Y
- SceneAccessory + CameraCaptureAccessory no fluxo Z

💡 Esforco estimado total: P0 = (X)h, P1 = (Y)h, P2 = (Z)h
```

---
Fim do SKILL iphone-duo-expert.
