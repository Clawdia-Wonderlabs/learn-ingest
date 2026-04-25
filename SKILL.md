---
name: learn-ingest
description: >
  Guided intake pipeline for learning materials — transcripts, course lessons, Drive folders, PDFs.
  ALWAYS trigger on `/ingest-material`. Also triggers when you paste a transcript, drops a file path,
  shares a Google Drive link to course content, or says anything like "ingest this", "add this to my vault",
  "I just watched a course on X", "save this lesson", "process this transcript", or "what should I do
  with this course content". Triggers on Skool lessons, YouTube transcripts, VTT files, and any
  structured learning content. When in doubt, trigger this skill.
---

# learn-ingest

Guided intake for any learning material → vault → action pipeline.

---

## Vault Structure

All course content lives here:
```
~/Your Vault/Courses/<Community>/<Course Name>/<Lesson>.md
```

Mining extracts:
```
~/Your Vault/Courses/<Community>/<Course Name>/Mining/<lesson>-<type>.md
```

The **Community** layer is new — it's the platform or group the course came from (e.g. "YourCommunity", "Skool", "Standalone", "YouTube"). Always ask for it when the material is course content.

---

## Intake Flow

This skill uses a guided intake conversation. Work through these steps in order. Don't front-load all the questions — ask one thing at a time, wait for the answer, then proceed.

### Step A: Material Type

If invoked via `/ingest-material` with no arguments, ask:

> "What type of material is this?
> - Course lesson / module
> - Workshop or webinar recording
> - Podcast / interview transcript
> - Article or guide (text)
> - Other"

If the type is obvious from context (e.g. you pasted a transcript), skip this and proceed.

---

### Step B: Community (course material only)

If the material is a **course lesson**, ask:

> "Which community or platform is this from? (e.g. YourCommunity, Skool, standalone purchase, YouTube, etc.)"

This becomes the top-level folder under `Courses/`.

If it's not course material (podcast, article, etc.), use a sensible default like the source platform name.

---

### Step C: Source

Point to the point to the source — or detect it from context:

- **Pasted text**: use directly
- **Local file path** (`.txt`, `.md`, `.vtt`, `.srt`): read the file
- **Google Drive link**: use the Google Drive MCP to list and fetch the content (see Drive section below)
- **Multiple files / a whole folder**: ask which specific lesson(s) to ingest, or offer "ingest all"

---

### Step D: Action Selection (multi-select)

After you have the source content in hand, ALWAYS present the full action menu exactly as shown below — including the plain-language description for each option. Never abbreviate it. you want the reminder of what each does every time.

Present it as checkboxes. Point to the reply with the letters she wants (e.g. "A C E") or say "all":

---
**What do you want to do with this?**
*Check all that apply — reply with the letters, or say "all"*

- [ ] **A — Insights** — I read it and tell you what matters: key ideas, frameworks, and what it actually changes about how you'd work. Good for when you just watched something and want the "so what."

- [ ] **B — Implement** — You want to build what's being taught. I pull out the actual steps and either kick off a GSD project or draft a phase for an existing one. Good for tutorials where there's something to deploy or configure.

- [ ] **C — Content mining** — I extract the raw material for creating content: frameworks you could teach, quotes worth reusing, and 5 specific post/article angles inspired by the lesson. Saved to your vault as a mining note.

- [ ] **D — Conversation mining** — Similar to content mining but focused on *how to talk about it*: opening hooks, punchy talking points, contrarian angles, and story frames. More useful for social posts, DMs, and sales conversations than long-form writing.

- [ ] **E — Skill extraction** — I look at the lesson and ask: "Could this process become a Claude skill?" If yes, I write up what the skill would do, what it'd be called, and how complex it'd be — and offer to build it on the spot with `/skill-creator`.

- [ ] **F — Teach me** — Instead of a summary, we have a back-and-forth. I introduce one concept at a time, use examples from your actual work, and ask questions to check understanding. Good for dense material you want to actually retain, not just file away.

- [ ] **G — Just archive** — Save it to the vault, nothing else. You want it there for reference but don't need to do anything with it right now.

---

If you say "all", run A through F in sequence (skip G since archiving is implicit when anything else is selected).

---

## Ingest: Read & Save to Vault

Before running any actions, always save to vault first.

**Gather:** course name, lesson name, community (from intake above).

**Classify content type:**

| Type | Signals |
|------|---------|
| **tutorial** | step-by-step, "first X then Y", code/config walkthrough |
| **concept** | framework or principle explanation, no specific how-to |
| **tool-walkthrough** | demos a specific product, narrated UI tour |
| **strategy** | positioning, decision-making, business/growth advice |
| **interview** | Q&A format, guest insights |

**For VTT/SRT**: strip all timestamp lines (`00:00:01.000 --> 00:00:05.000`) before processing.

**For long transcripts (>2000 words):** Write a structured summary (300–500 words) + Key Quotes section. Don't dump the raw wall of text.

**For short transcripts (<2000 words):** Include full text under `## Transcript`.

Use Python to write files (never heredocs/echo):

```python
import os
from datetime import date

vault_base = os.path.expanduser("~/Your Vault/Courses")
lesson_dir = os.path.join(vault_base, "<Community>", "<Course Name>")
os.makedirs(lesson_dir, exist_ok=True)

content = """---
title: <Lesson Name>
course: <Course Name>
community: <Community>
date: <today YYYY-MM-DD>
type: <tutorial|concept|tool-walkthrough|strategy|interview>
tags: [courses, <community-slug>, <course-slug>, <type>]
source: <file path, Drive URL, or "pasted">
---

## Summary

<structured summary>

## Key Quotes

- "<quote>"

## Transcript

<full text or omitted if long>
"""

with open(os.path.join(lesson_dir, "<lesson-slug>.md"), "w") as f:
    f.write(content)
```

Confirm: `"Saved to 3.Resources/Courses/<Community>/<Course Name>/<lesson-slug>.md"`

---

## Actions

### [A] Insights

Read the transcript carefully and deliver a structured debrief:

```markdown
## Key Insights — <Lesson Name>

### The Big Idea
<one-paragraph distillation of the core point>

### Frameworks & Models
<any named systems, formulas, or mental models>

### Surprising or Counterintuitive Points
<what challenges assumptions>

### What This Changes
<practical implication — what should you do differently after this?>
```

### [B] Implement

Ask: "Is this a new project, or does it fit an existing one?"

- **New** → "Run `/gsd-new-project` and use this as the requirements source." Extract a numbered implementation checklist from the transcript (actual steps, not paraphrases).
- **Existing** → "Run `/gsd-add-phase` or `/gsd-insert-phase`." Write a concise phase description based on the content.

### [C] Content Mining

Save to `Mining/<lesson-slug>-content-mining.md`:

```markdown
---
title: Content Mining — <Lesson Name>
source: <vault path>
date: <today>
tags: [mining, content, <course-slug>]
---

## Core Frameworks
## Key Concepts
## Quotable Lines
## Content Angles (5 ideas)
```

### [D] Conversation Mining

Save to `Mining/<lesson-slug>-convo-mining.md`:

```markdown
---
title: Conversation Mining — <Lesson Name>
source: <vault path>
date: <today>
tags: [mining, conversation, <course-slug>]
---

## Hooks & Opening Lines
## Talking Points
## Controversy / Tension Points
## Story Angles
## Audience Pain Points Addressed
```

### [E] Skill Extraction

Analyze the content through the lens of: "Could this process, workflow, or framework become a Claude skill?"

Deliver:

```markdown
## Skill Extraction Analysis — <Lesson Name>

### Automation Potential
<Is there a repeatable workflow here Claude could execute?>

### Proposed Skill Name & Trigger
<What would it be called? When would it fire?>

### What the Skill Would Do
<Step-by-step description of the skill's behavior>

### Complexity Estimate
<Simple (1 step) / Medium (3–5 steps) / Complex (multi-agent)>

### Recommendation
<Build it now / Add to backlog / Not worth automating — why>
```

If the recommendation is "build it now", offer: "Want me to start building this with `/skill-creator`?"

### [F] Teach Me

Socratic walkthrough — don't lecture:
1. 2-sentence overview of what the lesson covers
2. Present the first key concept with a concrete example from your actual work and projects
3. Ask: "Does this connect to anything you're working on, or want to go deeper here?"
4. Let you's response guide depth and direction

### [G] Just Archive

Confirm: "Archived at `3.Resources/Courses/<Community>/<Course>/<lesson>.md`. Nothing else to do."

---

## Reading from Google Drive

When you provide a Drive folder or file link:

1. Use `mcp__google__drive_list` to list contents of the folder
2. Present the file list: "Found X files — which would you like to ingest? Or say 'all'."
3. For each selected file, use `mcp__google__docs_read` (for Docs) or appropriate tool to fetch content
4. If a file is a video/audio recording (MP4, MOV), note: "This is a video file — do you have a transcript for it, or should I skip it?"
5. PDFs and text files: read and process normally

When reading from the any Skool or online community Drive folder, look for:
- Module/lesson folders (numbered or named)
- Transcript files (`.vtt`, `.txt`, `.md`, `.pdf`)
- Slide decks (note them but skip unless you want them ingested separately)

---

## Edge Cases

- **Multiple lessons in one session**: offer to process each as a separate vault note, or merge into one
- **Duplicate vault path**: warn — "A file already exists at that path — overwrite, rename, or skip?"
- **No clear lesson name in content**: use filename or ask
- **Drive file is a video with no transcript**: flag it and ask if you want to source a transcript separately
