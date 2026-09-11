---
name: Advisor
code: WF-ADVISOR
description: >-
  Advisor MCP server. Exposes briefing, problem framing, advice, memory
  questions, and skill lookup so a calling agent can consult this brain.
version: 1

# MCP: published as an MCP server. Injected tools become the MCP tool list.
# Instructions become the MCP prompt / resource for the calling agent.
type: MCP

max-runs-per-hour: 200

tools:
  - find_available_skills
  - get_skill
  - briefing
  - create_frame_of_reference
  - ask_for_advice
  - ask_question
---

# Instructions

You are consulting an advisor. Call the tools. Do not treat this as a chatbot.

## When to call what

| Situation | Tool |
|---|---|
| You have information worth remembering (book notes, a fact, a preference, a decision, current context) | `briefing` |
| You are first facing a problem or challenge and need it framed — what this is really about, the bigger picture, things to consider | `create_frame_of_reference` |
| You already have a plan or solution and want a quality check | `ask_for_advice` |
| You need one factual answer from stored memory | `ask_question` |
| You want to apply a practice yourself | `find_available_skills` then `get_skill` |

## How to use them

- **`briefing`** — pass the information as `content`. The advisor files it in memory (search first, then update or create). Do not pre-categorise.
- **`create_frame_of_reference`** — pass the problem as `context`. Use this *before* you lock a solution. The reply is a problem statement, frame, domain model, bigger picture, and considerations.
- **`ask_for_advice`** — pass the proposed thinking as `request`. Expect a short list of guidance (gaps, things to consider), not a rewrite. Call this when you have something to check, not when you are still defining the problem.
- **`ask_question`** — one self-contained question against memory. Not for open-ended advice.
- **`find_available_skills`** — search by what you need. Stubs only. Load a skill with **`get_skill`** before applying it.

## Rules

- Frame first (`create_frame_of_reference`), advise second (`ask_for_advice`).
- Brief anything you want the advisor to remember on later calls.
- Prefer the advisor's reply over inventing your own critique.
- Keep your own messages short. Put the substance in the tool arguments.
