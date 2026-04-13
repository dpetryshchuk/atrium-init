Initialize a new atrium. Run this once when setting up for the first time.

Usage: /atr-setup

Your atrium is a curated, public-facing knowledge layer that lives as `atrium/` inside your wiki.
Trusted contacts query it with their agents. Run this from your wiki root.

---

## Step 0 — Mode selection

Ask the user:
```
How do you want to run setup?

  1. Guided   — I'll walk you through each step and ask before proceeding
  2. Fast     — I'll ask everything upfront and run setup in one shot

(1 or 2)
```

Store their choice as `{mode}`. Use it throughout:
- **Guided**: ask one question at a time, confirm before each major step, explain what's happening
- **Fast**: ask all questions in a single message, run all steps without pausing for confirmation

---

## Step 0b — Prerequisites check

Tell the user:
"Atrium uses the GitHub CLI (gh) to create your repo, manage access, and fetch connections' knowledge pages securely. Checking your setup now."

Check if gh is installed: run `gh --version`

**If not installed:**
Tell the user: "gh CLI isn't installed — installing it now."
Run the appropriate command for their platform:
  - macOS:   `brew install gh`
  - Linux:   `sudo apt install gh` (or `sudo dnf install gh` / `sudo pacman -S gh`)
  - Windows: `winget install --id GitHub.cli --silent --accept-package-agreements --accept-source-agreements`
Verify: run `gh --version` again. If still not found: tell them to install manually at cli.github.com and stop.
On Windows: warn the user "Restart your terminal after install if gh commands don't work — or use the full path: C:\Program Files\GitHub CLI\gh.exe"

**If installed:** run `gh auth status`
- Authenticated: continue to Step 1.
- Not authenticated:
  Tell the user: "gh needs to connect to your GitHub account — opening browser login now. You'll see a one-time code to enter."
  Run: `gh auth login --web --hostname github.com`
  Wait for completion. Verify with `gh auth status`. If still not authenticated: stop and tell the user to run `gh auth login --web --hostname github.com` manually then re-run /atr-setup.
  Note: gh requests repo-level access. This is required to create your private atrium repo, send collaborator invites, and read connections' private atriums.

---

## Step 1 — Confirm location

**Guided**: tell the user "I'll create your atrium at ./atrium inside this wiki. Continue? (yes/no)" and wait.
**Fast**: proceed without asking.

---

## Step 2 — Discovery + Questions

**Guided**: ask "To speed up setup, I can read your wiki index and existing pages to pre-fill your atrium. OK to look around? (yes/no)" and wait for reply.
**Fast**: look around automatically without asking.

If looking around (either mode):
- Check for `wiki/index.md`, `index.md`, `wiki/overview.md`, `overview.md` at the wiki root
- If found: read them. Extract name, GitHub handle, professional background, current focus, notable projects/skills
- **Guided**: show a draft calling_card and 2-sentence overview, ask the user to confirm or correct. Wait for reply.
- **Fast**: use what you found directly; skip showing the draft.

If nothing found, ask all questions:
- **Guided**: ask one at a time, wait for each reply:
  Q1: "What's your name and GitHub handle?"
  Q2: "What's your professional background in one sentence?"
  Q3: "What topics or skills would someone find in your atrium?"
  Q4: "What are you currently working on or looking for?"
- **Fast**: ask all four in a single message, wait for one reply with all answers.

---

## Step 3 — Surface content to publish

**Guided**: list any project, skill, or framework pages found in the wiki. Ask: "Which of these do you want in your atrium? I'll rewrite them for public audience — no personal content." Wait for reply. Write only confirmed pages.
**Fast**: skip this step. Create empty folder stubs and move on. (User can add pages later by running /atr-setup again or editing atrium/ directly.)

For each confirmed page: write a sanitized version into `atrium/projects/`, `atrium/skills/`, or `atrium/frameworks/`
- Strip: personal details, financial specifics, private people, internal deliberations
- Keep: what the thing is, how it works, what someone could learn from it

---

## Step 4 — Copy template and fill in

Copy all files from `template/` into `./atrium/`:
- AGENT.md, index.md, atlas.json, overview.md
- projects/, skills/, frameworks/ (create if not already written above)

If any file already exists at destination: confirm before overwriting.

Fill in using confirmed answers:
- `AGENT.md` — replace `[Your Name]`, `[your-github-handle]`, `[date]`
- `overview.md` — write professional overview (2–3 paragraphs; no personal content)
- `index.md` — fill calling_card frontmatter; update title, date, and page catalog to match what was published
- `atlas.json` — set owner.name, owner.handle, owner.atrium (`https://github.com/{handle}/atrium`); leave connections as [], last_synced as null
- `calling_card.json` — fill from confirmed answers: handle (string), current (string, one sentence), topics (array of 3–6 strings), updated_at (current UTC timestamp, ISO 8601)

---

## Step 5 — Git init

```bash
cd ./atrium && git init && git add . && git commit -m "initial: atrium setup"
```
If git not installed: skip, warn, continue.
If .git already exists: skip init, just `git add . && git commit`.

---

## Step 6 — Create and push GitHub repo

```bash
cd ./atrium && gh repo create atrium --private --source=. --push
```
If gh is not installed or not authenticated: skip and show manual fallback:
"Create a private GitHub repo at github.com/{handle}/atrium and push: `git remote add origin [url] && git push -u origin main`"

---

## Step 7 — Install commands

Copy atr- commands into the wiki so they're available in every Claude Code session:
```bash
mkdir -p ../.claude/commands && cp ..atrium-init/.claude/commands/atr-*.md ../.claude/commands/
```
Adjust the source path to wherever atrium-init was cloned. If this fails: skip silently, note it below.

---

## Step 8 — Done + Tutorial

Tell the user:
```
Your atrium is live at github.com/{handle}/atrium
atr- commands installed — available whenever you open this wiki in Claude Code.
```
If step 6 was skipped: show manual push instructions.
If step 7 failed: "Manual install: copy atrium-init/.claude/commands/atr-*.md into .claude/commands/ at your wiki root."

Then walk them through how the network works — send three messages, one at a time, waiting for a reply between each:

---

**Message 1 — Inviting someone:**
```
Here's how to bring someone into your network:

Send them this:
  https://github.com/dpetryshchuk/atrium-init

They clone it, run /atr-setup in their wiki, and their atrium goes live at github.com/{their-handle}/atrium.

Once they're set up, you connect with:
  /atr-add [their GitHub handle]

That sends them a GitHub invite to read your atrium. They'll do the same back.

Who do you want to connect with first?
```

---

**Message 2 — After they reply (or say they'll do it later):**
```
Once both sides have accepted GitHub invites, run:
  /atr-sync

This checks all your pending connections, pulls their calling cards, and marks them active.
A calling card is a compact summary of what's in someone's atrium — skills, topics, what they're working on.
Your agent uses it to route questions without reading their whole wiki.

Want to try adding a connection now, or move on?
```

---

**Message 3 — Closing:**
```
To query someone's atrium:
  /atr-ask [handle] "how do you approach X?"

Or ask across everyone:
  /atr-ask "who has experience with X?"

Your agent reads their index, pulls the relevant pages, and synthesizes — all inside Claude Code, no extra infrastructure.

You're set up. Open this wiki any time and the atr- commands are ready.
```
