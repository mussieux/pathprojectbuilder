# PATH format reference (read when constructing manifests)

This reference reflects PATH's current `path-story` v1.0 / v1.1 / v1.2 conventions. The checked-in schemas/validator in the user's repository supersede it if they differ.

## Portable Player runtime shape

```json
{
  "format": "path-story",
  "version": "1.2.0",
  "id": "sample",
  "title": "Sample project",
  "entry": "start",
  "navigation": { "map": "enabled", "reveal": "adjacent" },
  "media": {
    "openingClip": { "playbackId": "REPLACE_WITH_PUBLIC_MUX_PLAYBACK_ID" },
    "branchClip": { "playbackId": "REPLACE_WITH_ANOTHER_PUBLIC_MUX_PLAYBACK_ID" }
  },
  "variables": { "score": { "type": "number", "initial": 0 } },
  "nodes": {
    "start": { "type": "start", "to": "opening" },
    "opening": { "type": "video", "title": "Opening", "media": "openingClip", "in": 0, "out": 12, "next": "choiceOne" },
    "choiceOne": {
      "type": "decision", "prompt": "What next?", "options": [
        { "id": "optionA", "label": "First route", "to": "firstEnd", "effects": [ { "add": { "variable": "score", "value": 2 } } ] },
        { "id": "optionB", "label": "Second route", "to": "branchVideo" }
      ]
    },
    "branchVideo": { "type": "video", "title": "Second route", "media": "branchClip", "in": 0, "out": 8, "next": "secondEnd" },
    "firstEnd": { "type": "ending", "title": "First ending" },
    "secondEnd": { "type": "ending", "title": "Second ending" }
  }
}
```

**Example only:** the playback IDs above are placeholders and the file is **not playable** until the user replaces them with their actual public IDs. Durations shown are also illustrative, not measured.

## Nodes and effects

- Start: `{ "type":"start", "to":"nodeId" }`
- Video: `{ "type":"video", "title":"...", "media":"mediaKey", "in":0, "out":10, "next":"nodeId" }`
- Decision: `{ "type":"decision", "prompt":"...", "options":[{"id":"choiceA","label":"...","to":"nodeId"}] }` plus optional description/when/unavailable/effects/playback.
- Ending: `{ "type":"ending", "title":"..." }`; the Ending node itself **does not play a video**. If the creator has an ending clip, place a Video node before the Ending.
- Router: `{ "type":"router", "rules":[{"id":"rule1","when":{"compare":{"variable":"score","op":"gte","value":5}},"to":"high"}], "fallback":"ordinary" }`. First matching rule wins. Fallback is mandatory. Router is not shown to the viewer as a question.
- Choice effects: `{"set":{"variable":"flag","value":true}}`, `{"add":{"variable":"score","value":-1}}`, `{"discover":"nodeId"}`. Effects apply in list order only when chosen. `add` requires a declared numeric variable.
- Conditions: `{"variable":"flag","equals":true}`, `{"compare":{"variable":"score","op":"lt","value":3}}`, `{"selected":{"decision":"decisionId","option":"optionId"}}`, `{"visited":"videoOrEndingId"}`, `{"discovered":"nodeId"}`, `{"all":[...]}`, `{"any":[...]}`, `{"not":{...}}`. Avoid empty `all`/`any`.
- Unavailable choice modes: `hidden`, `locked`, or `mystery`.

## Decision playback (v1.1+)

Held untimed decisions omit `playback`. An optional hold/countdown example:

```json
{
  "playback": {
    "while": "hold",
    "countdown": { "seconds": 10, "timeout": { "action": "choose", "option": "choiceA" } }
  }
}
```

`timeout` is `{"action":"wait"}`, `{"action":"choose","option":"choiceId"}`, or `{"action":"go","to":"nodeId"}`. At least 3 seconds. `while: "loop"` requires `"loop":{"media":"mediaKey","in":0,"out":5}` within `playback`; this is background footage while the decision is displayed. Don't add other fields/aliases.

## Versioning and file formats

- v1.0: ordinary videos, decisions, conditions (equality), endings, and discoveries.
- v1.1: adds decision playback loop/hold/countdown.
- v1.2: adds numeric `add`, `compare`, and Router nodes, retaining older features.
- Runtime `.json` has `format: "path-story"`. The editable `.pathproj` has a different envelope, `format: "path-studio-project"` version `1.0.0`, with nested story, assets and editor/layout state. Do not treat them as interchangeable or rename an extension to convert.
- Source times use seconds from the original video and intervals `[in, out)`.
- Player manifest media entries contain **public Mux playback IDs** (not asset IDs, stream URLs, credentials, or local paths).

## Limitations

PATH v1 prohibits graph cycles, though a decision can loop background footage. A project cannot be considered runtime-ready until every reachable branch has a valid destination and eventually ends. Source validation does not prove the remote Mux video is available. For live verification, open in the actual Player.
