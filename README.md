# PATH Project Builder

A reusable AI skill for preparing PATH Player branching-video projects by answering conversational questions. Supply public Mux playback IDs, describe where sections start/end, and say how choices connect; the skill produces a Player runtime `.json` without opening Studio.

## Installation

Download and extract the repository ZIP before following either platform's instructions.

### Codex

Copy the **entire** `path-project-builder` folder (including `references/`) into your repo's `.agents/skills/` folder, or into your user skills folder at `~/.agents/skills/`. Start Codex in that repo and invoke `$path-project-builder`.

### Claude Code

Copy the **entire** folder into your repo's `.claude/skills/` folder (or `~/.claude/skills/` for personal use). Invoke `/path-project-builder`.

## Example request

> Use PATH Project Builder. I'll give you my public Mux playback IDs. Ask me how the sections connect, include a decision with two options, and export a PATH Player manifest. Keep the story private and don't ask for Mux credentials.

## What this does and does not do

- **Does:** author a `path-story` JSON manifest compatible with PATH Player when the story is complete and validated. Supports scoring, Routers, and decisions where needed.
- **Optional:** create an editable `.pathproj` when the PATH Studio MCP is locally available.
- **Doesn't:** upload video, host files, call the Mux API, store credentials, deploy a website, or provide a graphical timeline.

A file built from incomplete inputs is a draft until its actual media and graph pass PATH validation. Full feature details are in `SKILL.md` and `references/path-format.md`.

## Get the skill

Download this public repository using **Code → Download ZIP**, extract it, and rename the extracted directory to `path-project-builder`. Copy that directory—including `SKILL.md` and `references/path-format.md`—into the Codex or Claude Code skills location above. The entire repository is the skill; no PATHPlayer repository access is required.

Repository: https://github.com/mussieux/pathprojectbuilder

Direct download: https://github.com/mussieux/pathprojectbuilder/archive/refs/heads/main.zip

To use it immediately, provide the opening video's **public Mux playback ID**, its start/end timestamps, and the intended decisions or connections. The skill will ask follow-up questions only when needed.

## After generating a project

In PATH Player select **Open Player manifest (.json)** to load the generated runtime file. If you requested an editable `.pathproj`, import it into PATH Studio. The two formats are different; do not rename extensions to convert between them.

**Never enter Mux Token IDs or secrets into a skill conversation.** Only public playback IDs are needed.