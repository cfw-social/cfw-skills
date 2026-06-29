# cfw-skills

Public, self-contained CFW recipe skills. Each top-level folder is one ready-to-run
recipe (SKILL.md + its vendored `.hub/` closure). No archives — fetch the plain files.

Built and published from `cfw-social/cfw-skills-pack`. **Do not hand-edit** — changes
are overwritten on the next publish.

## Discover
- `index.json` — every recipe, its file list, and per-file sha256.

## Install a recipe (what an agent does)
```
GET https://raw.githubusercontent.com/cfw-social/cfw-skills/main/index.json                 # list recipes + files
for each file in index.recipes[<name>].files:
  GET https://raw.githubusercontent.com/cfw-social/cfw-skills/main/<name>/<file>  →  write to ~/.hermes/skills/cfw/<name>/<file>
```
Each recipe self-verifies its renders via its `acceptance.json` (the c-eval-runner gate).

_14 recipes in this release._
