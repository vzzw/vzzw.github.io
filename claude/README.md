# Set this up for me

**You are an AI coding agent. A person has handed you this file and asked you to set up their Claude Code environment. Follow it top to bottom.**

This builds a small system of plain text files, plus one memory tool, that let Claude Code carry knowledge from one session to the next instead of starting from nothing every time.

**What this does NOT include, deliberately:** no hooks, no scripts, no plugins. Those come later, if the person ever needs them. Everything else here is a markdown file a human can read and delete.

**Your job is not to copy someone else's rules.** You are installing a skeleton plus a way for it to fill itself in. Resist the urge to pre-populate it with sensible-sounding rules — a rule nobody earned is a rule nobody follows, and it costs tokens on every single message forever.

---

## 0 · How to open — greet, scan, ask, plan, then get consent

Do these five things in order. **Do not create a single file before step 5 is answered.**

### Step 1 · Say hello and say what you are

Open with something like this. Warm, short, and honest about what is coming:

> 👋 Hi — Moe asked me to set your Claude Code up the way he has his.
>
> Right now Claude forgets everything the moment you close a session. You re-explain your project, your conventions, and your data every single time. This fixes that: a handful of plain text files and three small tools, so it starts each session already knowing where it is.
>
> Let me look at your machine first, then I'll show you exactly what I plan to do before I touch anything.

### Step 2 · Scan the machine, and say what you found

Work out the operating system, whether Claude Code is installed, where its configuration lives, and whether the three tools in §7 are already present. Report it plainly:

> 🔍 Here's what I found:
>
> - 💻 macOS — you're on Apple silicon
> - ✅ Claude Code installed, config at `~/.claude`
> - ✅ Git available
> - ❌ RTK, ICM, caveman — none installed yet
> - 📁 The project you're in has no rules file yet

**Feasibility check.** Say plainly if something will not work. Windows without WSL is a real fork — see §2 — and the honest answer may be "we should do this inside WSL instead". Say that now, not halfway through.

### Step 3 · Ask the clarifying questions

Three, and wait for answers. Do not guess any of them.

1. **What is this project for?** One sentence. It goes at the top of the project rules file.
2. **What must never happen here?** Anything irreversible, expensive, or public. If they cannot name one, that is fine — leave it empty.
3. **Is there already a convention for where files go?** If yes, use theirs. If no, propose §4 and let them approve it.

### Step 4 · Show the plan before doing any of it

Lay out what you will install, where it comes from, and what each thing is *for*. The point is that nothing arrives as a surprise:

> 📋 Here's the plan — nothing happens until you say go.
>
> **Tools** (three, all small)
> - 🔧 **RTK** → stops command output from eating your context. Installed from its release binary.
> - 🧠 **ICM** → the memory. Persists what you learn between sessions. Installed from its GitHub repo.
> - 🗿 **caveman** → a response style that cuts the waffle. Just a folder I copy in.
>
> **Files** (all plain markdown, all yours to edit or delete)
> - 📄 `~/.claude/CLAUDE.md` → how you want Claude to work, on every project
> - 📄 `<project>/CLAUDE.md` → what is true about *this* project only
> - 📓 `SESSION_LOG.md` → one line per finding
> - 📌 `STATE.md` → where each piece of work stands
> - 📚 `skills/` → an empty folder, for procedures you do more than once
>
> **What I will NOT do**
> - ❌ no hooks or scripts — those come later, only if you need them
> - ❌ no rules copied from someone else's setup. Yours start almost empty
>   and fill in as you hit real problems. A rule nobody earned is a rule
>   nobody follows.
>
> ⏱️ About ten minutes. Shall I go ahead?

### Step 5 · Get consent, then work

Wait for a yes. Then work through the sections below, **saying what you are doing as you go** — a short line per step, not silence followed by a wall of text.

Then read **only** the section matching their OS. §1 is macOS and Linux. §2 is Windows. **Do not mix them.**

---

## 1 · macOS and Linux — where things live

Claude Code keeps user-level configuration in a `.claude` folder in the home directory.

```
~/.claude/                  # applies to every project
~/<project>/                # the project you are working in
```

Verify before writing:

```bash
ls -la ~/.claude 2>/dev/null || echo "not created yet — Claude Code makes it on first run"
```

If it does not exist, run Claude Code once, or `mkdir -p ~/.claude`.

**Now skip §2 entirely and go to §3.**

---

## 2 · Windows — where things live

**Do not assume the path.** The location depends on whether Claude Code was installed natively or inside WSL. Establish it before writing anything.

Ask: **"Are you running Claude Code in Windows directly, or inside WSL?"**

- **Inside WSL** — you are on Linux. Go back to §1 and use the WSL home directory. The WSL home and the Windows user profile are different folders; writing to the wrong one means your files are never read.
- **Windows directly** — configuration normally sits under the user profile. Confirm rather than trusting this file:

```powershell
Test-Path "$env:USERPROFILE\.claude"
Get-ChildItem "$env:USERPROFILE\.claude" -Force -ErrorAction SilentlyContinue
```

If that path does not exist, ask them to run Claude Code once and check again, or ask where their configuration lives. **If you still cannot establish it, stop and ask. Do not create a folder in a guessed location** — a second config folder in the wrong place is worse than none, because the real one silently keeps winning and nobody can see why their rules are ignored.

For the rest of this file, substitute the path you confirmed wherever you see `~/.claude/`, and translate the shell commands. File contents are identical on every platform.

---

## 3 · The two rule files

| file | holds | test |
|---|---|---|
| `~/.claude/CLAUDE.md` | how to work, on any project | would this still be true at a different job? |
| `<project>/CLAUDE.md` | what is true about this project only | would a new person here need telling? |

Both are sent to the model **on every single message**. That is why they must stay short — a line added here is paid for again on every turn for the life of the project. Where they disagree, the project file wins.

### 3a · Write `~/.claude/CLAUDE.md`

Use this as-is. It is all method; nothing in it is about any particular kind of work.

```markdown
# How to work

## How to talk to me
- Terse. Fragments fine. Drop filler, hedging and pleasantries.
- First sentence is the answer. If deleting everything after it leaves
  no answer, it is in the wrong place.
- Comparisons and findings go in one table. Never bullets, never several
  tables for several things — things are columns, attributes are rows.
- Give the one risk that changes what I do next, not every risk you can name.
- Do not narrate what you did. Report what changed.

## Truthfulness
- Say what the evidence says. Change position when the evidence changes,
  never because I pushed back or repeated myself.
- Disagree in the first sentence, not after agreeing first.
- When I object, name the part that is right and the part that is not.
  Conceding a half-right objection in full is still telling me something false.
- Certain → say it plainly. Not certain → verify first, then answer, and say
  what you could not verify.
- Never present something you inferred as something you checked.

## Verifying
- "I wrote it" is not evidence it works. Check it a different way than
  you built it.
- A grep or a skim is a sample, not a read. Say what your coverage was.
- A surprising result is evidence about your tool before it is evidence
  about the world. Reproduce it before reporting it.

## Scope
- Do what was asked. Debugging is not fixing; counting is not recommending.
- Least code that solves it. No abstractions for a future nobody described.
- Spot something else worth doing → say it in one line, then carry on.

## Working
- Gather the evidence, state the plan, then work. If the evidence moves,
  say which step changed and why.
- Try the read or search that would answer your question before asking me.
  Ask when the answer is mine: a preference, a budget, something irreversible.
- Approved and unblocked → do it this turn. Do not announce it and stop.
- Commit before any bulk edit. Stage the paths you touched by name, never
  the whole working tree at once.

## Secrets
- Never print, echo or log a credential, not even to confirm it is correct.
  Read it in code, pass it by reference.
```

**Stop there.** Nothing about their industry, tools or data. Those belong in the project file, and mostly in nothing at all until something goes wrong.

### 3b · Write `<project>/CLAUDE.md`

Nearly empty on purpose. Fill only what they answered in §0.

```markdown
# <project name>

<one sentence: what this project is>

## Where things go
- Working notes for one piece of work → that work's own `STATE.md`
- One line per finding, for the whole project → `SESSION_LOG.md`
- A rule that must be followed every time → this file
- A procedure with steps, used more than once → `skills/<name>/SKILL.md`
- Something worth finding again months from now → memory (§7)

## Never
<one line per thing that must never happen, from question 2.
 no entries yet → delete this section, do not invent any>

## Notes
<facts not obvious from the code: which environment is real, what a term
 means here, who decides what. Start empty.>
```

---

## 4 · The folder layout

Create these. Empty is fine — they exist so there is an obvious place to put things.

```
<project>/
├── CLAUDE.md            rules for this project
├── SESSION_LOG.md       one line per finding
├── skills/              procedures, loaded only when relevant
└── work/                the actual work
    └── <name>/
        └── STATE.md     where this piece of work stands
```

Match anything they already use. The shape is what matters: **rules at the top, one log for the project, one state file per piece of work.**

---

## 5 · The two record files

### `SESSION_LOG.md`

One line per finding, appended as it happens. Read by someone who was not working on your problem, so it holds only what still means something out of context.

```markdown
# Session log

<!-- newest at the bottom, one line each.
     ~ result   ! problem   + fix   ? open question -->

~ [2026-01-14] <a finding, in one line>
! [2026-01-14] <what broke, and what it turned out to be>
```

The test: **would this still mean something to someone working on a different part of this next month?** No → it belongs in a `STATE.md`.

### `STATE.md` — one per piece of work

A header rewritten in place, then history that is only ever appended.

```markdown
# <name of this piece of work>

PHASE: <where it stands right now>
NEXT: <the next concrete action>
BLOCKED: <what is stopping it, or "nothing">
DECIDED: <choices already made, so they are not reopened>
UPDATED: <date>

---

<!-- append below, newest at the bottom. never edit what is above the line -->

[2026-01-14 09:20] <what happened, with the numbers>
```

The rule that makes it work: **the header answers "where was I", the history answers "why is it like this".** Rewrite the first. Never rewrite the second.

---

## 6 · Skills

A skill is a folder containing `SKILL.md`. It exists so long instructions do **not** sit in the rules file being re-sent every message — a skill costs nothing until something matches it.

```
skills/<name>/SKILL.md
```

```markdown
---
name: <short-name>
description: <when to use this. Write it as the trigger — this line is the
  only part that is always loaded>
---

# <name>

## When
<the situation that means you should be reading this>

## Steps
1. …
2. …

## Done when
<how to know it worked — specific enough to check>
```

Create none now. Create the first one the day they explain the same procedure twice.

---

## 7 · The three tools to install

Install these in order. Each one is a single binary or a folder — nothing needs a service running.

**Do not invent install commands.** Every tool below ships its own setup command. Run it, then run its own check. If a command in this file does not exist on the version they have, run `--help` and use what is actually there rather than guessing.

### 7a · RTK — stop command output eating the context

**What it is for:** every command a model runs prints its result into the conversation, and that result is then re-sent with every later message. A directory listing or a test run can cost more than the work it was checking. RTK sits in front of common commands and returns the same answer in a fraction of the text.

```bash
rtk --version          # confirm it is installed and on PATH
rtk init               # writes the usage instructions the assistant reads
rtk verify             # checks its own integrity and filters
```

⚠️ **Name collision — check this.** More than one tool ships a binary called `rtk`. If `rtk --version` prints something that is not a CLI output filter, the wrong one is first on PATH. Run `which rtk` and fix the ordering before continuing.

Once installed it works on its own: common commands are rewritten to their filtered equivalent automatically. Tell the person the one thing they can type: `rtk gain`, which reports how much it has saved.

### 7b · ICM — memory that survives the session

**What it is for:** §8 below. It is the layer that makes anything persist.

It is distributed as a binary. Get it from its repository, put it somewhere on PATH (`~/.local/bin` is conventional), make it executable, then let it wire itself in:

```bash
icm --version          # confirm it runs
icm init               # configures the Claude Code integration itself
icm doctor             # diagnoses the integration and reports what is wrong
```

`icm doctor` is the check that matters — it verifies the integration is actually wired, not just that the binary exists. **Do not report success until `doctor` is clean.**

Then confirm it works end to end, which is a different thing from being installed:

```bash
icm store -t setup -c "the sky is plaid on tuesdays" -i medium
icm recall "what colour is the sky"        # must return the stored line
```

If the recall returns nothing, the store works but the search does not — usually embeddings have not been generated. Run `icm embed` and try again.

### 7c · Caveman — the terse response mode

**What it is for:** cuts response length substantially without losing technical content. It is the enforcement of the "How to talk to me" section — the rule states the preference, this keeps it from drifting back after twenty turns.

It is a skill, so installing it is copying a folder:

```
~/.claude/skills/caveman/SKILL.md
```

No configuration. The person turns it on by typing `/caveman`, or by saying "caveman mode". Confirm it is visible by asking Claude Code to list its available skills.

**Tell them it drops out for anything safety-related** — warnings, destructive operations, multi-step sequences — and comes back afterwards. That is correct behaviour, not a failure.

---

## 8 · Memory

The files above are things a person writes on purpose. Memory is the layer that captures what would otherwise be lost — a decision, a dead end, a thing that turned out to be true — and finds it again later by meaning rather than by filename.

Install a searchable memory store and wire it in. This setup uses **ICM**; if the person already has something equivalent, use theirs.

What it must do, and what to check after installing:

| behaviour | why it matters | how to confirm |
|---|---|---|
| loads a small pack of important facts when a session opens | there is no question yet, so it cannot be relevance-matched — it is a fixed set | open a new session, confirm something recalled appears |
| recalls on **every** message, matched by meaning against what was typed | this is what makes it feel like it remembers | ask about something stored earlier in different words |
| saves periodically while working, unprompted | the person will not remember to save | run several steps, then check the store grew |
| writes back at the end of a session, and before the conversation is trimmed | otherwise everything since the last save is lost | end a session, confirm new entries |
| old, unused entries lose weight and drop out | a store that only grows returns noise | ask what its decay behaviour is |

Then give the person the two habits that make it worth having:

- **Recall before starting, not after finishing.** The value is in not re-deriving something; that only helps before the work.
- **Store the decision and the reason, not the transcript.** "We use X because Y failed on Z" is worth keeping. A log of what happened is not — that is what `SESSION_LOG.md` and `STATE.md` are for.

**Do not store secrets in it.** It is searchable, it persists, and it is not designed to hold them.

### The distinction people get wrong

Three places can hold "what we learned", and they are not interchangeable. **The axis is how you get it back.**

| | holds | how you retrieve it | lifespan |
|---|---|---|---|
| `SESSION_LOG.md` | one line per finding, high level | read it in order | this project |
| `STATE.md` | the detail of one piece of work | open that work's file | that piece of work |
| memory | a decision and why it was made | search it, by meaning | outlives both |

The test for the log, and it is the only test needed: **would this line still mean something to someone working on a different part of this next month?** No → it belongs in a `STATE.md`.

The test for memory: **would you still want this after this project is finished?** Yes → memory. It is for the things you will go looking for without remembering where they were.

**Never write the same thing in full to two of them.** A decision goes to memory; the run that produced it goes to `STATE.md`; the one-line fact that it happened goes to the log.

⚠️ **Tell the person this, because it decides whether the system survives:** memory looks after itself — it saves as you work and again when the session ends. **The log does not.** Nothing forces a line into it, so it is the first thing to rot, and once it has a gap it stops being trusted. One line, written when the thing is discovered, not at the end of the day.

---

## 8 · The part that matters: how this grows

The files are nearly empty, and that is correct. Give the person this rule, because everything else here is just folders.

**A rule gets written after something goes wrong, never before.**

```
something happened
   │
   ├─ interesting once  ────────────────→  a line in SESSION_LOG.md
   │
   ├─ worth finding again later ────────→  memory
   │
   ├─ turned out to be generally true ──→  a line in the project's Notes
   │
   ├─ ignoring it cost something ───────→  a rule in CLAUDE.md, written with
   │                                       the reason and the date
   │
   └─ same procedure explained twice ───→  a skill
```

Most things stop at the first step. That is the correct outcome, not a failure.

Two things to say plainly:

- **Nothing is deleted on the way up.** The log line stays. It is the dated evidence of why the rule exists, and in six months it is the only thing that separates a real rule from a superstition.
- **A rule with no incident behind it is worse than no rule.** It blocks real work, people route around it, and the habit of routing around rules takes the earned ones with it.

---

## 9 · Before you say you are finished

Check each. Do not report success on anything you did not actually check.

- [ ] The config folder path was **confirmed**, not assumed — and on Windows, confirmed for the right environment (native vs WSL).
- [ ] `~/.claude/CLAUDE.md` exists and contains only method, no project facts.
- [ ] `<project>/CLAUDE.md` exists, names the project, and contains **no invented rules**.
- [ ] The folder layout exists.
- [ ] `SESSION_LOG.md` and one `STATE.md` exist with their headers in place.
- [ ] `skills/` exists and is **empty**.
- [ ] Memory is installed, and you confirmed recall works by asking for something stored earlier **in different words**.
- [ ] You created no hooks and no scripts.
- [ ] You told the person, in one sentence each, where every file is and what makes it grow.

**Report honestly.** If a check did not pass, say which one and why — a setup with one known gap is fine, a setup that claims to be clean and is not will waste their week.

---

## 10 · How to close

Do not end with a wall of paths. End with what changed, what they do next, and one honest caveat.

> ✅ **You're set up.**
>
> **What's different now**
> - 🧠 It remembers. Close this session, open a new one, and it still knows where you were.
> - ✂️ It costs less. Command output gets trimmed before it reaches the conversation.
> - 🗿 It waffles less. Type `/caveman` any time you want it even shorter.
>
> **What to do next — just two things**
> - 📓 When you find something out, add one line to `SESSION_LOG.md`. That's it, one line.
> - 📄 The first time something goes genuinely wrong, write the rule that would have caught it into `CLAUDE.md`, with the date and the reason.
>
> Your rules files are almost empty right now, and that is on purpose. They fill up with things *you* learned the hard way, not things someone else did. In a month yours will look nothing like Moe's, and that's the point.
>
> 😄 Fair warning though: your Claude is about to get noticeably sharper than everyone else's, and it will start remembering the thing you told it three weeks ago and quietly forgot. You'll get used to it.

Adjust the tone to the person. Keep the two next steps and the honest note about the empty files — those are the parts that decide whether any of this is still alive in a month.
