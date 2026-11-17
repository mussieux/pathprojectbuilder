---
name: path-project-builder
description: Build or revise PATH Player interactive branching-video projects through a guided conversation using public Mux playback IDs, scene in/out times, choices, connections, cumulative scores, and automatic routing. Use when someone wants a playable PATH .json manifest without manually using PATH Studio; optionally create editable .pathproj with the PATH Studio MCP when available. Never ask for Mux credentials.
---

# PATH Project Builder

Create **real PATH-compatible branching projects**, not just a text diagram. Use a brief conversational interview to obtain the media and route structure, then write a runtime `path-story` JSON file that the user can open directly in PATH Player. When PATH Studio MCP is connected, optionally create an editable `.pathproj` as well. No PATH Studio browser session is required for ordinary runtime JSON generation.

## Before starting

- Read `references/path-format.md` when you need the exact manifest shapes; use the current repo's `schemas/story-manifest-v1*.schema.json`, `src/story/types.ts`, `src/story/validate.ts`, and `docs/STORY_MANIFEST_V1.md` as authoritative if available. If the local repo contradicts the reference, follow the local checked-in validator and report the difference.
- The user's project/story content is private input. Never substitute an example story, invented clip, invented playback ID, or invented duration for missing facts.
- **Do not ask for Mux Token ID, Token Secret, API key, account access, or video upload permission.** Only public **playback IDs** (not asset IDs) are required for normal PATH playback. If the user provides a playback URL, extract the likely ID only with explicit confirmation; don't silently infer IDs from arbitrary strings.
- Do not claim the public Mux video exists, is licensed, is accessible, or has a measured duration merely because its ID matches the syntax.
- Keep all branching logic story-agnostic. Never assume a film has chapters, a fixed number of choices, or particular endings.

## Guided authoring interview

**Ask the minimum questions required, one short cluster at a time.** Incorporate answers already supplied. For short projects, do not use a giant questionnaire. Start with:

> What's the project title, and what's the public Mux playback ID for the opening video? You can give me several IDs at once if you've already uploaded all your clips.

As the user supplies clips, create a working media inventory: stable media key, label, public playback ID, optionally known duration, and proposed playable ranges. Then ask:

1. **Sequence:** Which video starts first? Does it play its entire length or a segment (`in`/`out` in source-video seconds)? What plays next?
2. **Choices:** At which point should playback pause and ask a question? What is the exact prompt and each choice label? What clip or outcome follows each choice?
3. **Connections:** After each choice-specific clip, do routes rejoin a shared next clip, stay separate, or end? Ask only about unresolved connections; diagram the current draft as `opening → decision → {A,B} → ...`.
4. **Behavior, only if requested:** Should a decision hold or show looping footage? Is there a countdown and default/keep-open/direct destination? Are any choices hidden/locked/mystery until a condition is met?
5. **Advanced logic, only if useful:** Does a choice set a variable, add/subtract a numeric score, record a discovery, or depend on prior decisions? Are invisible automatic Router rules needed? Ask their priority and **mandatory fallback**.
6. **Display:** Should the Paths map be enabled, and should reveal be `adjacent` (safer default) or `all`? Ask only when it matters.

At every stage, summarize confirmed choices in a small table or compact route sketch. Distinguish **confirmed** from **missing**. If user says “just create it,” build the smallest valid project from known facts, and mark missing playable IDs/times plainly rather than inventing them.

## Default creation strategy

- For a plain clip: `start → video → ending`, Paths map disabled.
- For branching: `start → video → decision → choice destinations → ending(s)` with meaningful labels. Each choice must have a valid destination; branch clips can reconnect later. One media ID can be reused for multiple non-overlapping or overlapping `[in, out)` sections.
- Separate a *source video* from a *PATH section*: an asset is declared once in `media`, while multiple `video` nodes can reference it with different in/out values.
- Use a stable human-readable ID beginning with a letter for story, node, variable, and choice IDs; do not derive structure from titles once IDs are assigned.
- The story has exactly one Start node. Do not create graph cycles. All reachable playable routes must terminate in an Ending node; internal decision loop footage is **not** a graph cycle.
- For numeric score routing, use `version: "1.2.0"`, `{ "add": { "variable": "score", "value": 2 } }`, `{ "compare": { "variable": "score", "op": "gte", "value": 3 } }`, and a Router's ordered rules + fallback. Never model an invisible automatic router as a viewer-facing decision.
- For basic stories without v1.1/v1.2 features, use the earliest compatible manifest version. Decision loop/countdown requires v1.1 or newer. Score/compare/Router requires v1.2.
- Source in/out times must be finite, nonnegative, and `out > in`; `out` may not exceed a known actual duration. If duration is unknown, treat timing as unverified.
- Every published runtime `media` entry must have a real public Mux playback ID; don't embed local video files or Mux credentials.

## Create and validate outputs

1. Show a compact **proposed route plan** (media inventory + decision edges + automatic rules, if any). Ask for one approval to finalize if the user has not already authorized generation.
2. Generate valid JSON with exactly the `path-story` schema supported by the current repo. Preferred filename: `<story-id>.path.json`. Save in the user's requested workspace/export location, not an invented public upload location.
3. **Validate before claiming ready:** Parse the JSON. When the PATH repository is accessible, use its real `validateManifest` implementation or the existing validated Studio/MCP export workflow; inspect reported errors and fix only those. At minimum check that IDs/references resolve, ranges are legal, the graph is acyclic, conditions and effects reference declared typed variables, Routers have valid fallbacks, and all routes can reach endings. If PATH's validator was unavailable, say the manifest is **schema-unverified** and direct the user to open it in PATH Player for final validation.
4. When the PATH Studio MCP server is available and the user wants an editable project, use its `create_project`/`read_project`, `preview_graph_batch`, `apply_graph_batch`, `validate_project`, and `export_project` tools (follow their current signatures and overwrite-review requirements) to generate an editable `.pathproj`. An MCP `.pathproj` is **not** a runtime Player `.json`. Do not invent a live browser bridge.
5. Provide the generated file(s), a compact route summary, any unresolved issues, and exact next step: **PATH Player → Open Player manifest (.json)**. If `.pathproj` is generated: **PATH Studio → Open/import `.pathproj`**. Do not claim deployment, hosting, or automatic video uploads.

## Editing existing work

If user provides a `.json` runtime manifest or `.pathproj`, read the entire file before modifying it. Preserve stable IDs, version compatibility, asset references, existing variables, conditions, timed decisions, discoveries, and history semantics. For `.pathproj`, prefer PATH Studio MCP for edits; don't replace it with a runtime `.json` without consent. Preview any destructive/reconnecting changes and obtain confirmation where the MCP requires it. Save a **new named output** by default; never silently overwrite the user's only project copy.

## Keep it efficient

- Prefer one concise question over a multi-step form.
- Build sections incrementally and reuse the answers you've already been given.
- Do not run broad browser test suites, content audits, or repeated rebuilds solely to generate a manifest. Perform one real validation when available.
- A useful partial project is preferable to fabricated data, but don't label an unfinished graph playable.