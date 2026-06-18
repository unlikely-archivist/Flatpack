# Flatpack
**An Obsidian Templater Template** that combines some of all of your notes into Master documents, including your embeds while restraining "embed echo" that comes from heavily cross embedded vaults and retaining your folder structure.

(I got really sick of updating a master doc every time I added a file or folder, and even sicker of having the same embed removed from a .baked file 16 times. Flatpack will collect your files, retain your folder structure and only include an embed the firtst time it shows up)


## The problem

When you embed the same note in multiple places across your vault, traditional baking tools duplicate that content — you end up with a bloated document where important sections repeat 6+ times.

## The solution

Flatpack combines your notes into one clean master document, embedding all content but eliminating duplicates. Repeated embeds become `> 🔁 See:` pointers instead. Your folder structure is preserved as heading levels (top folder = `##`, subfolder = `###`, files = `####`, etc.).

**Example:** A typical heavily cross-embedded vault that bakes to 104k lines combines into 9.7k lines with Flatpack.

## Install

1. Copy `Flatpack.md` into your Obsidian `templates` folder
2. Open any note → Templater → Insert Template → Flatpack
3. A combined master document appears

## What you can combine

- **Your whole vault:** `ROOTS = []`
- **Specific folders only:** `ROOTS = ["01 Operations","03 GPTS"]`
- **Everything except certain folders:** use `EXCLUDE_FOLDERS`
- **Everything except certain notes:** use `EXCLUDE_FILES`

## Configure

Edit the five settings at the top of the template:

| Setting | What it does |
|---------|--------------|
| `TITLE` | Output note name (e.g., `"Master"`) |
| `ROOTS` | `[]` = whole vault; or `["01 Operations","03 GPTS"]` to combine only those |
| `EXCLUDE_FOLDERS` | Folder names to skip (e.g., `["templates","archive"]`) |
| `EXCLUDE_FILES` | Note names to skip (e.g., `["WIP","Untitled"]`) |
| `OUT_FOLDER` | Where to save it (e.g., `"00 Master"` or `""` for vault root) |

## Example configs

**Combine your whole vault:**
```javascript
const TITLE = "Master";
const ROOTS = [];
const EXCLUDE_FOLDERS = ["templates"];
const EXCLUDE_FILES = [];
const OUT_FOLDER = "";

