# Herdr Marks as Unread

![Mark as unread in the Herdr TUI (prefix, then M) and in the Herdr app (hover an agent, click the unread icon)](assets/mark-as-unread.png)

Flag an agent you want to come back to. It gets a red `unread` badge in the
sidebar, and the badge clears by itself when you return to it. The mark is
display-only: it never changes the agent's status, name, or notifications.

Works in both Herdrs, with a separate implementation for each:

| You run | Install from |
|---|---|
| The Herdr TUI (`herdr` 0.9.3+) | the repo root, with `herdr plugin install` |
| The Herdr desktop app | the `herdr-app/` folder, as a Herdr mod |

Marks set in one are not shown by the other.

## Herdr TUI

From a shell inside Herdr:

```sh
herdr plugin install sachin-philip/herdr-marks-as-unread
herdr plugin action invoke local.herdr-marks.setup
```

Setup adds the badge to your sidebar layout, binds `prefix+m`, and reloads the
config. Rerun it after updating. Needs Node.js 18+ and Python 3.11+.

Press **Control+B**, release, then **M** to mark the focused agent. Press again
to unmark. Switch away; when you come back, the mark clears.

Details, manual configuration, development, and uninstall: [docs/tui.md](docs/tui.md).

## Herdr desktop app

Open Claude Code (or another agent) in a Herdr terminal and ask:

```
Add the Herdr mod from https://github.com/sachin-philip/herdr-marks-as-unread,
folder herdr-app. Follow its RECIPE.md.
```

Or by hand: copy `herdr-app/mods/agents-marks/` into `~/.config/herdr-app/mods/`,
clone your layout with `herdr-app do layout.clone '{"name":"main","as":"herdr-marks"}'`,
change its `"mod": "agents"` to `"mod": "agents-marks"`, and add
`"layout:herdr-marks": { "cmd-shift-m": "agents-marks.toggle" }` to `keys.json`.

Press **Cmd+Shift+M** to mark the focused agent, or hover its row and click the
square icon. Click the badge, or focus the agent, to clear it. From a shell,
`hc label t4 herdr.mark=unread` sets a mark and `hc label t4 herdr.mark=` clears it.

The mod is a clone of Herdr's Agents sidebar and does not follow Herdr's later
changes to it. Everything else: [herdr-app/RECIPE.md](herdr-app/RECIPE.md).

## License

MIT. See [LICENSE](LICENSE).
