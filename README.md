# Kanban Grid

An [Obsidian](https://obsidian.md) plugin that puts a whole semester of weekly [Kanban](https://github.com/mgmeyers/obsidian-kanban) boards on one pan-and-zoom canvas, in its own tab. Tick tasks straight from the canvas, or click a board to edit it in place.

It replaces the "Excalidraw drawing full of embedded Kanban boards" setup, which gets slow and crashes once a dozen boards are live at the same time.

## Features

- **One canvas, every week.** Boards are laid out in a grid automatically, sorted by week number.
- **Excalidraw-style navigation.** Drag or two-finger scroll to pan; pinch or Cmd/Ctrl+scroll to zoom.
- **Tick tasks anywhere.** Each checkbox click writes `[x]` (or `[ ]`) to that week's note.
- **Edit in place.** Click a board and it becomes the real Kanban view right on the canvas. At most two boards are live at once.
- **Hands-off.** New week notes appear by themselves; renamed or deleted ones disappear.
- **Several semesters.** Switch between them from a dropdown; by default it shows the newest.
- **Light.** Boards you aren't editing are static HTML styled by Kanban's own stylesheet, so a full semester costs about as much as one long note.

## Requirements

- Obsidian 1.5 or later (desktop).
- The [Kanban](https://github.com/mgmeyers/obsidian-kanban) community plugin, enabled. Kanban Grid uses its styles and opens its view for editing.

## Installation

Kanban Grid isn't in the community plugin directory, so it's installed by hand:

1. Download `kanban-grid-plugin-<version>.zip` and unzip it. You get a `kanban-grid` folder containing `main.js`, `manifest.json` and `styles.css`.
2. In Obsidian, open **Settings → Community plugins** and click the folder icon next to **Installed plugins**. This opens `<your vault>/.obsidian/plugins/`.
3. Move the `kanban-grid` folder there. The result must be `.obsidian/plugins/kanban-grid/main.js`, not a folder nested twice.
4. Back in **Community plugins**, click the refresh icon and turn on **Kanban Grid**.

To update, replace the `kanban-grid` folder with the new one, then turn the plugin off and on again.

## Setting up your week notes

Kanban Grid reads notes **directly inside one folder** (not in subfolders) whose names follow this pattern:

```
<semester> - W<number>.md
```

Examples:

| File name               | Semester    | Week          |
| ----------------------- | ----------- | ------------- |
| `2026S2 - W9.md`        | `2026S2`    | 9             |
| `2026S1 - W14.md`       | `2026S1`    | 14            |
| `2026S2 - NA.md`        | ignored     | (no number)   |
| `2026S2 - W9 1.md`      | ignored     | (extra text)  |

Everything before ` - W` is the semester name, matched exactly as written: `2026S2` and `2026 S2` count as different semesters. Each note must be a Kanban board (with `kanban-plugin: board` in its frontmatter).

### Creating weeks automatically with Templater

A [Templater](https://github.com/SilentVoid13/Templater) template can name and file each new week for you. Name the template after the semester (for example `2026S2`) and set the semester's start date:

```js
<%*
let startDate = moment("2026-01-05").startOf('day');   // first day of week 1
let targetFolder = "Weeks";                            // same as the plugin's Folder setting
let weekNum = Math.floor(moment().startOf('day').diff(startDate, 'days') / 7) + 1;
let result = weekNum < 1 ? "NA" : `W${weekNum}`;
if (tp.file.title.startsWith("Untitled")) {
  await tp.file.move(`${targetFolder}/${tp.config.template_file.basename} - ${result}`);
}
%>
```

Put your Kanban lanes below it. Each new note created from the template lands in the folder as `2026S2 - W<n>` and shows up on the canvas. For a new semester, copy the template, rename it (for example `2027S1`) and change the start date.

## Usage

Open the canvas from the **grid icon** in the left ribbon, or the command palette: **Kanban Grid: Open**. To give it a shortcut, go to **Settings → Hotkeys**, search for "Kanban Grid", and assign one to **Kanban Grid: Open**.

The tab remembers your position, zoom and semester, and stays open between sessions like any other tab.

| Action          | How                                                                                                  |
| --------------- | ---------------------------------------------------------------------------------------------------- |
| Pan             | Two-finger scroll, mouse wheel (Shift+wheel for sideways), or drag anywhere outside a live board     |
| Zoom            | Pinch, Cmd/Ctrl+scroll, or the − / + buttons at the bottom left                                       |
| Fit everything  | The fit button at the bottom left, or the command **Kanban Grid: Zoom to fit**                       |
| Back to 100%    | Click the zoom percentage                                                                            |
| Tick / untick   | Click a checkbox. A drag that starts on a checkbox pans instead of ticking.                          |
| Edit a board    | Click anywhere on it, or its pencil icon. It turns into the real Kanban view with an accent outline. |
| Make it taller  | Drag the bottom edge of a live board                                                                 |
| Stop editing    | The minimize icon in the live board's header, or **Close all live boards** at the bottom left        |
| Open a week     | Click its name or the arrow icon in its header; it opens in a new tab                                |
| Switch semester | The dropdown at the top left. **Newest** follows the semester you most recently created a note for. |
| Reload          | The reload button at the bottom left                                                                 |

Inside a live board, drags move cards and scrolling scrolls the board. Pinch or Cmd/Ctrl+scroll still zooms the canvas; move the pointer off the board to pan. Editing is easiest at 100% zoom.

When you open a third board for editing (with the default limit of two), the board you opened first switches back to the lightweight copy.

## Settings

**Settings → Kanban Grid**

| Setting             | Default                           | What it does                                                                                          |
| ------------------- | --------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Folder              | `Weeks`                           | Folder that holds the week notes, relative to the vault root.                                         |
| Boards per row      | 3                                 | Columns in the grid (1–6).                                                                            |
| Live boards at once | 2                                 | How many boards can be in edit mode together (0–4). More uses more memory. 0 opens a new tab instead. |
| Week order          | First week first                  | Or latest week first.                                                                                 |

**Kanban's own settings** also apply, globally (**Settings → Kanban**) or per board (a live board's ⋯ menu → **Open board settings**). The static copies follow collapsed lanes, lane width, hidden card counts, card checkboxes, "Add a card" position, and lane limits such as `## COURSE (5)`.

## How ticks are written

Cards in the week notes look like `- [ ] - [ ] Lecture`. The first box is the card itself and the second is the task the board displays. Ticking changes only that one character, using the same rule as the Kanban plugin.

Before writing, the plugin checks that the card's text still matches what it displayed. If the note changed in the meantime (for example, edited on another device), it reloads the board instead of writing. If Kanban's **Show checkboxes** setting is on, the outer card checkbox is shown and can be ticked too.

## Privacy and security

- **No network access.** The plugin makes no web requests and has no analytics or telemetry. Its only runtime dependency is Obsidian's own API.
- **Writes only what you tick.** It changes a file only when you click a checkbox, and only that single character in a week note inside the configured folder. Edits made in a live board are saved by the Kanban plugin, as in a normal tab.
- **Stored data.** `.obsidian/plugins/kanban-grid/data.json` holds the four settings above. The canvas position, zoom and chosen semester are saved in Obsidian's workspace layout. No note content is stored.
- **Rendering.** Card text is rendered with Obsidian's own Markdown renderer, which sanitises HTML the same way Reading view does. File and folder names are inserted as plain text.

## Limitations

- **Live editing relies on an internal Obsidian API.** It uses the same technique as Excalidraw embeds and Hover Editor (a workspace split that isn't attached to the main layout). A future Obsidian update could break live editing. The canvas, the copies and ticking use only the public API and would keep working.
- **Copies follow Kanban 2.0.51's class names.** If Kanban renames its CSS classes, the copies may look slightly different until this plugin is updated.
- **Not shown on copies:** card dates, tag colours, and Kanban's table and list views. They appear once a board is live.
- **Buttons on copies are decorative.** The ⋯ menus and "Add a card" on a copy just switch that board to edit mode.
- **One folder.** Week notes in subfolders aren't picked up.

## Development

```bash
npm install
npm run dev     # rebuild main.js on every change
npm run build   # type-check, then production build
npm test        # parser and checkbox tests
```

For a quick dev loop, clone the repo into `<vault>/.obsidian/plugins/kanban-grid/`, run `npm run dev`, and reload the plugin after each change.

```
src/
  main.ts       plugin entry: view, ribbon icon, commands, settings tab
  view.ts       the canvas tab (saves position, zoom and semester)
  canvas.ts     pan/zoom, layout, semester picker, live-board limit
  board.ts      one board: static copy, checkbox writes, switching to live
  liveLeaf.ts   hosts a real Kanban view inside the canvas
  discover.ts   finds "<semester> - W<n>" notes in the folder
  parser.ts     reads Kanban markdown and toggles checkboxes (no Obsidian imports)
  settings.ts   settings and the settings tab
test/
  parser.test.ts, fixtures/week.md
```

## License

[MIT](LICENSE) © 2026 Parun Thummadetsak
