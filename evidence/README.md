# #485 evidence — data-flow vertical-extent repair contract

Fixtures and receipts for the data-flow half of #468 (rebased revision `3256900` on `dev` `603bbb41`).

## Cases

| case | spec | what it shows |
| --- | --- | --- |
| `case-a-before` | 940×520 canvas, `audit` node on row 3 | the diagnostic: message, measured evidence, ordered repairs (`raise meta.viewBox[1] to at least 602 and at most 606`) |
| `case-a-raised` | same spec, `meta.viewBox[1]` raised to 602 (inside the offered range) | repaired by the raise: validate / deliver / browser-check / visual-check all pass |
| `case-a-compacted` | same spec, `audit` `yOffset: -180` | repaired by moving the node: same chain all pass |
| `case-b-before` | same canvas, `audit` height 156 | the required raise (700) is above the 606 ceiling: the raise is not offered, the message carries the predicted 1128px page |
| `case-b-raised` | case-b with `meta.viewBox[1] = 700` anyway | the trap the gate avoids: `validate` passes, `browser-check` fails `viewer/viewport-overflow` at 1440×900 / 1600×1000 / 1920×1080 light and 1440×900 dark |

## Reproduction

From a checkout of the archify skill at this revision:

    node archify/bin/archify.mjs validate      dataflow evidence/case-a-before.dataflow.json --json
    node archify/bin/archify.mjs deliver       dataflow evidence/case-a-raised.dataflow.json
    node archify/bin/archify.mjs browser-check evidence/case-a-raised.html
    node archify/bin/archify.mjs visual-check  evidence/case-a-raised.html

`validate` prints the JSON receipt to stdout. `deliver` writes `<spec>.html` plus the `.delivery.json` receipt; `browser-check` and `visual-check` write their sidecars next to the artifact (they refuse to overwrite an existing sidecar — use `--out-dir` to re-run).

## Screenshots

- `dataflow-repair-raised-1440x900.png` / `.dark.png` — `case-a-raised` at 1440×900.
- `dataflow-repair-compacted-1440x900.png` / `.dark.png` — `case-a-compacted` at 1440×900.

## Lifecycle half (why it is unchanged)

On the PR's base (`dev`, `1dff447`), the lifecycle renderer derives its canvas from rendered geometry: `meta.viewBox` is not even a schema property any more, so a state cannot be pinned past a too-short canvas and the pre-v3 `585 → 658` repair has no surface.

- `lifecycle-v3-derived.*` — the v3 example (10 states) validates with zero diagnostics; the delivered canvas is derived (`viewBox="0 0 1152 500"`, not authored); `browser-check` passes at 1440×900 / 1600×1000 / 1920×1080 / 2048×1320 light.
- `lifecycle-pinned-viewbox.*` — the same spec with `meta.viewBox: [900, 585]` added: `validate` rejects it with `schema/additionalProperties`.

