# Compilation Methodology

How to compile an article into high-quality, connected knowledge. Based on Luhmann's Zettelkasten method.

## Core Principle

You are a **thinker**, not an extractor. Don't summarize what the article says. Write your OWN thoughts triggered by the reading. "The article says X" is worthless. "X contradicts what we know about Y, which suggests Z" is valuable.

## Process

### 1. Fetch the article

```bash
lens fetch <url> --save --json
```

This returns `source_id` and `markdown`. Read the article fully before proceeding.

### 2. Search existing knowledge

```bash
lens search "key concept 1" --json
lens search "key concept 2" --json
```

Search 2-4 key concepts from the article. For each result found:

```bash
lens show <id> --json
```

Read the full note. Understand what you already know. This step is critical — the value of compilation comes from **collision** between new ideas and existing knowledge.

### 3. Think: for each candidate idea from the article

Ask yourself:

- **Does it duplicate an existing note?** → Don't create. Optionally update the existing note with new evidence.
- **Does it support an existing note?** → Update the existing note (add evidence, strengthen qualifier). Only create a new note if the support itself is an independent insight.
- **Does it contradict an existing note?** → THIS is the most valuable outcome. Create a new note with a `contradicts` link. Explain the tension.
- **Does it refine an existing note?** → Update or create with `refines` link.
- **Is it genuinely new?** → Create a new note. Link it to related existing notes.
- **Is it common knowledge?** → Skip entirely.

### 4. Write

For updates to existing notes:
```bash
echo '{"type":"update","id":"note_EXISTING","add":{"evidence":[{"text":"supporting quote","source":"src_NEW"}]},"set":{"qualifier":"likely"}}' | lens write --json
```

For new notes:
```bash
echo '{"type":"note","text":"Your concept-oriented insight","role":"claim","qualifier":"likely","source":"src_NEW","contradicts":["note_EXISTING"]}' | lens write --json
```

## Quality Rules

**1. Concept-oriented, not source-oriented.** Bad: "Notes from Fowler's article." Good: "High internal quality has negative cost in software." The note must stand alone without naming its source article.

**2. Rewrite in your own words.** Don't copy passages. Reformulate the idea. Use `voice: "synthesized"` for your thinking. Use `"extracted"` only for verbatim quotes inside `evidence[]`.

**3. One idea per note.** If a note contains multiple claims, split it. Each note should express exactly one independent thought.

**4. Every link needs a reason.** Don't link because topics are vaguely related. Loose associations are noise. Ask: "why does this note SPECIFICALLY support/contradict/refine that one?"

**5. Update before create.** If a new article provides evidence for an existing claim, update it — don't create a duplicate. The knowledge graph should grow in depth, not just in breadth.

**6. Zero new notes is acceptable.** An article covering familiar ground should produce updates to existing notes, not new ones. The number follows from genuine thinking, not from a quota.

**7. Contradictions are the most valuable output.** When you find a tension between a new idea and an existing note, that IS the insight. Don't smooth over disagreements.

**8. Scope awareness.** Mark notes as `big_picture` (core insight) or `detail` (supporting point). Big-picture notes supported by detail notes form natural hierarchy through links — no folders needed.

## Anti-Patterns

- Creating a note for every paragraph of the article
- Notes that are rephrased sentences from the article (extraction, not thinking)
- Notes that only make sense if you know which article they came from
- Orphan notes with no links to existing knowledge
- Using `supports` everywhere because it's safe (look harder for contradictions)
- Creating a `structure_note` per article (structure notes are post-hoc, sparse index entries created after clusters form naturally)
