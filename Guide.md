# CommitCrawl Guide for LLM Contributors

CommitCrawl is a community-built text-adventure dungeon. The project grows through small pull requests: usually **one PR adds one YAML room**. The engine loads those YAML files and lets players walk through the dungeon from room to room.

This guide is written for contributors using an LLM or coding agent. If you are an LLM reading this repository, use this file as your starting map.

## What this project is

CommitCrawl has two main parts:

1. **Rooms** — content files in `rooms/`
   - Each room is a `.yaml` file.
   - Rooms define title, description, exits, optional items, monsters, puzzles, ASCII art, and easter eggs.
   - Adding a room usually does not require writing JavaScript.

2. **Engine and viewers** — code that reads rooms
   - `engine/node/play.js` runs the terminal game.
   - `web/` contains the static web viewer.
   - `scripts/validate.js` checks room validity.
   - `scripts/build-web-data.js` converts YAML rooms into `web/rooms.json` for the web viewer.

The core idea: contributors keep expanding the dungeon by adding connected rooms and optional locked `TODO-` exits for future contributors.

## Repository map

```text
.
├── rooms/                  # YAML room files
│   ├── 000-entrance.yaml   # Starting room
│   ├── _template.yaml      # Copy this when adding a room
│   └── level1/...          # Nested room groups are allowed
├── engine/node/play.js     # Terminal game engine
├── web/                    # Static web viewer
│   ├── app.js
│   ├── index.html
│   ├── rooms.json          # Generated from YAML
│   └── style.css
├── scripts/
│   ├── validate.js         # Validates room YAML and exits
│   └── build-web-data.js   # Rebuilds web/rooms.json
├── README.md               # Human-facing project overview
├── CONTRIBUTING.md         # Short contributor rules
└── package.json            # npm scripts
```

## Quick commands

Install dependencies:

```sh
npm install
```

Play the dungeon in the terminal:

```sh
npm start
```

Validate all rooms:

```sh
npm run validate
```

Rebuild web viewer room data:

```sh
npm run build:web
```

Preview the web viewer with any static server, for example:

```sh
npm run build:web
npx serve web
```

## Most common contribution: add one room

Follow this workflow when an issue or prompt asks to add a room.

### 1. Inspect existing rooms

Check the highest existing numbered room in `rooms/` and find a good place to connect your new room.

A room is reachable only if some existing room has an exit pointing to it, directly or indirectly from `000-entrance`.

### 2. Pick a new ID and filename

Use the next free number for top-level rooms:

```text
rooms/NNN-short-room-name.yaml
```

The `id` field must exactly match the filename without `.yaml`.

Example:

```text
rooms/010-clockwork-pantry.yaml
```

```yaml
id: 010-clockwork-pantry
```

Nested level rooms may use IDs like `l1c1-03-room-name`, matching their filename.

### 3. Copy the template

Start from:

```text
rooms/_template.yaml
```

Delete unused optional sections instead of leaving placeholder text.

### 4. Connect the room

A valid room must have at least one exit. For a good contribution, it should also be reachable from the existing dungeon.

There are two common patterns:

#### Pattern A: extend from an existing locked door

If an existing room has:

```yaml
exits:
  north: TODO-haunted-library
```

You can create:

```yaml
id: NNN-haunted-library
exits:
  south: existing-room-id
```

Then update the original room's exit from `TODO-haunted-library` to your new room ID:

```yaml
exits:
  north: NNN-haunted-library
```

#### Pattern B: add a new branch from an existing room

Add an exit in an existing room pointing to your new room, and add a return exit in the new room.

Existing room:

```yaml
exits:
  east: NNN-new-room
```

New room:

```yaml
exits:
  west: existing-room-id
```

Keep this minimal. Do not rewrite unrelated exits.

### 5. Optionally add a future locked door

Locked/unbuilt exits must start with `TODO-`:

```yaml
exits:
  north: TODO-frozen-balcony
```

This is encouraged because it gives the next contributor a clear place to build.

### 6. Validate

Run:

```sh
npm run validate
```

Fix any errors before submitting. Warnings about unreachable rooms usually mean no existing room links to your new room.

### 7. Rebuild web data if needed

If your change should appear in the web viewer, run:

```sh
npm run build:web
```

This updates `web/rooms.json`.

## Room YAML reference

Required fields:

```yaml
id: NNN-room-id

title: A Short Title

description: >
  Two or three flavorful sentences describing what the player sees, hears,
  smells, or fears.

exits:
  south: 000-entrance

author: your-name
```

Optional fields:

```yaml
ascii_art: |
  /\
 /  \
/____\

items:
  - name: Strange Key
    description: A key that hums when pointed toward debt.
    on_use: The key coughs politely.
    consumable: false

monster:
  name: Tiny Dragon Accountant
  greeting: Your invoice is overdue.
  difficulty: 2

puzzle:
  locked_exit: north
  description: The door is sealed by a riddle.
  question: Place the correct item to open the door.
  answer: Strange Key
  success_msg: The lock sighs and opens.

easter_egg: Say "balance" and the room briefly becomes symmetrical.
```

Valid exit directions are:

```text
north, south, east, west, up, down, in, out
```

## LLM contribution checklist

Before editing:

- Read `AGENTS.md`, `README.md`, `CONTRIBUTING.md`, and `rooms/_template.yaml`.
- Inspect existing room IDs before choosing a new number.
- Search for existing `TODO-` exits if you want to open a locked door.
- Check `git status` so you do not overwrite user work.

When adding a room:

- Add one room per PR or branch unless explicitly asked otherwise.
- Make the filename and `id` match exactly.
- Ensure the room has at least one real exit.
- Ensure the room is reachable from `000-entrance`.
- Use `TODO-...` only for intentionally unbuilt future rooms.
- Keep the tone playful, weird, and PG-13.
- Avoid heavy dependencies; rooms are YAML only.

After editing:

- Run `npm run validate`.
- Run `npm run build:web` if web data should be updated.
- Summarize changed files and validation results.

## How to inspect connectivity

The built-in validator checks for invalid directions, missing required fields, broken non-`TODO` exits, and warns about rooms with no incoming exits.

For a stricter mental model, every non-`TODO` exit should point to an existing room, and every room should be reachable from:

```text
000-entrance
```

If a room is unreachable, fix it by adding or updating an exit in an existing reachable room.

## Engine notes for LLMs

The terminal engine is intentionally simple:

- It loads all `.yaml` files under `rooms/` recursively.
- It starts at `000-entrance` by default.
- It supports commands like:
  - `look`
  - `go <direction>`
  - `take <item>`
  - `check <item>`
  - `use <item>`
  - `place <item>` for puzzles
  - `fight`
  - `inventory` or `i`
  - `quit`

If you modify engine behavior, read `engine/node/play.js` carefully and update docs/tests or validation logic as needed.

## Web viewer notes for LLMs

The web viewer does not read YAML directly. It fetches:

```text
web/rooms.json
```

That file is generated by:

```sh
npm run build:web
```

If rooms are added or exits change and the web viewer needs to reflect them, rebuild `web/rooms.json`.

## Safe editing rules

- Do not delete or rewrite existing rooms unless the user explicitly asks.
- Do not repoint another contributor's exits unless needed to connect your room or fix a broken reference.
- Keep changes focused.
- Prefer native Node.js APIs for scripts; avoid new dependencies unless truly necessary.
- Never hardcode secrets or use paid APIs.
- Do not commit or create branches unless the user asks.

## Good room example

```yaml
id: 010-clockwork-pantry
title: The Clockwork Pantry
description: >
  Shelves tick in perfect rhythm while cans of soup rotate like tiny moons.
  A brass toaster watches you with the suspicion of a retired guard captain.
exits:
  west: 005-cosmic-observatory
  down: TODO-biscuit-catacombs
items:
  - name: Wind-Up Cracker
    description: It walks in circles until someone compliments its crunch.
monster:
  name: The Soup Warden
  greeting: No sampling without a spoon license.
  difficulty: 2
easter_egg: Say "al dente" and every noodle in the room salutes.
author: your-name
```

Remember: the best CommitCrawl contribution is small, connected, valid, and memorable.
