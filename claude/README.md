# Set this up for me

**You are an AI coding agent. A person has unzipped this folder and asked you to set up their Claude Code. Follow this file top to bottom. Every file it names is in this folder. It works on macOS, Linux, WSL and native Windows.**

This builds a small system of plain text files, seven small hooks, two helper agents, one memory tool and one notes vault. Together they let Claude Code start each session already knowing where the work stands, cost fewer tokens, write less like a machine, and fix itself when something goes wrong.

**Your job is to install a skeleton, never to copy someone's rules.** Leave the rule files almost empty. A rule nobody earned is a rule nobody follows, and it is paid for in tokens on every message.

---

## 0 · Open: greet, scan, ask, plan, consent

Do these five steps in order. **Create no file before step 5 is answered.**

### Step 1 · Say what this is

> 👋 Hi. This is Moe's Claude Code setup, and I'll install it on your machine.
>
> Right now Claude forgets everything when a session closes, so you explain your project again every time. This adds a few plain text files, seven small scripts that run on their own, two cheap helpers, a memory tool and a notes vault, so each session starts where the last one stopped.
>
> I'll look at your machine first, then show you the plan before I touch anything.

### Step 2 · Scan the machine and report

Find out, and report plainly:

| check | macOS / Linux / WSL | native Windows (PowerShell) |
|---|---|---|
| operating system | `uname -a` | `$PSVersionTable; [Environment]::OSVersion` |
| Claude Code version | `claude --version` | `claude --version` |
| config folder | `ls -la ~/.claude` | `Get-ChildItem "$env:USERPROFILE\.claude" -Force` |
| Python 3, its full path, and whether that is a virtual env | `python3 -c "import sys; print(sys.executable, sys.prefix != sys.base_prefix)"` | `py -3 -c "import sys; print(sys.executable, sys.prefix != sys.base_prefix)"` (then try `python`) |
| Git, and on Windows Git Bash | `git --version` | `git --version` |
| tools already present | `rtk --version; icm --version` | `rtk --version; icm.exe --version` |
| Obsidian installed | macOS `ls /Applications \| grep -i obsidian` · Linux `which obsidian` · WSL: check on the Windows side | `winget list --id Obsidian.Obsidian` |

Then say what will and will not work:

- **No Python 3** → the hooks need it. Offer to install it (macOS `brew install python` · Linux/WSL `sudo apt install python3` or the distro's package manager · Windows `winget install -e --id Python.Python.3.12`, or the version `winget search Python.Python` lists) or skip the hook part and say so.
- **Native Windows without Git for Windows** → Claude Code still runs, using its PowerShell tool for commands. Recommend installing Git for Windows so the Bash tool works. The hooks below run without any shell and watch both tools, so they work either way.
- **A tool below has no installer listed for this system** → skip that tool, say so in the report, and carry on. Never improvise an install command.
- **WSL** → treat it as Linux, and use the WSL home folder. The WSL home and the Windows user folder are different places; files written to the wrong one are never read. Obsidian runs on the Windows side and opens a vault in WSL through `\\wsl.localhost\<distro>\...`.
- On Windows, `~/.claude` means `%USERPROFILE%\.claude`. Confirm it exists; if it does not, ask them to run Claude Code once. **Never create a config folder in a guessed place.** A second folder in the wrong spot is worse than none, because the real one keeps winning and nobody sees why.

Write down the full Python path from the scan. Every hook uses it. **If the second value printed is `True`, that Python lives in a virtual environment** that can be deleted later and take every hook with it: deactivate it and scan again, or use the system Python (`/usr/bin/python3`, or on Windows `C:/Windows/py.exe` with `"-3"` as the first item in `args`).

**Claude Code version:** 2.1.295 or later uses every setting here. On an older version, drop the `onFailure` line from §6 and the `omitClaudeMd` line from §7, and say so.

### Step 3 · Ask what they will use it for

Ask these four, and wait. Do not guess any of them.

1. **What is this project for?** One sentence. It goes at the top of the project rules.
2. **What must never happen here?** Anything irreversible, expensive or public. None is a fine answer.
3. **Where should the notes vault live?** Default: a folder named `vault` next to the project. If they already use Obsidian, use their vault.
4. **Is there already a convention for where files go?** Yes → use theirs. No → propose §3.

### Step 4 · Show the plan

> 📋 Here's the plan. Nothing happens until you say go.
>
> **Files** (plain text, yours to edit or delete)
> - `~/.claude/CLAUDE.md` → how Claude works on every project
> - `<project>/CLAUDE.md` → what is true about this project
> - `SESSION_LOG.md`, `MISTAKES.md`, one `STATE.md` per piece of work → where things stand and what went wrong
> - `<vault>/` → an Obsidian vault for what you know about your domain
>
> **Hooks** (seven small Python scripts that run on their own)
> - block reading credential files · back up rules before an edit · flag machine-sounding text · show where you left off when a session opens · point to the vault note for what you asked · remind Claude to pick the right model for a helper · keep decisions when a long chat is summarised
>
> **Tools**
> - RTK → shrinks command output before it fills the conversation
> - ICM → searchable memory between sessions
> - caveman → shorter replies
> - Obsidian → the app to read and edit the vault (optional; the vault works without it)
>
> ⏱️ About fifteen minutes. Shall I go ahead?

### Step 5 · Consent, then work

Wait for a yes. Then work through §1 to §9, saying one short line per step as you go. At the end run §10 and report with §12.

---

## 1 · Where a thing goes

This is the whole design. Every later section is one row of it.

```
something worth keeping turned up
   │
   ├─ how Claude should work, on any project ───→  ~/.claude/CLAUDE.md
   ├─ true about this project only ─────────────→  <project>/CLAUDE.md
   ├─ a check a script can see (a path, a word,   →  a hook
   │  a command)
   ├─ steps for a task, done more than once ────→  a skill
   ├─ what you know about your field, your data,  →  a vault note
   │  your systems, your people
   ├─ a decision and why, wanted months from now →  memory (ICM)
   ├─ where one piece of work stands ───────────→  that work's STATE.md
   ├─ one line, findable later ─────────────────→  SESSION_LOG.md
   └─ Claude got something wrong ───────────────→  MISTAKES.md, then §11
```

Two splits people get wrong:

- **Skill or vault note?** A skill holds STEPS that work anywhere (how to review code, how to write a guide). A vault note holds KNOWLEDGE about one field (what this table means, how this client counts a sale). A skill for a domain task holds only its trigger and a pointer to the vault note that owns the facts.
- **Rule or hook?** A rule in CLAUDE.md is advice; Claude can miss it. A hook runs every time. A rule that keeps getting missed, and that a script can see, becomes a hook.

---

## 2 · The two rule files

Both are read at the start of every session and stay in the conversation, so every line is paid for on every turn. Keep the global file under 8 KB; §10 checks it.

### 2a · `~/.claude/CLAUDE.md`

Use as is. It is all method, nothing about any field.

→ copy from the zip: `templates/global-CLAUDE.md`

### 2b · `<project>/CLAUDE.md`

Nearly empty. Fill only what step 3 answered.

→ copy from the zip: `templates/project-CLAUDE.md`

---

## 3 · Folders

```
<project>/
├── CLAUDE.md         rules for this project
├── SESSION_LOG.md    one line per finding
├── MISTAKES.md       what Claude got wrong, and where it was fixed
└── work/
    └── <name>/
        └── STATE.md  where this piece of work stands

<vault>/              the Obsidian vault (§5)
```

Match whatever they already use. The shape is what matters.

---

## 4 · The record files

### `SESSION_LOG.md`

→ copy from the zip: `templates/SESSION_LOG.md`

Test for a line: would it still mean something to someone on a different part of this next month? No → it goes in a `STATE.md`.

### `STATE.md`, one per piece of work

→ copy from the zip: `templates/STATE.md`

The header answers "where was I". The history answers "why is it like this". The session-start hook shows each header that changed in the last two weeks. **If the project keeps work somewhere other than `work/<name>/`, edit `STATE_GLOB` at the top of `hooks/session_start.py` to match** (for example `tickets/*/STATE.md`).

### `MISTAKES.md`

→ copy from the zip: `templates/MISTAKES.md`

`seen` counts repeats. §11 says what to do at 2.

---

## 5 · The notes vault (Obsidian)

A vault is a folder of Markdown files. The Obsidian app reads it and adds a hidden `.obsidian` settings folder the first time it opens it. Claude reads and writes the same files directly, so the vault works with or without the app.

1. **They already have a vault** → use it. Add `Map.md` at its root if there is none.
2. **No vault** → create this at the path from step 3:

```
<vault>/
├── Map.md          the index: one line per note, "question → note"
├── Systems/        how each system, table or tool works, and its traps
├── Methods/        how this field does a task (the knowledge a skill points to)
└── People/         who decides what, how they like things
```

`Map.md`:

→ copy from the zip: `templates/vault/Map.md`

3. **Install the app** if they want it: macOS `brew install --cask obsidian` · Windows `winget install -e --id Obsidian.Obsidian`. Then ask them to open Obsidian and pick **Open folder as vault** with the vault folder. Do not try to register the vault from the command line.
4. If the vault is not inside the project, ask whether to make it a git repo (`git init` in the vault). History is how a deleted line comes back.

**How the vault grows:** after a piece of work, a fact about the field that the next session would need goes into the note that owns that subject, in place. A new note only when no note owns the subject; then add its line to `Map.md`.

---

## 6 · Hooks

A hook is a script Claude Code runs on an event, every time, whatever the model is thinking. Install these seven. They need only Python 3; nothing else.

Copy the zip's `hooks/` folder to `~/.claude/hooks/`:

- `secret_guard.py` → refuses to open credential files (`.env`, keys, `credentials.json`), through any tool or shell
- `config_backup.py` → copies a CLAUDE.md, SKILL.md or Claude settings file to `~/.claude/setup-backups/` before it is edited
- `slop_check.py` → after Claude writes prose, names the new lines that read machine-written
- `session_start.py` → when a session opens, shows the end of `SESSION_LOG.md` and each recent `STATE.md` header
- `agent_guard.py` → refuses to start a helper until a model is picked for its job (§7)
- `vault_router.py` → when you type, shows the vault notes whose `Map.md` line shares words with your question
- `precompact.py` → before a long chat is summarised, tells the summary to keep decisions, the goal and open items
- `sample_event.json` → a planted `.env` read, for testing the guard by hand (§11)

### Wire them into `~/.claude/settings.json`

Merge `settings-hooks.json` into `~/.claude/settings.json`. No file yet → create it with this content. A file already there → keep every key, and **append** these entries to any `hooks` event list that already exists; never replace someone's existing hooks. Replace `PYTHON` with the full Python path from step 2, `HOOKS` with the full path of `~/.claude/hooks`, and `VAULT` with the full path of the vault. **Use forward slashes on every platform**, including Windows (`C:/Users/name/.claude/hooks`).

→ copy from the zip: `settings-hooks.json`

`args` runs the script directly, with no shell in between, so the same entry works on macOS, Linux, WSL and Windows. `cleanupPeriodDays` keeps 90 days of session history (the default is 30), long enough to look back at what was asked.

Claude Code picks up hook changes on its own. Type `/hooks` to see them listed. `onFailure: block` on the credential guard means that if the script ever crashes or Python goes missing, file reads are refused until it is fixed, so a broken guard never quietly lets keys through.

---

## 7 · Helpers, and the right model for each job

### When to start a helper

- **Yes:** reading many files to answer one question, a second opinion on work that has to be right, drafting wording, jobs that do not depend on each other and can run side by side.
- **No:** a lookup you can do in one or two commands, or a job whose next step needs your own judgment at every turn. A helper starts from zero and you only see its summary, so small jobs cost more as helpers.
- **The one that checks is never the one that made it.** Work someone will act on gets a second helper that reads the result cold.
- A helper's finding is a lead. Verify it before acting on it.

### Every helper prompt carries

```
GOAL: <what to find or make>
ALREADY KNOWN: <what you ruled out, so it does not redo it>
RETURN: <the shape you want back: a list, a table, file:line>
```

**The model** is set two ways: the `model:` line in a helper's file (the two below have one), or the Agent tool's `model` setting at launch. The built-in helpers (Explore, Plan, general-purpose) have none, so without it they run on the main session's model, usually the most expensive. The `agent_guard.py` hook refuses a launch that has neither, before anything is spent, and says which model fits; Claude picks one and launches again. To get a reminder instead of a refusal, set `BLOCK = False` at the top of the file (the reminder then arrives after that launch and applies to the next one).

**How long it runs** is capped by `maxTurns:` in the helper's file. At the limit, Claude Code returns what the helper has so far, marked partial.

### Pick the model and effort for the job

| job | model | effort |
|---|---|---|
| wording, formatting, renaming, pulling fields out of text | haiku | low |
| reading files and logs, gathering facts, routine code, checking work against a written list | sonnet | medium |
| planning, design, debugging something not yet understood, judging work with no clear answer | opus | high |
| a decision still stuck after two tries, a last review before something is published | the top model your plan offers | high |

Start at the lowest row the job allows. Move one row up only when a check finds the cheaper model missed something; move back down when two runs in a row pass. The main session works the same way: `/model` and `/effort` change it for the job in front of you.

### The two helpers in this kit

A helper (subagent) is a second Claude that does one job and reports back. Two cheap ones cover most of the work. Copy the zip's `agents/` files to `~/.claude/agents/`.

`reader.md`

→ copy from the zip: `agents/reader.md`

`drafter.md`

→ copy from the zip: `agents/drafter.md`

`omitClaudeMd: true` keeps the personal and project rule files out of the helper, so it starts small; the prompt carries any rule it needs. On one setup, moving wording jobs to a Haiku drafter cut those jobs from about 114,000 to 13,000 tokens with no loss in quality.

---

## 8 · Tools

Run each tool's own setup and its own check. **Do not invent commands.** If one below is not on their version, run `--help` and use what is there.

### RTK, shrinks command output

| | macOS / Linux | Windows |
|---|---|---|
| install | `brew install rtk` (Linux/WSL without Homebrew: `curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh \| sh`) | `winget install rtk-ai.rtk` |
| wire into Claude Code | `rtk init -g --auto-patch` | `rtk init -g --auto-patch` |
| check | `rtk gain` shows a savings table | same |

⚠️ Another tool is also called `rtk`. If `rtk gain` says the command is unknown, the wrong one is first on PATH. Never install it with `cargo install rtk`.

Tell them two traps: RTK rewrites `diff` into a summary (use `cmp` to test if files match), and counts piped through it can be wrong (`wc -l file` is right).

### ICM, memory between sessions

| | macOS / Linux | Windows (PowerShell) |
|---|---|---|
| install | `brew tap rtk-ai/tap && brew install icm` (Linux/WSL without Homebrew: `curl -fsSL https://raw.githubusercontent.com/rtk-ai/icm/main/install.sh \| sh`) | `irm https://raw.githubusercontent.com/rtk-ai/icm/main/install.ps1 \| iex` |
| wire into Claude Code | `icm init` | `icm.exe init` |
| check | `icm config`, and `icm doctor` if `icm --help` lists it | same, with `icm.exe` |

On Windows type `icm.exe`: in PowerShell `icm` alone is a built-in alias for something else.

**Turn off automatic capture.** `icm init` adds hooks that save memories from tool output on their own. On a working setup they filled the store with thousands of near-useless entries, and recall got worse. Open `~/.claude/settings.json` and remove the entries whose command contains `icm hook post`, `icm hook compact` or `icm hook end`. Keep the others. `icm init` also adds a short block of instructions to `~/.claude/CLAUDE.md`; leave it, and if it says to store memories as you go, change that line to "store a decision and its reason when a piece of work finishes". Memory is then written on purpose: one entry per finished piece of work, holding a decision and its reason.

Then prove recall works, in different words than the store:

```
icm store -t setup -c "the staging database is the one named blue" -i medium
icm recall "which database is staging"
```

On Windows write `icm.exe` in place of `icm` on every line. Empty result → `icm embed` (`icm.exe embed`), then try again.

### caveman, shorter replies

In Claude Code: `/plugin marketplace add JuliusBrussee/caveman`, then `/plugin install caveman@caveman`. Check by typing `/caveman`. It steps aside for warnings and destructive steps; that is intended.

### Obsidian

Covered in §5.

---

## 9 · Keep it cheap as it grows

- **Rule files stay small.** Both CLAUDE.md files load on every message. §10 warns over 8 KB. When one grows, move steps into a skill and facts into the vault; the rule file keeps the line that says where to look.
- **Skills cost almost nothing until used.** Only their one-line description loads; the body loads when it matches. Write the description as the trigger.
- **Facts that can be looked up are pointers.** A list of tables, versions or options goes stale silently. Write the command that returns it.
- **Compaction.** When a conversation gets long Claude Code summarises it (compaction). Leave it on. The `precompact.py` hook tells the summary to keep your decisions, the goal, open items and file paths and to drop raw tool output, and `session_start.py` runs again right after, putting the log and STATE headers back in view. Claude Code also compacts a long chat you walked away from (idle compaction), so the next message is cheaper; turn that off with `"idleCompaction": false` only if a summary ever lost something you needed.

---

## 10 · Quality checks

Copy the zip's `check_setup.py` to `~/.claude/` and run it with the Python from step 2:

```
<python> <home>/.claude/check_setup.py <project folder> <vault folder>
```

It runs each hook through the exact command settings.json uses, so a wrong Python path fails here. It plants a bad case for every hook (a `.env` read, including one under an accented folder name and one through PowerShell; a machine-sounding line; an edit to CLAUDE.md) and confirms the hook catches it, then confirms clean cases pass (a `.env.example`, a commit message that mentions credentials). **A check that never failed proves nothing**, so before reporting, break one thing on purpose (rename `Map.md`), run it, confirm it says FAIL, and put it back.

Then check by hand:

- [ ] New session opened in the project shows the end of `SESSION_LOG.md` (session-start hook works live, not only in the test).
- [ ] Asking Claude to read a file named `.env` is refused.
- [ ] `/hooks` lists all seven.
- [ ] Memory recall returned the stored line in different words.
- [ ] The vault opens in Obsidian, if installed.
- [ ] No rule was invented. Both CLAUDE.md files hold only §2, the step 3 answers, and the short blocks RTK and ICM add for themselves.

Report every FAIL with its reason. A setup with one known gap is fine; one that claims to be clean and is not wastes their week.

---

## 11 · When something breaks

Teach this to the person, and follow it yourself in every later session.

### Fix the layer that failed

When Claude gets something wrong, ask what was missing at that moment, then fix that place:

| what was missing | fix it here |
|---|---|
| a rule existed and was ignored | turn it into a hook if a script can see it; else move it into the skill that loads for that task |
| Claude did not know the steps | a skill |
| Claude did not know a fact about the field | the vault note owning that subject |
| a file told it the wrong thing | fix or delete that line; never stack a new rule on top of it |
| a tool misbehaved | the tool's setup (below) |

Log each one in `MISTAKES.md`. **Seen twice → make it mechanical** (a hook or a check). A rule nobody can enforce comes back.

### Common breaks

| symptom | cause | fix |
|---|---|---|
| hook never runs | wrong Python path, backslashes, or invalid JSON in settings.json | run `check_setup.py`; validate the JSON; use forward slashes |
| hook runs but nothing happens | it crashed | run it by hand with the sample event (below) and read the error |
| Windows: hook says file not found | `~` or `$HOME` in the path | write the full path; `args` form never expands `~` |
| every file read refused | the credential guard crashed (`onFailure: block`) | run it by hand with the sample event; usually the Python path moved |
| rules ignored | rule file too long, or in the wrong folder | check §10's size line; confirm `~/.claude` is the folder Claude Code reads |
| `rtk gain` unknown command | the other `rtk` on PATH | `which rtk` / `where.exe rtk`; reinstall from §8 |
| memory recall empty | no embeddings yet | `icm embed` |
| memory full of noise | automatic capture still on | §8: remove `icm hook post` and `icm hook compact` |
| a session starts lost | STATE.md header stale | update PHASE and NEXT at the end of each piece of work |
| rules file edited badly | | restore from `~/.claude/setup-backups/` (made by `config_backup.py`) |
| vault note never shown | the `Map.md` line does not start with `- `, has no `->`, or its question shares fewer than two words with how you ask | rewrite the question part the way you would ask it |
| helper launch refused | no model picked (§7) | Claude sets the Agent tool's model and launches again; for a helper you use often, add a `model:` line to its file |
| a summary lost a decision | compaction | write decisions to STATE.md as they land; the summary is a backup |

### How to debug any hook

1. Run it by hand with the sample event shipped in `hooks/sample_event.json` (a `.env` read; the guard should print a deny):
   - macOS / Linux / WSL: `<python> <hooks>/secret_guard.py < <hooks>/sample_event.json`
   - Windows PowerShell: `Get-Content "<hooks>\sample_event.json" | & "<python>" "<hooks>\secret_guard.py"`
   Run it yourself in a terminal; Claude's own command would be refused by the guard because it names `.env`.
2. Read what it prints. Empty output means it allowed the action.
3. Fix, then run `check_setup.py` again. It must go back to all PASS.

---

## 12 · Close

End with what changed, the two habits, and one honest note.

> ✅ **You're set up.**
>
> - 🧠 New sessions start with your log and your open work in view.
> - 🔒 Credential files are refused (`.env`, keys, `credentials.json`), and rule files are backed up before every edit.
> - ✂️ Command output is trimmed and replies are shorter.
> - 📚 Your field's knowledge has a home: the vault, starting at `Map.md`.
>
> **Two habits**
> - 📓 Found something out → one line in `SESSION_LOG.md`.
> - 🔧 Claude got something wrong → one row in `MISTAKES.md`, and fix the layer that failed.
>
> The rule files are nearly empty on purpose. They fill with what you learn, so in a month yours will look nothing like the one you started from.
