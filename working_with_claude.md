# Working With Claude — Field Guide

Written 2026-09-23 in the "Claude feedback" admin/training chat, after a day of cleanup and a
rules session. The rules themselves live in `C:\Users\mattc\.claude\CLAUDE.md` (the rulebook).
This file is how Matt works the tool. Companion: `chat_starter_template.md`.

---

## 1. Reading a Permission Prompt

Source: code.claude.com/docs/en/permissions, plus experience.

A prompt shows the tool, the command (or file + changes), and Claude's description.
**Read the command, not the description.** The description is a claim; the command is what runs.

**5-second check:**
1. **Read or change?** Reads (`git status`, `Get-Content`, `ls`, Read, Grep) are low risk.
   Changes (Edit, Write, delete, move, restart, push) deserve a real look.
2. **Where does it point?**

   | Address | What it is |
   |---|---|
   | `10.0.0.89` | Printer (Octopussy) |
   | `10.0.0.90` / `100.126.228.3` | AllStar node |
   | `10.0.0.112` / `100.75.2.114` | OptiPlex |
   | `ssh` anything | A command running on another machine |

3. **Danger words:** `rm`, `Remove-Item`, `del`, `sed -i`, `systemctl restart/stop`, `reboot`,
   `shutdown`, `Stop-Process`, `Set-Service`, `git push --force`, `git reset --hard`,
   any M-code/G-code, `curl` with `-X POST` or `-d` (sends a command, not just a read).
4. **Is it a chain?** `;` `&&` `|` join commands. Read every piece; the risky part hides at the end.
5. **Did Claude explain it first** (what + risk + source)? If not, click No and ask.

**Buttons:** never click "Yes, and don't ask again." No is cheap; a wrong yes cost a day.
Outward-facing steps (create repo, push, send) should arrive in their own prompt, never chained.

Quiz lessons (2026-09-23): a read-only check with `&& systemctl restart` tacked on = No.
`Remove-Item ...\* -Recurse -Force` described as "clean up old files" = No (skips Recycle Bin,
deletes everything, no explanation given).

---

## 2. Starting a Project Chat

Full template: `chat_starter_template.md`.

A new chat reads only: the rulebook, the folder's CLAUDE.md + project notes, and the memory
INDEX. Not individual memory files, not other projects' notes, not other chats. All project chats
likely open in the Ham Radio folder, so point Claude at the right notes.

Six lines: project + goal, current state, what changed, files to read, limits, what "done" means.
End with: **"Tell me where things stand and your plan before you do anything."**

Before working in any repo: `git status -sb`. `behind` = pull first (phone sessions push to
GitHub only). `ahead` = local work not pushed yet.

---

## 3. When to Start a Fresh Chat

Long chats get **compacted**: the older part is squeezed into a summary and details drop out
(that's how a failed print got restarted). **Files never compact**: notes, memory, git stay exact.

Start fresh when:
1. The topic changes (one chat per project).
2. A milestone is reached and saved.
3. Hardware work is coming and the chat is already long.
4. Claude starts slipping: repeats a question, circles back, contradicts a decision.
5. A compaction just happened during hardware work.

Don't start fresh mid-task with unsaved state. Say "wrap up" first.

**Switch cleanly:** "wrap up" → approve notes/memory/commit → new chat → paste starter template →
check Claude's first reply.

---

## 4. Git Undo

Sources: git-scm.com/docs/git-restore, git-scm.com/docs/git-revert.

**Look first (read-only, always safe):**
- `git status`: which files changed
- `git diff`: exactly what changed, not yet committed
- `git log --oneline`: commit list with short IDs

**Undo:**

| Situation | Command | Note |
|---|---|---|
| File messed up, not committed | `git restore <file>` | Uncommitted edits to that file are gone |
| Want a file from an older commit | `git restore --source <id>~1 <file>` | Overwrites current file; commit after |
| A whole commit was wrong | `git revert <id>` | New commit reverses it; history kept; needs clean tree. Then push. |

**Say No to:** `git reset --hard` (throws changes away, no undo) and `git push --force`
(overwrites GitHub; would have wiped the phone session's DVSwitch docs).

**Repos (all private on GitHub):**

| Repo | Folder |
|---|---|
| ham-radio-ai-assistant | `Desktop\Ham Radio\` |
| ham-shack-references | `Desktop\Ham Shack\` |
| allstar-node | `Desktop\AllStar Node\` |
| satellite-rotator | `Desktop\Satellite Rotator\` |
| claude-config | `C:\Users\mattc\.claude\` (rules, memory, skills) |

---

## 5. Framing Questions

From experience, not a manual. Claude answers what's in front of it; a narrow question gets a
narrow answer, and a guess stated as fact gets built on.

1. **Goal + what changed, not just the step.** "Fine on stock plate, swapped to PEI, first layer
   drags" beats "what M851 should I use?"
2. **Say what you saw; label theories.** "No TX audio. I *think* Rigblaster, not sure. Pwr slider
   at -29."
3. **Paste exact text.** Error messages and readings, not paraphrases.
4. **Plan before action.** "What would you check, in order? Don't run anything yet."
5. **Ask confidence.** "How sure are you, and what's your source?"
6. **Ask for other ways.** "Is this the only way?" (the Da Vinci $35 board swap lesson)

| When | Say |
|---|---|
| Starting a problem | "It worked until ___ changed. Here's what I see: ___." |
| Before any action | "Plan first, don't run anything." |
| Answer sounds too smooth | "Source?" |
| Going in circles | "Stop. What do we actually know for sure?" |
| Big procedure coming | "Other ways to get there?" |
