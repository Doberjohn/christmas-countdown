# Christmas Countdown

Three interactive Christmas countdown scenes, each a single static HTML page:

- `scenes/frosted-window/` (finished): the countdown written in a frosted window, a
  soft-focus room behind it, drips, gusts of wind and quiet music.
- `scenes/paper-village/` (to polish): a paper-cut village where an Advent window lights
  up each night of December.
- `scenes/bauble-garland/` (to polish): one glass bauble for each sleep left.

## Run locally

No build step. Serve the folder and open a scene:

```
npx serve .
```

Then visit `http://localhost:3000/`. The home page is the frosted window (a rewrite in
`serve.json` locally and `vercel.json` on Vercel); the other scenes are at
`/scenes/paper-village/` and `/scenes/bauble-garland/`. Add `#christmas-day` to the
frosted window's address to preview Christmas Day.

## Working on it with Claude Code

`CLAUDE.md` has the full context: status, decisions, how each page works and the next
steps. A good first message:

> Read CLAUDE.md. Let's polish the paper village scene next, to the same standard as
> the frosted window. Start by serving it locally and telling me what you'd improve.
