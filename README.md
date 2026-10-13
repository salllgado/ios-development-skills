# iOS Development Skills

Coleção de skills (instruções estruturadas para agentes de IA) focadas em desenvolvimento iOS com UIKit e SwiftUI.

Cada skill neste repositório é um pacote auto-contido com procedimento passo-a-passo, templates de código e referências de documentação da Apple. As skills são agnósticas de projeto e funcionam com múltiplas IDEs / agentes de IA.

---

## Skills disponíveis

| Skill | Descrição |
|-------|-----------|
| [uikit-scene-lifecycle-transition](skills/uikit-scene-lifecycle-transition/) | Migra apps UIKit do ciclo de vida baseado em AppDelegate para o ciclo de vida baseado em Scenes (UISceneDelegate / UIWindowSceneDelegate). Obrigatório a partir de iOS 27. |
| [iphone-duo-expert](skills/iphone-duo-expert/) | Adaptação de apps iOS (SwiftUI + UIKit) para o iPhone Duo foldable. Auditoria obrigatória, NavigationSplitView, APIs foldable iOS 27.1 (ArrangementView, Reserved Regions, onHingeChange, CameraCaptureAccessory), safe areas assimétricas, barras verticais, checklist de 8 testes e formato de resposta auditoria P0/P1/P2. Baseado nas 6 Tech Talks oficiais Apple 111461–111466 e WWDC26 Session 269. |
| [swiftui-expert](skills/swiftui-expert/) | Autoridade técnica em SwiftUI (iOS + macOS): fluxo de dados com `@State`/`@Observable`, composição de views, performance e diffing, listas e `ForEach`, environment, layout, navegação, animações, acessibilidade, Swift Charts, localização, Liquid Glass e migração de APIs. Inclui referências por tópico e scripts para captura e análise de traces do Instruments. |

---

## Como instalar

Escolha a sua IDE / agente de IA abaixo e execute o comando correspondente no terminal.

### 1. TRAE

O TRAE (edição Internacional) carrega skills do diretório `~/.trae/skills/`.

```bash
# Clone o repo (se ainda não tiver) ou atualize
cd ~ && git clone https://github.com/salllgado/ios-development-skills.git 2>/dev/null || true

# Copie a skill desejada para a pasta global de skills do TRAE
# Exemplo para a skill uikit-scene-lifecycle-transition:
cp -R ~/ios-development-skills/skills/uikit-scene-lifecycle-transition \
  ~/.trae/skills/uikit-scene-lifecycle-transition
```

Reinicie o TRAE (ou recarregue a janela) para que a skill seja carregada.

#### Versão CN do TRAE

Se você usa a edição chinesa do TRAE, substitua `.trae` por `.trae-cn` no caminho:

```bash
cp -R ~/ios-development-skills/skills/uikit-scene-lifecycle-transition \
  ~/.trae-cn/skills/uikit-scene-lifecycle-transition
```

#### Skill por projeto (escopo local)

Para habilitar a skill apenas em um repositório iOS específico (qualquer pessoa que clonar o repo já ganha a skill automaticamente):

```bash
cd /caminho/do/seu/projeto-ios
mkdir -p .trae/skills
cp -R ~/ios-development-skills/skills/uikit-scene-lifecycle-transition \
  .trae/skills/uikit-scene-lifecycle-transition
```

---

### 2. Cursor

O Cursor suporta a adição de "Custom Instructions" e repositórios de contexto. Para usar essas skills:

```bash
# Clone o repo
cd ~ && git clone https://github.com/salllgado/ios-development-skills.git 2>/dev/null || true

# (Opcional) Adicione a pasta de skills como workspace folder no Cursor
# Para isso, abra o Cursor, use Cmd+Shift+P > "Add Folder to Workspace"
# e selecione ~/ios-development-skills
```

**Alternativa via linha de comando (symlink para fácil atualização):**

```bash
# (macOS) Crie um symlink dentro do seu projeto iOS
cd /caminho/do/seu/projeto-ios
ln -sf ~/ios-development-skills/skills .ios-skills
```

Agora ao conversar com o Cursor, basta referenciar os arquivos `SKILL.md` como contexto. Exemplo de prompt:

> Use o procedimento em `.ios-skills/uikit-scene-lifecycle-transition/SKILL.md` para migrar meu AppDelegate para SceneDelegate. Verifique primeiro se eu já tenho o UIApplicationSceneManifest no Info.plist.

---

### 3. Claude Code (CLI)

O Claude Code CLI carrega contexto através de `--context` ou arquivos locais.

```bash
# Clone o repo
cd ~ && git clone https://github.com/salllgado/ios-development-skills.git 2>/dev/null || true
```

Opção A — Passe a skill como contexto em cada chamada:

```bash
claude --context ~/ios-development-skills/skills/uikit-scene-lifecycle-transition/SKILL.md
```

Opção B — Instale globalmente via symlink para o diretório de skills do Claude Code (se o seu setup usar):

```bash
# Ajuste o path de destino conforme a sua instalação do Claude Code
# Exemplo (ajuste conforme necessário):
mkdir -p ~/.config/claude-code/skills
ln -sf ~/ios-development-skills/skills/uikit-scene-lifecycle-transition \
  ~/.config/claude-code/skills/uikit-scene-lifecycle-transition
```

Opção C — Dentro do seu projeto iOS, crie um atalho via arquivo `.claude`:

```bash
cd /caminho/do/seu/projeto-ios
echo 'Contexto iOS skills disponíveis em ~/ios-development-skills/skills/' > .claude
```

---

### 4. Kiro

O Kiro carrega skills a partir do diretório de configuração do usuário ou via contexto inline.

```bash
# Clone o repo
cd ~ && git clone https://github.com/salllgado/ios-development-skills.git 2>/dev/null || true

# Copie as skills para o diretório de skills do Kiro
# Ajuste o path abaixo conforme a sua instalação do Kiro (macOS):
KIRO_SKILLS_DIR="$HOME/Library/Application Support/Kiro/skills"
mkdir -p "$KIRO_SKILLS_DIR"
cp -R ~/ios-development-skills/skills/uikit-scene-lifecycle-transition \
  "$KIRO_SKILLS_DIR/uikit-scene-lifecycle-transition"
```

Se o Kiro da sua instalação usar um diretório diferente, substitua `$KIRO_SKILLS_DIR` pelo caminho correto. Como alternativa, basta abrir a pasta `~/ios-development-skills/` como projeto no Kiro ou anexar o `SKILL.md` desejado como contexto em uma conversa.

---

## Atualizando as skills

```bash
cd ~/ios-development-skills && git pull --ff-only
```

Depois re-copie ou re-symlink a skill atualizada para o diretório da sua IDE conforme os comandos acima.

---

## Estrutura de uma skill

Cada pasta em `skills/` segue o formato padrão:

```
skills/<skill-name>/
├── SKILL.md           ← Procedimento, regras e gatilhos
├── assets/            ← Templates de código, snippets XML
├── references/        ← Documentação de referência (ex: artigos Apple em Markdown)
└── scripts/           ← (opcional) Scripts auxiliares
```

---

## Contribuindo

1. Fork do repositório
2. Crie uma branch: `git checkout -b feat/nova-skill`
3. Adicione a pasta da skill em `skills/<nome-kebab-case>/`
4. Commit: `git commit -m "feat: adicionar skill <nome>"`
5. Push: `git push origin feat/nova-skill`
6. Abra um Pull Request

---

## Licença

MIT — veja o arquivo [LICENSE](LICENSE).
