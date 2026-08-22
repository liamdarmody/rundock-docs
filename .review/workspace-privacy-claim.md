# The `~/.rundock/` privacy claim: search record and sources

This file is the evidence for the change that corrected where per-user state lives. It is deliberately part of the change rather than a note written alongside it, so the coverage claim can be checked from the diff. It is not published: it sits outside the navigation in `docs.json`.

## What was wrong

The documentation said per-user state was kept outside the workspace folder, in a hidden directory at `~/.rundock/`, and warned readers not to place that directory inside Dropbox, OneDrive or Google Drive because it would leak secrets.

The directory is inside the workspace folder, which is the folder the same documentation tells teams to sync. A reader who followed the guidance exactly got the outcome it warned about, and had been told they were safe.

## How the affected pages were found

Searches were run over the whole repository with `git grep`, which covers every tracked file. The repository is 25 MDX files plus images, so this is exhaustive rather than sampled. Run from the repository root.

```
git grep -n -I -- '~/\.rundock'
git grep -n -I -- '\.rundock'
git grep -n -I -- 'rundock/secrets\|prefs\.json\|rundock/conversations/'
git grep -n -I -i 'outside the workspace'
git grep -n -I -i 'never shared\|never leaves\|stays local'
git grep -n -I -i 'secret\|credential\|api key\|oauth'
git grep -n -I -i 'per-user'
```

The first search finds the claim as written. The rest exist because the claim also appears without the path: as a directory that is "separate", as state that "never leaves your machine", and as a credential store that does not exist. Searching only for `~/.rundock` would have found two of the four pages.

### Before

`git grep -n -I -- '~/\.rundock'`

```
concepts/workspaces.mdx:40:  Rundock keeps these outside the workspace folder, in a hidden directory at `~/.rundock/`
concepts/workspaces.mdx:51:  | Lives in workspace folder (syncs) | Lives in `~/.rundock/` (stays local) |
concepts/workspaces.mdx:58:  If you accidentally place `~/.rundock/` inside a synced folder ... you will leak secrets
reference/workspace-structure.mdx:22:   ~/.rundock/     # per-user state (outside the workspace)
reference/workspace-structure.mdx:29:   The `~/.rundock/` directory is per-user and never shared.
reference/workspace-structure.mdx:70:   the credentials each MCP server needs come from each user's `~/.rundock/secrets/`
reference/workspace-structure.mdx:101:  ### `~/.rundock/`
reference/workspace-structure.mdx:113:  Do not place `~/.rundock/` inside a synced folder ... Doing so will leak secrets
reference/workspace-structure.mdx:128:  | `~/.rundock/state.json` | Outside workspace | **No** |
reference/workspace-structure.mdx:129:  | `~/.rundock/secrets/` | Outside workspace | **Never** | API keys and OAuth tokens. Leak risk. |
reference/workspace-structure.mdx:130:  | `~/.rundock/conversations/` | Outside workspace | **No** |
reference/workspace-structure.mdx:131:  | `~/.rundock/prefs.json` | Outside workspace | **No** |
reference/workspace-structure.mdx:153:  editing `~/.rundock/state.json` directly
```

The claim without the path:

```
trust/data-privacy-security.mdx:14: Per-user state lives in a separate `.rundock/` directory: conversation
                                    history ..., per-user preferences, and any local credentials.
concepts/search.mdx:20:             The index is a small local file inside your workspace's `.rundock/`
                                    folder ... It never leaves your machine.
```

### After

```
$ git grep -n -I -- '~/\.rundock'
(no matches)

$ git grep -n -I -- 'rundock/secrets\|prefs\.json\|rundock/conversations/'
(no matches)

$ git grep -n -I -i 'outside the workspace'
concepts/workspaces.mdx:54            (runtime sign-in, which is genuinely outside)
guides/team-workspace.mdx:35          (runtime sign-in, which is genuinely outside)
reference/workspace-structure.mdx:117 (runtime sign-in, which is genuinely outside)
trust/data-privacy-security.mdx:16    (runtime sign-in, which is genuinely outside)

$ git grep -n -I -i 'never shared\|never leaves\|stays local'
concepts/how-rundock-works.mdx:105    (card text: "what stays local", a section title)
concepts/workspaces.mdx:34            (section heading "What syncs, what stays local")
guides/team-workspace.mdx:22          (section heading "What syncs, what stays local")
quickstart.mdx:125                    (card text pointing at that section)
```

The four remaining `outside the workspace` hits describe the Claude Code and Codex sign-in, which each command-line tool holds in its own configuration. That is outside the workspace folder, and Rundock never reads it. The four `stays local` hits are headings and the cards that link to them.

The remaining three searches are broad by design: they exist to catch the claim phrased without the path, so they match accurate prose as well. Their after-state is a hit list rather than zero, and every hit was read.

```
$ git grep -c -I -- '\.rundock' -- '*.mdx'          # 30 lines across 8 files
concepts/conversations.mdx:1        (on-disk location, accurate)
concepts/search.mdx:1               (corrected in this change)
concepts/workspaces.mdx:4           (corrected in this change)
guides/team-workspace.mdx:6         (corrected in bad3ca2, git wording adjusted here)
reference/agent-file-format.mdx:1   (path reference, accurate)
reference/routine-format.mdx:1      (path reference, accurate)
reference/workspace-structure.mdx:12 (corrected in this change)
trust/data-privacy-security.mdx:4   (corrected in this change)

$ git grep -c -I -i 'secret\|credential\|api key\|oauth' -- '*.mdx'
concepts/how-rundock-works.mdx:1    (runtime sign-in held by the CLI, accurate)
concepts/runtimes.mdx:2             (the existence check, accurate)
concepts/workspaces.mdx:1           (the MCP key warning added here)
guides/team-workspace.mdx:4         (the MCP key warning, pre-existing and accurate)
reference/workspace-structure.mdx:2 (both rewritten here to say no store exists)
troubleshooting/authentication.mdx:2 (an API error message, unrelated)
trust/data-privacy-security.mdx:7   (six concern the model provider's terms; one rewritten here)

$ git grep -c -I -i 'per-user' -- '*.mdx'
concepts/workspaces.mdx:3           (all three now describe state inside the workspace)
guides/team-workspace.mdx:4         (MCP configuration and settings overrides, accurate)
reference/workspace-structure.mdx:8 (rewritten here; none now places the directory outside)
trust/data-privacy-security.mdx:1   (rewritten here)
```

No hit in these three carries the false location or the credential store. The searches that are expected to return nothing return nothing, and the searches that are expected to return prose return prose that was read.

## Every file that matched, and what happened to it

| File | What it claimed | Disposition |
|---|---|---|
| `concepts/workspaces.mdx` | Per-user state outside the workspace folder; a table column headed "stays local"; a warning against a placement that was always the default; API keys and MCP configuration listed as per-user | Corrected in this change |
| `reference/workspace-structure.mdx` | A layout diagram placing the directory outside the workspace with three children that do not exist; "per-user and never shared"; MCP credentials from a per-user secrets store; four sync-table rows marked outside the workspace | Corrected in this change |
| `trust/data-privacy-security.mdx` | Per-user state in a "separate" directory holding preferences and local credentials, on the page written to be forwarded to a compliance reviewer | Corrected in this change |
| `concepts/search.mdx` | Location correct, consequence wrong: the index "never leaves your machine", which fails on a shared workspace | Corrected in this change |
| `guides/team-workspace.mdx` | Carried the same claim until commit `bad3ca2`, "Docs: correct what stays private in a shared team workspace", merged 4 August 2026, which rewrote it to say the directory is inside the workspace and added the per-tool exclusion steps. It was not missed here. This change touches it again for two things that correction did not cover: its git exclusion wording, which still said the directory was handled for you and never committed, and the unqualified home-directory claim about the recent-workspace file | Already correct on the location; git wording and one qualifier adjusted |
| `concepts/conversations.mdx:43` | States the on-disk location as `.rundock/conversations.json` and `.rundock/transcripts/` | Accurate, no change |
| `reference/agent-file-format.mdx:128` | References `.rundock/state.json` | Accurate, no change |
| `reference/routine-format.mdx:100` | References `.rundock/routine-state.json` | Accurate, no change |
| `concepts/how-rundock-works.mdx`, `principles.mdx`, `quickstart.mdx` | Say the stack runs on your machine and files stay in your workspace. `how-rundock-works.mdx` also states that a shared folder syncs through the team's chosen layer | Accurate, no change |
| `troubleshooting/authentication.mdx`, `concepts/runtimes.mdx` | Concern runtime sign-in, which is held outside the workspace | Accurate, no change |

## Sources for every product-behaviour claim these pages now make

All paths are in the application repository, not this one.

| Claim | Source |
|---|---|
| The directory is `WORKSPACE/.rundock`, not `~/.rundock` | `lib/store/persistence.js:15`, `rundockDir()` returning `path.join(getWorkspace(), '.rundock')` |
| What the directory holds | The writers themselves: `persistence.js` for `conversations.json`, `lists.json` and `state.json`; `lib/store/transcripts.js:22` for `transcripts/`; `lib/scheduler.js:92,177,348` for `routine-state.json`, `routine-slots.json` and `runs/`; `lib/runtime/claude.js:91,184` for `scratch/` and `child-pids.json`; `server.js:2508,2617` for `startup.log` and `search-index.db` |
| Rundock appends `.rundock/` to the workspace `.gitignore` at setup | `lib/workspace/scaffold.js:270-281` |
| `.rundock/` is the only exclusion Rundock writes | `lib/workspace/scaffold.js:270-281` is the only writer of a workspace `.gitignore` anywhere in the source. Nothing writes an exclusion for `.claude/settings.local.json`, which is why that file has to be excluded by hand |
| It creates that file when there is none | `lib/workspace/scaffold.js:276`, `fs.appendFileSync`. Line 273 treats a missing file as empty content rather than branching, and `appendFileSync` defaults to flag `a`, which creates the file if it does not exist. The behaviour comes from that default rather than from a branch in the scaffold |
| That write is skipped when the file already mentions `.rundock` in any form | `lib/workspace/scaffold.js:274`, the guard `if (!existing.includes('.rundock'))`, a substring test. `.rundock-recent-workspaces.json` is a real filename in this product and would satisfy it |
| A failed write is not surfaced to the user | `lib/workspace/scaffold.js:280`, `console.warn` in a desktop application |
| `state.json` records where the workspace was last seen, to detect a move | `server.js:2565` reads `state.workspacePath` as the previous value, `server.js:2582` writes it only when the path differs, and the surrounding function returns `moved` |
| The recent-workspace list is a file in the home directory in the desktop app | `server.js:180`, `~/.rundock-recent-workspaces.json` under `RUNDOCK_ELECTRON`, and beside the server code otherwise |
| Rundock stores no credentials and only checks that a runtime's credentials file exists | No match for a credential store anywhere in the source; the existence check is already documented at `concepts/runtimes.mdx:60` |
| There is no `secrets/` directory, no `prefs.json`, no `conversations/` directory | No writer for any of the three. The conversation list is a single `conversations.json`; the theme preference is browser state under the key `rundock-theme` |

## Deliberately left alone

Moving the directory out of the workspace folder is separate planned work. Every page that describes the current location flags it as planned rather than present.

`.mcp.json` sits in the workspace folder, syncs on every option including git, and commonly holds an API key. The pages here no longer describe it as per-user state, which was false, but the underlying exposure is its own piece of work.
