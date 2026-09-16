---
name: Briefing
code: WF-BRIEFING
description: >-
  Takes valued information and files it into advisor memory. Searches for an
  existing entry first, then updates or creates. Use to store notes from books,
  conversations, facts, preferences, decisions and context.
version: 1

# TOOL: invoked via tools/execution/advisor/briefing.yml as {{input.content}}.
type: TOOL

system-prompt-code: WF-SYSTEM-PROMPT

output-tokens: 2048, 4096
caching: automatic
max-turns: 16
max-runs-per-hour: 200
session-timeout: 15

tools:
  - search_blueprint_entries
  - list_blueprint_entries
  - get_blueprint_entry
  - add_blueprint_entry
  - update_blueprint_entry
---

# Instructions

You are filing a **briefing** into this advisor's working memory. Distil what
is worth remembering. Do not store a transcript. Do not invent facts that are
not in the briefing.

## Briefing

<briefing>
{{input.content}}
</briefing>

## Memory categories

File each distinct concept under exactly one category:

| Category | Use for |
|---|---|
| Facts | Stable true things about people, organisations, products, markets |
| Context | What is happening now — live work, recent events |
| Preferences | Values, constraints, working style, non-negotiables |
| Goals | Desired outcomes and why they matter |
| Decisions | Commitments already made and the rationale |
| Insights | Distilled reusable observations we now own |
| Sources | Notes from books, articles, conversations — attribute the source |
| Open questions | Unresolved questions worth returning to |

Transferable practices (how to do something next time) do **not** belong here —
skip those and mention them in the reply so they can become skills later.

## Process

1. Read the briefing. Split it into **distinct concepts** (usually 1–5). A
   book dump may yield a Sources note plus one or two Insights. A status update
   is usually Context. A stated constraint is Preferences.
2. For **each** concept:
   1. Choose the category.
   2. Call `search_blueprint_entries` with a query for that concept (and the
      category when you are confident). If results are thin, try
      `list_blueprint_entries` for that category.
   3. If a hit is the **same concept** (not merely the same topic area), call
      `get_blueprint_entry` and then **`update_blueprint_entry` once**: merge
      the new information, preserve what is already there, and keep
      `old_str` unique.
   4. If there is **no** close match, call **`add_blueprint_entry` once** with
      a short unique title and concise markdown.
3. Prefer update over create when the concept already exists. Never create a
   duplicate title. Never dump the raw briefing unchanged.
4. Reply in a short list: one line per write — created or updated, category,
   and title. If nothing was worth storing, say so in one line.

## Rules

- Always search before writing.
- One write per concept. Several concepts in one briefing is fine.
- Attribute Sources (book, author, conversation) in the entry body.
- British English. No preamble, no tool commentary.
