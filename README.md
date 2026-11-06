# Herdr Marks as Unread

![Herdr sidebar with a red unread badge on an agent and the Control+B, then M shortcut](assets/mark-as-unread.png)

Keep track of agents you want to revisit. Add a red `unread` badge to an
agent's sidebar row, then have it clear automatically when you return to that
terminal.

The marker is display-only metadata. It does not change the agent's lifecycle
status, completion badge, name, title, or notifications.

This repo ships two implementations, one per Herdr:

| You run | Install from | Marks are |
|---|---|---|
| The Herdr TUI (`herdr`, 0.9.3+) | the repo root, with `herdr plugin install` | pane tokens, drawn by a `$herdr_mark` sidebar token |
| The Herdr desktop app (Herdr.app on herdr-core) | the `herdr-app/` folder, as a Herdr mod | terminal labels, drawn by a cloned Agents sidebar |

They do not share state: a mark set in one is not shown by the other.

# Herdr TUI

## Install

Requirements: Herdr 0.9.3+, Node.js 18+, and Python 3.11+. Supports macOS and Linux;
live verification has been performed on macOS.

Run these commands from a shell inside Herdr:

```sh
herdr plugin install sachin-philip/herdr-marks-as-unread
herdr plugin action invoke local.herdr-marks.setup
```

Setup backs up changed configuration, adds the red marker token to the start
of your existing second sidebar row, binds `prefix+m`, and reloads the config.
A marker left on another line by an earlier version is moved there, keeping
its colours. Rerun the setup action after updating the plugin.
It refuses to overwrite another command already using that shortcut. Check setup
logs with:

```sh
herdr plugin log list --plugin local.herdr-marks
```

The stable plugin ID is `local.herdr-marks` to remain compatible with the initial
local installation. If you already linked that version, unlink it before using
`plugin install` to switch to GitHub-managed updates:

```sh
herdr plugin unlink local.herdr-marks
```

## Use

1. Focus the agent you want to mark.
2. Press **Control+B**, release, then **M** (or your configured prefix, then M).
3. Switch away. When you return to the agent's terminal, its marker clears.

Repeat the shortcut to remove a marker manually. On macOS this uses Control,
not Command, and does not require Option. Multiple agents can be marked.
There is no custom sidebar right-click handler.

## Behavior and limitations

- Marking the terminal you are already viewing keeps the marker until you leave
  and return. Pane, tab, and workspace focus events trigger clearing.
- This is a personal unread marker, not a change to Herdr's `done`/`idle` state.
- Marks are pane metadata, shared by clients attached to the same server.
  Returning from another client can also clear a mark.
- Marks disappear on pane closure or server restart. A replacement agent in the
  same pane inherits the mark until it is cleared.
- Expanded sidebar rows are required. Status filters can still hide marked agents.
  Custom `rows_by_agent` layouts must also include the `$herdr_mark` token.
- Herdr always joins the marker to the next token with ` · `, so a marked agent
  reads `unread · idle · claude`. The separator is hard-coded in Herdr's
  sidebar renderer and no layout setting removes it.
- The marker cannot replace the state word. Herdr's token rules only see their
  own token's value, so `state_text` cannot hide itself when a mark is set.
- The plugin records the latest focus result and last verified clear under its
  Herdr-provided state directory, in `last-focus.json` and `last-cleared.json`.

## Manual configuration

Add the styled token before the first token of your existing second agent row,
or of the only row in a single-line layout. For example:

```toml
[ui.sidebar.agents]
rows = [
  ["state_icon", "machine", "workspace", "tab"],
  [{ token = "$herdr_mark", fg = "#f7768e", bold = true, dim = false }, "state_text", "agent"],
]

[[keys.command]]
key = "prefix+m"
type = "plugin_action"
command = "local.herdr-marks.toggle"
description = "Toggle agent marker"
```

Merge these entries with existing sections instead of duplicating them, then run
`herdr server reload-config`. Choose a different shortcut if `prefix+m` is occupied.

## Local development

```sh
git clone https://github.com/sachin-philip/herdr-marks-as-unread.git
cd herdr-marks-as-unread
python3 install.py
node --test marks.test.mjs
python3 -m unittest discover -p 'test_*.py'
```

`install.py` links the checkout and configures the sidebar. For a GitHub-managed
installation, the setup action uses `--configure-only` so it retains its managed
registration and update support. Do not move a linked checkout while using it.

From the source directory, you can toggle a specific live agent pane directly:

```sh
node marks.mjs toggle w1:p1
```

Replace `w1:p1` with an actual pane ID from `herdr agent list`.

## Update and uninstall

```sh
herdr plugin update local.herdr-marks
herdr plugin uninstall local.herdr-marks
```

Before uninstalling, clear any markers. After uninstalling, remove `$herdr_mark`
from your sidebar layouts and remove this plugin's `[[keys.command]]` entry, then
reload the config. For a local checkout, use `herdr plugin unlink` instead.

# Herdr desktop app

![Herdr app Agents list with a red unread badge on an idle agent](herdr-app/preview.png)

The desktop app does not run TUI plugins. For it, `herdr-app/` holds a mod: a
copy of the Agents sidebar that shows a red `unread` badge on any agent whose
terminal carries the label `herdr.mark=unread`, and clears it when you focus
that agent. `herdr-app/RECIPE.md` describes it in full.

## Install with your agent

Open Claude Code (or another agent) in a Herdr terminal and ask:

```
Add the Herdr mod from https://github.com/sachin-philip/herdr-marks-as-unread,
folder herdr-app. Follow its RECIPE.md.
```

The agent copies the mod, clones your layout to use it, adds the shortcut, and
runs `herdr-app check`. It will tell you the mod runs `hc`; that is how marks
are written.

## Install by hand

1. Copy `herdr-app/mods/agents-marks/` into `~/.config/herdr-app/mods/`.
2. Clone the layout you use (shipped layouts are read-only):

   ```sh
   herdr-app do layout.clone '{"name":"main","as":"herdr-marks"}'
   ```

3. In `~/.config/herdr-app/layouts/herdr-marks.json`, change `"mod": "agents"`
   to `"mod": "agents-marks"`. The window updates within a second.
4. Add the shortcut to `~/.config/herdr-app/keys.json`:

   ```json
   "layout:herdr-marks": { "cmd-shift-m": "agents-marks.toggle" }
   ```

   Editing keys.json makes the file yours: Herdr stops adding new default
   shortcuts to it, so add future ones by hand.
5. Check it:

   ```sh
   herdr-app check ~/.config/herdr-app/mods/agents-marks
   ```

## Use

- Focus an agent and press **Cmd+Shift+M** to mark it. Press again to unmark.
- Or hover its row in the Agents list and click the square icon. Click the
  `unread` badge to clear it.
- Focusing a marked agent clears the mark. Marking the agent you are viewing
  keeps it until you leave and return.
- From a shell: `hc label t4 herdr.mark=unread` sets it and `hc label t4 herdr.mark=`
  clears it. Marks are terminal labels, so every Herdr client sees them, and
  they end with the terminal.

## Limitations

- Only agents (terminals that report a status) are listed, so only they can
  show a mark.
- The mod is a clone of Herdr's `agents` mod at version 0.1.13 and does not
  receive Herdr's updates to the sidebar. Updating the mod means copying the
  folder again.
- Switch back to the shipped layout at any time with Cmd+Shift+L; your marks
  stay on the terminals but are not drawn there.

## Uninstall

Switch to the `main` layout, delete `~/.config/herdr-app/mods/agents-marks/`
and `~/.config/herdr-app/layouts/herdr-marks.json`, and remove the
`layout:herdr-marks` section from keys.json.

# License

MIT. See [LICENSE](LICENSE).
