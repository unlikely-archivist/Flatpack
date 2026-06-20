# Flatpack

**An Obsidian Templater template** that combines some or all of your notes into one Master Document — pulling in your embeds while restraining the embed "echo" from heavily cross-embedded vaults, and keeping your folder structure intact.

## The echo problem

Embed the same note in ten places, then bake your vault, and that note's content gets copied ten times. Shared sections balloon. A clean vault turns into a bloated master document where the important stuff repeats over and over:

# Master
## ALL GPT KNOWLEDGE
[1,200 lines of content]
...
## ALL GPT KNOWLEDGE
[the same 1,200 lines]
...
## ALL GPT KNOWLEDGE
[again]
```

## What Flatpack does instead

It keeps the first copy and turns every repeat into a one-line pointer:

```
# Master
## ALL GPT KNOWLEDGE
[1,200 lines — once]
...
> 🔁 See: ALL GPT KNOWLEDGE
...
> 🔁 See: ALL GPT KNOWLEDGE
```

Same information, none of the bloat. In testing, a vault that baked to 104,000 lines came out as 9,700 with Flatpack.

Your folder structure becomes the heading hierarchy automatically: a top-level folder = `##`, a subfolder = `###`, the files inside = `####`, and so on.

## Install

1. Copy `Flatpack.md` into your Obsidian `templates` folder
2. Open any note → Templater → Insert Template → Flatpack
3. Your Master Document appears

## Configure

Five settings live at the top of the template — these are the only things you touch:

| Setting | What it does |
|---------|--------------|
| `TITLE` | Name of the output note, and its top heading |
| `ROOTS` | `[]` for your whole vault, or a list like `["Projects","Research"]` to combine only those folders |
| `EXCLUDE_FOLDERS` | Folder names to skip (e.g. `["templates","Archive"]`) |
| `EXCLUDE_FILES` | Note names to skip (e.g. `["Untitled"]`) |
| `OUT_FOLDER` | Where to save the result (`""` = vault root) |

## Examples

**Your whole vault:**
```javascript
const TITLE = "Master";
const ROOTS = [];
const EXCLUDE_FOLDERS = ["templates"];
const EXCLUDE_FILES = [];
const OUT_FOLDER = "";
```

**Just your Projects folder:**
```javascript
const TITLE = "All Projects";
const ROOTS = ["Projects"];
const EXCLUDE_FOLDERS = [];
const EXCLUDE_FILES = [];
const OUT_FOLDER = "";
```

**Projects + Research, skipping templates and drafts:**
```javascript
const TITLE = "Working Set";
const ROOTS = ["Projects","Research"];
const EXCLUDE_FOLDERS = ["templates"];
const EXCLUDE_FILES = ["Untitled"];
const OUT_FOLDER = "Master";
```

## How it works

- Walks the folders you selected and combines every note into one document
- Folder depth sets the heading level, so your structure stays readable
- Keeps the first copy of any embedded note; later repeats become `> 🔁 See:` pointers
- Notes are sorted alphabetically within each folder
- File attachments (images, PDFs, spreadsheets) become `📎 [filename]` placeholders
- Broken links are flagged with `> (unresolved: [link])`
- Empty folders are skipped, and your source notes are never touched

## Requirements

- Obsidian with the Templater plugin installed

## Tips

- Run it from any note — it doesn't read the note you're in, it reads the vault
- Re-run anytime to regenerate as your vault grows
- Tweak `ROOTS` to make focused documents — one per project, one per area, whatever you need

## Tips

- Run it from any note — it doesn't read the note you're in, it reads the vault
- Re-run anytime to regenerate as your vault grows
- Tweak `ROOTS` to make focused documents — one per project, one per area, whatever you need

## Changelog

### v.1.1.0
- Added `FINAL_DESTINATION` setting to move files after creation
- Added page breaks (`---`) between files for cleaner PDF exports
- Fixed file path tracking after move operation
- Improved notification message to show actual save location
- Fixed duplicate code line
- Better error handling for move failures
- Auto-append date to filename (YYYY-MM-DD format) — customize `FILENAME` prefix only

### v.1.0.0
- Initial release: vault scanning, deduplication, folder-depth heading levels

## Requirements

- Obsidian with the Templater plugin installed
