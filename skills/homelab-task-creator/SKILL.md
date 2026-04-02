---
name: homelab-task-creator
description: >
  Create a new Obsidian task note in the Personal vault under Projekt/Homelab/tasks/ when the user describes something they want to do, research, build, or install. Use this skill whenever the user says things like "create a task", "add a task", "I want to set up X", "remind me to research Y", "make a note to install Z", "log this as a task", "add this to my homelab tasks", or just describes a piece of work they want to track. Triggers on any new task, to-do, project idea, or research item the user wants to capture in their Obsidian vault. Don't wait for the user to say "Obsidian" explicitly — if they describe work to be tracked, use this skill.
---
 
# Homelab Task Creator
 
Creates a structured task note in the user's Obsidian **Personal** vault at `Projekt/Homelab/tasks/`.
 
## What to do
 
Given the user's description, you will:
1. Derive a short filename (1–8 words) from the core intent
2. Decide on the metadata fields (status, category, priority, project, tags, links)
3. Write a rich but concise description, suggest useful resources, and open a blank Notes/Log section with today's date
4. Create the note using the Obsidian CLI
 
## Step 1 — Derive the filename
 
The filename = the `title` value in the frontmatter. Keep it short (1–8 words), written in title case, capturing exactly what the task is. No date prefix, no numbering.
 
Good examples:
- `Set Up SMTP Relay with Postfix`
- `Research Self-Hosted Password Manager`
- `Install Tailscale on Home Server`
- `Configure Automated Backups`
 
## Step 2 — Fill in the frontmatter fields
 
Use good judgment to fill these in from context clues in the user's message. When unsure, prefer sensible defaults.
 
| Field | Values | Default |
|---|---|---|
| `status` | `To do` · `In Progress` · `On Hold` · `Done` | `To do` |
| `category` | `Task` · `Planning` · `Research` · `Reference` | `Task` (use `Research` if user says "look into", "explore", "figure out") |
| `priority` | `Low` · `Medium` · `High` · `Critical` | `Low` |
| `project` | `Homelab` · `Email` · `Media` | `Homelab` |
| `tags` | list of lowercase strings | Infer relevant tags from technology names, tool names, concepts, and action type — be generous, more tags are fine as long as they're all relevant and non-duplicate |
| `links` | list of URL strings | Extract any URLs the user mentions; otherwise omit or leave empty list |
 
**Priority guidance:** Default is always `Low` — only raise it when there's a clear signal:
- `Low` — no urgency mentioned, just something to get to eventually (this is the default)
- `Medium` — user mentions a deadline, "soon", or mild urgency
- `High` — user says "need to fix", "blocking", or something is degraded but still partially working
- `Critical` — something is completely down or broken right now ("not starting", "stopped working", "ASAP", active outage)
 
**Project guidance:**
- `Email` → anything involving mail servers, SMTP, IMAP, spam filtering, DNS mail records
- `Media` → media servers, streaming, Plex/Jellyfin, downloads, transcoding
- `Homelab` → everything else (networking, containers, VMs, monitoring, storage, auth, etc.)
 
## Step 3 — Write the note body
 
After the frontmatter, write three sections:
 
### Description
A concise paragraph (2–5 sentences) explaining what this task involves and what "done" looks like. Include any relevant constraints or context the user mentioned.
 
### Resources
A short bullet list of useful starting points — official docs, relevant GitHub repos, Docker images, blog posts, or tools. Use your knowledge to suggest 2–5 genuinely useful resources even if the user didn't mention any. Format them as Markdown links where possible. Keep it focused — resist the urge to add checklists, sub-sections, or investigation steps here. The note should be a starting point, not a project plan.
 
### Notes / Log
A single dated entry using today's date (format: `YYYY-MM-DD`), briefly capturing how the task was initiated. One line, factual. This section will grow as the user works on the task — don't pre-fill it with checklists, sub-sections, or planned steps. That would be presumptuous and clutters the log before any real work has started.
 
```
## Notes / Log
 
- 2026-04-02: Task created — [one-line summary of intent]
```
 
## Step 4 — Create the note with the Obsidian CLI
 
Construct the full note content (frontmatter + body) and create it using:
 
```bash
obsidian vault="Personal" create path="Projekt/Homelab/tasks/<Title>.md" content="<escaped content>" silent
```
 
**Critical:** The `content` parameter uses `\n` for newlines. Construct the content string carefully — YAML frontmatter must use proper `\n` indentation, and the `---` delimiters must be on their own lines. For the YAML list fields (`tags`, `links`), each item needs `\n  - value`.
 
**Practical approach** — to avoid shell escaping nightmares with complex YAML, write the note to a temp file first, then use `obsidian eval` to read it into the vault:
 
```bash
# Write content to temp file
cat > /tmp/homelab_task.md << 'EOF'
---
title: Set Up SMTP Relay with Postfix
status: To do
category: Task
priority: Low
project: Email
tags:
  - postfix
  - smtp
  - docker
links:
  - https://www.postfix.org/BASIC_CONFIGURATION_README.html
---
 
## Description
 
Configure Postfix as an outbound SMTP relay...
 
## Resources
 
- [Postfix documentation](https://www.postfix.org)
 
## Notes / Log
 
- 2026-04-02: Task created — set up outbound SMTP relay with TLS and DKIM support
EOF
 
# Create the note in Obsidian via eval
obsidian vault="Personal" eval code="
const content = require('fs').readFileSync('/tmp/homelab_task.md', 'utf8');
await app.vault.create('Projekt/Homelab/tasks/Set Up SMTP Relay with Postfix.md', content);
'done'
"
```
 
If the note already exists, `vault.create` will throw. In that case, use `vault.modify` on the existing file instead, or prompt the user to confirm overwrite.
 
## Step 5 — Confirm to the user
 
After creating the note, tell the user:
- The note name and vault path
- The key metadata (status, priority, project)
- A one-line summary of what was captured
 
Keep it brief. No need to repeat the full note content.
 
## Full template for reference
 
```markdown
---
title: <Short Title>
status: To do
category: Task
priority: Low
project: Homelab
tags:
  - tag1
  - tag2
links:
  - https://example.com
---
 
## Description
 
<2-5 sentences describing the task and what done looks like.>
 
## Resources
 
- [Resource name](https://url)
- [Another resource](https://url)
 
## Notes / Log
 
- YYYY-MM-DD: Task created — <one-line intent summary>
```
