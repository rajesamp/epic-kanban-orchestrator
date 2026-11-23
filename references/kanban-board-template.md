# Kanban Board State — Template & Schema

State is persisted in **two** files under `workspace/kanban/` and also rendered in-response each turn.

- `board.json` — machine-readable source of truth. Read at start of an execution turn, mutate, write back.
- `board.md` — human-readable mirror, regenerated from `board.json`.

## board.json Schema

```json
{
  "epic": {
    "title": "string",
    "goal": "string",
    "success_criteria": ["string"],
    "status": "in_progress | done",
    "domain_choice": "string"
  },
  "wip_limit": 2,
  "phases": [
    {
      "id": "P1",
      "name": "string",
      "objective": "string",
      "stories": [
        {
          "id": "US-1.1",
          "title": "string",
          "as_a": "role",
          "i_want": "capability",
          "so_that": "benefit",
          "acceptance_criteria": [
            { "id": "AC-1", "text": "string", "met": false }
          ],
          "subtasks": [
            {
              "id": "ST-1.1.1",
              "task": "string",
              "dod": "measurable outcome",
              "column": "Backlog | Ready | In Progress | Review | Done"
            }
          ],
          "status": "Backlog | Ready | In Progress | Review | Done"
        }
      ]
    }
  ]
}
```

## Invariants (enforce on every write)

- Count of sub-tasks in `In Progress` across the whole board <= `wip_limit` (default 2).
- A story's `status` becomes `Done` only when every sub-task `column == "Done"`.
- The epic `status` becomes `done` only when every story `status == "Done"`.
- A sub-task may move to `Done` only after passing through `Review` (user-approved).

## In-Response Render (board.md style)

Render this each execution turn so the user sees live status:

```
Epic: <title>  [<n_done>/<n_total> stories done]

| Backlog | Ready | In Progress (WIP a/2) | Review | Done |
|---------|-------|-----------------------|--------|------|
| ST-...  | ST-...| ST-...                | ST-... | ST-..|

Active story: US-1.1 — <title>  (<x>/<y> sub-tasks done)
Next pull: ST-1.1.x
```

## Minimal Bootstrap

On first epic, create `workspace/kanban/` and write both files. All new sub-tasks start in `Backlog`; move into `Ready` the ones whose dependencies are satisfied for the current phase.
