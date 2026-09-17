---
name: uikit-scene-lifecycle-transition
description: Migra apps UIKit do ciclo de vida baseado em AppDelegate para o ciclo de vida baseado em Scenes (UISceneDelegate/UIWindowSceneDelegate). Use quando o usuario pedir para adotar scene manifest, migrar lifecycle de AppDelegate, adicionar SceneDelegate, ou resolver warnings de UIScene lifecycle. Nao use para apps SwiftUI que ja usam App protocol.
---

# UIKit Scene-Based Lifecycle Transition

Migre apps UIKit do ciclo de vida legado (AppDelegate) para o ciclo de vida baseado em Scenes. O ciclo de vida por scenes e obrigatorio a partir de iOS 27, iPadOS 27, Mac Catalyst 27, tvOS 27 e visionOS 27. Apps que nao adotarem falham no launch. A partir de iOS 18.4 UIKit emite warning, a partir de iOS 26 o warning se torna mais grave.

## Procedimento Padrao

Siga os passos abaixo na ordem. Ao final, rode o Checklist de Verificacao.

### Passo 1 — Diagnosticar necessidade de migracao

Verifique se o app ja adotou scenes. Se qualquer item abaixo for verdadeiro, a migracao e necessaria:

1. O Info.plist nao contem a chave `UIApplicationSceneManifest`, ou ela existe mas nao tem configuracoes definidas.
2. O AppDelegate nao implementa `application(_:configurationForConnecting:options:)`.

Procure por esses dois sinais antes de comecar.

### Passo 2 — Configurar o Info.plist (Scene Manifest)

Adicione a chave `UIApplicationSceneManifest` ao Info.plist do app. Use o template em `assets/Info.plist-scene-manifest.xml`.

Regras:
- `UIApplicationSupportsMultipleScenes` — mantenha `false` a menos que o app precise de multiplas janelas (iPad Stage Manager, Side-by-Side). Habilitar multiplas scenes exige que o modelo de dados seja por-scene.
- `UISceneConfigurationName` — nome da configuracao. Use exatamente o mesmo nome no AppDelegate se implementar configuracao dinamica.
- `UISceneDelegateClassName` — normalmente `$(PRODUCT_MODULE_NAME).SceneDelegate`.
- `UISceneStoryboardFile` — inclua apenas se o root VC carregar via Storyboard. Se for programatico, omita essa chave.

Alternativa via Xcode:
1. Abra o projeto → selecione o target do app.
2. Aba General → Deployment Info → marque "Scene manifest".

### Passo 3 — Implementar SceneDelegate

Crie um arquivo `SceneDelegate.swift` (ou nome correspondente ao configurado no Info.plist). Use o template em `assets/SceneDelegate.swift`.

Requisitos:
- Subclasse de `UIResponder`.
- Implementa `UIWindowSceneDelegate`.
- Possui propriedade `var window: UIWindow?`.
- Implementa `scene(_:willConnectTo:options:)` para criar a window e definir o rootViewController (apenas se for carregamento programatico; se usar Storyboard, o sistema faz isso automaticamente).

### Passo 4 — Atualizar AppDelegate

O AppDelegate continua existindo para lidar com o ciclo de vida do processo. Use o template em `assets/AppDelegate.swift`.

Mudancas obrigatorias:
1. Remova a propriedade `var window: UIWindow?` do AppDelegate — ela agora pertence ao SceneDelegate.
2. Se precisar de configuracao dinamica de scenes (ex: diferentes scenes baseadas em NSUserActivity), implemente `application(_:configurationForConnecting:options:)`.
3. Mantenha apenas logica global do processo no AppDelegate: inicializacao de SDKs, registro de push, configuracao global. Tudo que for de UI vai para o SceneDelegate.

### Passo 5 — Migrar metodos de ciclo de vida

Mova a logica dos metodos abaixo de AppDelegate para SceneDelegate. Cada metodo agora e por-scene.

| AppDelegate (legado)              | SceneDelegate (scene-based)        |
|-----------------------------------|------------------------------------|
| applicationDidBecomeActive(_:)    | sceneDidBecomeActive(_:)           |
| applicationWillResignActive(_:)   | sceneWillResignActive(_:)          |
| applicationDidEnterBackground(_:) | sceneDidEnterBackground(_:)        |
| applicationWillEnterForeground(_:)| sceneWillEnterForeground(_:)       |

Importante: depois da migracao, UIKit NAO chama mais os metodos de AppDelegate, mesmo que continuem implementados. Se precisar do estado agregado de todo o app (ex: "alguma scene esta em foreground?"), use as notificacoes de `UIApplication` como `willEnterForegroundNotification` e a propriedade `UIApplication.shared.applicationState`.

### Passo 6 — Suportar external display (se aplicavel)

A partir de iOS 27, a scene `windowExternalDisplayNonInteractive` nao e mais conectada automaticamente. O app deve registrar um `UISceneAccessory`.

Se o app tinha logica checando `windowExternalDisplayNonInteractive` em `application(_:configurationForConnecting:options:)` ou no SceneDelegate:
1. Remova essa logica especifica de role.
2. Em um view controller da interface principal, registre `UISceneAccessory.externalNonInteractive(sceneConfiguration:)`.
3. Use um delegate dedicado ou reutilize o SceneDelegate existente para configurar a janela do display externo.

Veja exemplo de codigo na documentacao em `references/apple-transitioning-to-scene-lifecycle.md`.

### Passo 7 — Testes

Teste o app nas seguintes configuracoes, especialmente em iPad:
- Full Screen Apps
- Windowed Apps
- Stage Manager
- Rotacao de tela
- Background / Foreground transitions
- (Se multiplas scenes habilitadas) abrir multiplas janelas

## Checklist de Verificacao

Marque todos os itens antes de considerar a migracao concluida:

- [ ] Info.plist contem `UIApplicationSceneManifest` com ao menos uma configuracao de `UIWindowSceneSessionRoleApplication`.
- [ ] Arquivo SceneDelegate existe, herda de `UIResponder`, implementa `UIWindowSceneDelegate`.
- [ ] `window: UIWindow?` foi movido de AppDelegate para SceneDelegate.
- [ ] `scene(_:willConnectTo:options:)` cria a window e define rootViewController (se programatico).
- [ ] Metodos de ciclo de vida foram migrados para o SceneDelegate.
- [ ] Nenhuma logica de configuracao de window permanece no AppDelegate.
- [ ] App compila sem warnings de "This process does not adopt UIScene lifecycle".
- [ ] App lanca corretamente em iOS 18.4+, iOS 26 e iOS 27+.
- [ ] Push notifications, deep links, URL schemes e universal links continuam funcionando.
- [ ] (Opcional) External display content funciona apos registrar UISceneAccessory.

## Assets e Referencias

- Snippets XML para Info.plist: `assets/Info.plist-scene-manifest.xml`
- Templates Swift: `assets/AppDelegate.swift`, `assets/SceneDelegate.swift`
- Documentacao Apple original em Markdown: `references/apple-transitioning-to-scene-lifecycle.md`
