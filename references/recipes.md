# BATIQ recipes

Each block is one `requests.jsonl` file. Start every file with:

```json
{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocol_version":{"major":1,"minor":0}}}
```

Requests are wrapped here for reading; in the real file each request is one
line. Paths are examples; use absolute paths. These were run against BATIQ
0.2.38. If a newer or older build refuses a field, read the error and the
CLI guide (https://batiq.app/guides/cli.md).

## Look before you touch

Read-only batches need no `destination` and write nothing.

```json
{"jsonrpc":"2.0","id":2,"method":"validate_project","params":{"project":"/shots/sh010/comp.batiq"}}
{"jsonrpc":"2.0","id":3,"method":"edit_graph","params":{"project":"/shots/sh010/comp.batiq","actions":[
  {"action":"list"},
  {"action":"params","node":"Grade1"},
  {"action":"animatable","node":"Grade1"},
  {"action":"get_timeline"}
]}}
{"jsonrpc":"2.0","id":4,"method":"node.input_slots","params":{"project":"/shots/sh010/comp.batiq","node_id":4}}
```

`validate_project` says whether the file needs migrating. `node.input_slots`
names each input (a Merge has `A` = 0, `B` = 1, `mask` = 2).

To see every action `edit_graph` accepts in this build, send one it does not
know; the error lists them all:

```json
{"jsonrpc":"2.0","id":5,"method":"edit_graph","params":{"new":true,"actions":[{"action":"help"}]}}
```

## Key a plate over a background

```json
{"jsonrpc":"2.0","id":2,"method":"edit_graph","params":{"new":true,"destination":"/shots/sh020/comp.batiq","actions":[
  {"action":"project","width":3840,"height":2160,"fps":24,"frame_range":[1001,1100]},
  {"action":"create","kind":"Read","name":"Plate"},
  {"action":"set","node":"Plate","name":"path","value":"/plates/sh020_fg.####.exr"},
  {"action":"create","kind":"Keyer","name":"Key"},
  {"action":"connect","node":"Key","source":"Plate"},
  {"action":"create","kind":"Read","name":"Bg"},
  {"action":"set","node":"Bg","name":"path","value":"/plates/sh020_bg.####.exr"},
  {"action":"create","kind":"Merge","name":"Over"},
  {"action":"connect","node":"Over","input":0,"source":"Key"},
  {"action":"connect","node":"Over","input":1,"source":"Bg"},
  {"action":"create","kind":"Write","name":"Out"},
  {"action":"connect","node":"Out","source":"Over"},
  {"action":"set","node":"Out","name":"write_format","value":"exr"},
  {"action":"params","node":"Key"}
]}}
```

The last action returns every Keyer setting (screen mode, despill, core
matte...) so you can tune it in a second batch with `set`, using dotted
paths into nested values, for example `screen_key.despill`.

## Animate and drive values

```json
{"jsonrpc":"2.0","id":2,"method":"edit_graph","params":{"project":"/shots/sh020/comp.batiq","in_place":true,"actions":[
  {"action":"create","kind":"Transform","name":"Move"},
  {"action":"key","node":"Move","property":"TranslateX","frame":1001,"value":0},
  {"action":"key","node":"Move","property":"TranslateX","frame":1100,"value":240},
  {"action":"set_node_expression","node_id":"Move","property":"TranslateY","formula":"frame*2"}
]}}
```

Property names come from `animatable`. Newer builds take an
`interpolation` on `key` (`Step`, `Linear`, `EaseIn`, `EaseOut`, `Smooth`,
`Bezier`); expression syntax is in the docs.

## Render

```json
{"jsonrpc":"2.0","id":2,"method":"render","params":{"project":"/shots/sh020/comp.batiq","output":"/shots/sh020/renders/sh020.####.exr","frames":{"start":1001,"end":1100,"step":1}}}
```

- Create the output folder first, and give a full file pattern.
- With more than one Write node, pass `"write_node": <id>`.
- Success is the terminal `render_event` with `"event":"finished"`; a
  `failed` event can still arrive with a normal `result`.
- Variants: `variants` picks one option per group, `variant_options` renders
  combinations, `variant_presets` renders saved presets.

## Upgrade an old project

```json
{"jsonrpc":"2.0","id":2,"method":"migrate_project","params":{"project":"/old/comp.batiq","destination":"/old/comp_current.batiq"}}
```

Edit the migrated copy, not the original.

## Track, solve and bake a camera

```json
{"jsonrpc":"2.0","id":2,"method":"edit_graph","params":{"project":"/shots/sh030/comp.batiq","in_place":true,"actions":[
  {"action":"add_track","node_id":4,"frame":1001,"center":[812,440]},
  {"action":"solve_tracks","node_id":4,"from":1001,"to":1100}
]}}
```

A Camera Tracker solves with `analyze_node` (`frame`, `last`) and
`bake_camera` keys the result into a 3D Set's camera; see the CLI guide.

## 3D Set, Functions, Remote Links, NodeGen

These have richer data than fits here. Read the guide before writing them:

- 3D Set (cards, USD/Alembic, lights, cameras, passes, `connect_offer`):
  https://batiq.app/guides/python.md and https://batiq.app/tutorial-3d-set
- Functions: https://batiq.app/guides/functions.md
- Remote Links: https://batiq.app/guides/links.md
- NodeGen (`generate_kernel`): https://batiq.app/guides/nodegen.md
