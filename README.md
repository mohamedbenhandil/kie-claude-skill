# kie-claude-skill

Un skill Claude Code pour générer des images avec [KIE.ai](https://kie.ai) (GPT Image 2.5, Nano Banana 2.1) directement depuis Claude, et les voir s'afficher dans la conversation.
A Claude Code skill to generate images with KIE.ai and see them inline.

## Installation (FR)
1. Créez une clé API sur https://kie.ai.
2. Copiez le dossier `skills/kie-image` dans `~/.claude/skills/` (Windows : `%USERPROFILE%\.claude\skills\`).
3. Définissez votre clé (ne la partagez jamais) :
   - Windows (PowerShell) : `setx KIE_API_KEY "votre-clé"` puis rouvrez Claude.
   - macOS/Linux : `export KIE_API_KEY="votre-clé"` dans votre profil shell.
4. Demandez à Claude : « génère une image de ... avec nano banana 2.1, 1K, 16:9 ».

## Install (EN)
1. Get an API key at https://kie.ai.
2. Copy `skills/kie-image` into `~/.claude/skills/`.
3. Set `KIE_API_KEY` as an environment variable (never commit it).
4. Ask Claude: "generate a ... image with nano banana 2.1, 1K, 16:9".

## Ce que le skill sait faire
- Optimiser votre prompt puis générer avec GPT Image 2.5 Flare ou Nano Banana 2.1.
- Utiliser une image de référence (upload automatique).
- Afficher le résultat en grand, générer plusieurs variantes en parallèle.
- Éviter les pièges connus (image de référence renvoyée sans changement, texte mal orthographié).

Non affilié à KIE.ai ni à Anthropic. / Not affiliated with KIE.ai or Anthropic.

---
Auteur / Author : **MOHAMED BENHANDIL** — https://github.com/mohamedbenhandil
