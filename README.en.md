# iOS Development Skills

A collection of structured, reusable skills (agent-ready instructions) for iOS development with UIKit and SwiftUI.

Each skill in this repository is a self-contained package with step-by-step procedures, code templates, and Apple documentation references. Skills are project-agnostic and work across multiple IDEs / AI agents.

---

## Available skills

| Skill | Description |
|-------|-------------|
| [uikit-scene-lifecycle-transition](skills/uikit-scene-lifecycle-transition/) | Migrate UIKit apps from the AppDelegate-based lifecycle to the scene-based lifecycle (UISceneDelegate / UIWindowSceneDelegate). Required starting in iOS 27. |
| [iphone-duo-expert](skills/iphone-duo-expert/) | Adapt iOS apps (SwiftUI + UIKit) for the iPhone Duo foldable. Mandatory audit, NavigationSplitView migration, iOS 27.1 foldable APIs (ArrangementView, Reserved Regions, onHingeChange, CameraCaptureAccessory), asymmetric safe areas, vertical sidebars, 8-check validation checklist and P0/P1/P2 audit response format. Based on the 6 official Apple Tech Talks 111461–111466 and WWDC26 Session 269. |
| [swiftui-expert](skills/swiftui-expert/) | Technical authority for SwiftUI (iOS + macOS): data flow with `@State`/`@Observable`, view composition, performance and diffing, lists and `ForEach`, environment, layout, navigation, animations, accessibility, Swift Charts, localization, Liquid Glass, and API migration. Ships topic-by-topic references plus scripts for recording and analyzing Instruments traces. |

---

## Installation

Pick your IDE / AI agent below and run the corresponding commands in your terminal.

### 1. TRAE

TRAE (International edition) loads skills from the `~/.trae/skills/` directory.

```bash
# Clone the repo (if you haven't already) or update it
cd ~ && git clone https://github.com/salllgado/ios-development-skills.git 2>/dev/null || true

# Copy the desired skill into TRAE's global skills folder
# Example for the uikit-scene-lifecycle-transition skill:
cp -R ~/ios-development-skills/skills/uikit-scene-lifecycle-transition \
  ~/.trae/skills/uikit-scene-lifecycle-transition
```

Restart TRAE (or reload the window) so the skill is loaded.

#### TRAE CN Edition

If you use the Chinese edition of TRAE, replace `.trae` with `.trae-cn` in the path:

```bash
cp -R ~/ios-development-skills/skills/uikit-scene-lifecycle-transition \
  ~/.trae-cn/skills/uikit-scene-lifecycle-transition
```

#### Per-project skill (local scope)

To enable the skill in a single iOS repository only (so anyone cloning the repo automatically gets it):

```bash
cd /path/to/your/ios-project
mkdir -p .trae/skills
cp -R ~/ios-development-skills/skills/uikit-scene-lifecycle-transition \
  .trae/skills/uikit-scene-lifecycle-transition
```

---

### 2. Cursor

Cursor supports Custom Instructions and context folders. To use these skills:

```bash
# Clone the repo
cd ~ && git clone https://github.com/salllgado/ios-development-skills.git 2>/dev/null || true

# (Optional) Add the skills folder as a workspace folder in Cursor
# Open Cursor, use Cmd+Shift+P > "Add Folder to Workspace"
# and select ~/ios-development-skills
```

**Command-line alternative (symlink for easy updates):**

```bash
# (macOS) Create a symlink inside your iOS project
cd /path/to/your/ios-project
ln -sf ~/ios-development-skills/skills .ios-skills
```

Now when chatting with Cursor, reference a `SKILL.md` file as context. Example prompt:

> Follow the procedure in `.ios-skills/uikit-scene-lifecycle-transition/SKILL.md` to migrate my AppDelegate to SceneDelegate. First check whether I already have UIApplicationSceneManifest in Info.plist.

---

### 3. Claude Code (CLI)

Claude Code CLI loads context via `--context` or local files.

```bash
# Clone the repo
cd ~ && git clone https://github.com/salllgado/ios-development-skills.git 2>/dev/null || true
```

Option A — Pass the skill as context on each call:

```bash
claude --context ~/ios-development-skills/skills/uikit-scene-lifecycle-transition/SKILL.md
```

Option B — Install globally via symlink into Claude Code's skills folder (if your setup uses one):

```bash
# Adjust the destination path to match your Claude Code installation
# Example (adjust as needed):
mkdir -p ~/.config/claude-code/skills
ln -sf ~/ios-development-skills/skills/uikit-scene-lifecycle-transition \
  ~/.config/claude-code/skills/uikit-scene-lifecycle-transition
```

Option C — Inside your iOS project, drop a hint file:

```bash
cd /path/to/your/ios-project
echo 'iOS skills context available at ~/ios-development-skills/skills/' > .claude
```

---

### 4. Kiro

Kiro loads skills from a user config directory or via inline context.

```bash
# Clone the repo
cd ~ && git clone https://github.com/salllgado/ios-development-skills.git 2>/dev/null || true

# Copy the skills into Kiro's skills directory
# Adjust the path below for your Kiro installation (macOS):
KIRO_SKILLS_DIR="$HOME/Library/Application Support/Kiro/skills"
mkdir -p "$KIRO_SKILLS_DIR"
cp -R ~/ios-development-skills/skills/uikit-scene-lifecycle-transition \
  "$KIRO_SKILLS_DIR/uikit-scene-lifecycle-transition"
```

If your Kiro installation uses a different directory, replace `$KIRO_SKILLS_DIR` with the correct path. Alternatively, just open the `~/ios-development-skills/` folder as a project in Kiro, or attach the desired `SKILL.md` as context to a conversation.

---

## Updating skills

```bash
cd ~/ios-development-skills && git pull --ff-only
```

Then re-copy or re-symlink the updated skill into your IDE's directory using the commands above.

---

## Skill structure

Each folder under `skills/` follows the standard layout:

```
skills/<skill-name>/
├── SKILL.md           ← Procedure, rules, and triggers
├── assets/            ← Code templates, XML snippets
├── references/        ← Reference docs (e.g. Apple articles in Markdown)
└── scripts/           ← (optional) Helper scripts
```

---

## Contributing

1. Fork the repository
2. Create a branch: `git checkout -b feat/new-skill`
3. Add your skill folder under `skills/<kebab-case-name>/`
4. Commit: `git commit -m "feat: add <name> skill"`
5. Push: `git push origin feat/new-skill`
6. Open a Pull Request

---

## License

MIT — see the [LICENSE](LICENSE) file.
