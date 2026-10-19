---
name: batiq
description: Drive the BATIQ node compositor from the command line. Use when the user wants to build, inspect, edit or render a BATIQ project (.batiq), wire nodes (Read, Keyer, Merge, 3D Set, Tracker, Write...), set or animate parameters, run Variants or Functions, migrate old projects, or automate BATIQ from scripts, Python or a render farm.
---

# BATIQ

BATIQ is a free, native node compositor (macOS, Windows, Linux): float
scene-linear EXR, real OpenColorIO, a GPU 3D Set node, tracking, keying and an
Edit node. Everything the desktop app edits, the `batiq-headless` command line
can edit too, through the same code.

BATIQ changes every week. This file teaches the shape of the tool and how to
discover the rest; **the live docs and the binary's own answers are the truth,
not this file.** When they disagree with anything below, trust them.

## 1. Read the current docs first

Before a non-trivial task, fetch the guide for the area you need. Plain-text
copies are made for agents:

| Need | Plain text | Same guide on the web |
| --- | --- | --- |
| Index of everything, current feature list | https://batiq.app/llms.txt | https://batiq.app/ |
| Headless CLI: methods, `edit_graph` actions, render | https://batiq.app/guides/cli.md | https://batiq.app/docs#guide-cli |
| Python (embedded `batiq` and parameter names) | https://batiq.app/guides/python.md | https://batiq.app/docs#guide-python |
| Desktop guide: nodes and what they do | https://batiq.app/guides/desktop.md | https://batiq.app/docs#guide-desktop |
| Reusable Functions | https://batiq.app/guides/functions.md | https://batiq.app/docs#guide-functions |
| Remote Links | https://batiq.app/guides/links.md | https://batiq.app/docs#guide-links |
| NodeGen (local model writes a node's kernel) | https://batiq.app/guides/nodegen.md | https://batiq.app/docs#guide-nodegen |
| Playback, caches, export | https://batiq.app/guides/playback.md | https://batiq.app/docs#guide-playback |
| Colour / OCIO / ACES | https://batiq.app/guides/color.md | https://batiq.app/docs#guide-color |

If a plain-text URL fails, use the web page. Tutorials (for example the 3D Set
one) are linked from `llms.txt`.

## 2. Find the executable

`batiq-headless` ships beside the desktop app in every release
(https://github.com/georgeantonopoulos/batiq-releases/releases):

- **macOS:** `/Applications/BATIQ.app/Contents/MacOS/batiq-headless`
  (or `~/Applications/BATIQ.app/...`)
- **Windows:** `batiq-headless.exe` in the unzipped BATIQ folder, next to `batiq.exe`
- **Linux:** `batiq-headless` in the unzipped folder, next to `batiq`
- Or whatever `BATIQ_HEADLESS_EXECUTABLE` / `PATH` points to.

If it is not installed, ask the user before downloading a release.

## 3. How a call works

It speaks JSON-RPC 2.0, one JSON object per line. Write every request to a
file, pipe it in, close stdin; read responses from stdout. Keep stderr apart
(diagnostics only).

```sh
batiq-headless < requests.jsonl > responses.jsonl 2> diagnostics.log
```

Always send `initialize` first. Its reply tells you the BATIQ version, the
project format and **which methods this build has** (`capabilities`). Check
it rather than assuming a method exists.

```json
{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocol_version":{"major":1,"minor":0}}}
```

Every request needs its own `id`. Use absolute paths.

### Main methods (confirm with `capabilities`)

- `edit_graph`: an ordered batch of `actions` on one project, saved
  atomically. Needs `project` plus `destination` or `in_place: true`, or
  `new: true` with `destination`. A batch of reads only needs no destination.
- `render`: `project`, `output`, `frames: {start, end, step}`, optional
  `write_node`, `variants`, `variant_options`, `variant_presets`.
- `validate_project`, `migrate_project`: check or upgrade an old file.
- `edit_function`, `edit_variants`, `edit_expression`, `node.*`
  (`node.input_slots`, `node.input_offers`, numeric inputs),
  `set.export_camera`.

### Discovering what you can do

The tool tells you what it accepts. Use these instead of guessing:

- **All `edit_graph` actions:** send an unknown action such as
  `{"action":"help"}` in a read-only batch. The error lists every action name.
- **A node's parameters:** `{"action":"params","node":"Blur1"}`. A wrong
  parameter name is refused with the full list.
- **What a node's inputs take / what a wire can land on:**
  `node.input_slots` and `node.input_offers` (for a 3D Set: cards, sky,
  meshes, cameras).
- **What can be animated:** `{"action":"animatable","node":"..."}`.
- **The graph:** `{"action":"list"}`.
- **Node kinds:** create an unknown kind; newer builds list every kind in
  the error (older ones: see the desktop guide). Kinds accept the label or stored name,
  any case (`"Batiq"` or `"Coverage"`, `"OCIO ColorSpace"` or `"ColorSpace"`).

Inside a batch, refer to a node by id, by name, or `"$N"` (the Nth node
created earlier in the same batch, from `$0`).

## 4. A tested example

Build a comp and render it (PNG to keep it small; EXR, TIFF, DPX, JPEG also work):

```json
{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocol_version":{"major":1,"minor":0}}}
{"jsonrpc":"2.0","id":2,"method":"edit_graph","params":{"new":true,"destination":"/work/sh010/comp.batiq","actions":[
  {"action":"project","width":1920,"height":1080,"fps":24,"frame_range":[1001,1010]},
  {"action":"create","kind":"Read","name":"Plate"},
  {"action":"set","node":"Plate","name":"path","value":"/plates/sh010.####.exr"},
  {"action":"create","kind":"Blur","name":"Soft"},
  {"action":"connect","node":"Soft","source":"Plate"},
  {"action":"set","node":"Soft","name":"blur_size","value":4},
  {"action":"create","kind":"Write","name":"Out"},
  {"action":"connect","node":"Out","source":"Soft"},
  {"action":"set","node":"Out","name":"write_format","value":"png"}
]}}
{"jsonrpc":"2.0","id":3,"method":"render","params":{"project":"/work/sh010/comp.batiq","output":"/work/sh010/renders/sh010.####.png","frames":{"start":1001,"end":1010,"step":1}}}
```

Point the Read at frames that exist: a path with no files is stored as
typed, and the render then fails on the missing plate. (Requests must each
be on one line in the real file; they are wrapped here for reading.) More recipes: [references/recipes.md](references/recipes.md).

## 5. Rules that save you

- **Give `render` a full file pattern** (`/dir/name.####.exr`). Newer
  builds also take a folder (`/dir/`) and name the frames after the Write;
  older ones write `dir.####.exr` beside it, so the pattern is safe everywhere.
- **A render can fail with a normal `result`.** Read the `render_event`
  stream: success is a terminal `finished` event. Exit code 0 proves nothing.
- **Batches are all-or-nothing.** On error nothing is written and the message
  names the failing action's index. Fix that action and resend the batch.
- **Values are validated like the app.** Out-of-range or misspelled input is
  refused, never guessed. Read the error, it usually lists the valid options.
- **Old project?** `validate_project` first; `migrate_project` to a new file
  before editing.
- **The 3D Set node needs a GPU.** `batiq-headless` uses one off-screen; with
  no GPU it errors by name rather than rendering black.
- **Commands that need the live window** (selection, menus, open/save dialogs)
  are refused headless. Use the request's `project` / `destination` / `new`.
- **Don't overwrite the user's project** unless asked: write to a new
  `destination`, or use `in_place: true` only when told to.
- **Don't parse stdout as text.** It is JSON lines only; stdin is capped at 2 MiB.

## 6. Python

- Inside the running app (Script Editor, startup modules), `import batiq`
  drives the open project: `batiq.nodes.create("Blur")`, `node["blur_size"] = 4`.
  Parameter and action names are the same as the CLI. See the Python guide.
- From outside the app, call `batiq-headless` with `subprocess` as above.

## 7. Keep this skill current

This skill is a git checkout. To update it: `git -C <this folder> pull`.
If BATIQ behaves differently from these notes, follow BATIQ and the docs.
