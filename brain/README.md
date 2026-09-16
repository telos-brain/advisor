# Advisor brain (Sol)

This is a **Telos Brain** that acts as an advisor to *your* AI agents — Claude Cowork, Grokbot, Cursor, and anything else that can call MCP tools.

The advisor's name is **Sol**. Treat Sol like a human advisor: brief them, give them the craft they need, then use them when you are reviewing work and decisions. Sol is not a chatbot and not a second pair of hands on the keyboard. Sol holds memory, skills, and judgment, and answers when the calling agent asks.

This README is written for an **AI agent** that will set the brain up with a human. Do not invent API keys. Collect each secret from the user, or from the signup API response, then continue.

---

## What Sol does

Once deployed and connected over MCP, the calling agent can:

| Situation | Tool |
|---|---|
| Information worth remembering (notes, facts, preferences, decisions, current context) | `briefing` |
| Current or external information that should also be remembered | `research` |
| A new problem that needs framing before a solution | `create_frame_of_reference` |
| A plan or piece of thinking that needs a quality check | `ask_sol` |
| One factual question against stored memory | `ask_question` |
| A transferable practice to apply | `find_available_skills` then `get_skill` |

Memory holds situation-specific knowledge. Skills hold transferable practices. Do not confuse the two.

The brain already ships skill books for advisory craft, decision making, and business. You can add more (see [Working with Sol](#working-with-sol)).

---

## The name "Sol"

**Sol** is the advisor's persona — how the brain introduces itself to calling agents.

To rename Sol, edit these files and redeploy. Search the `brain/` folder for `Sol` so you do not miss a line.

| File | What to change |
|---|---|
| `workflows/system-prompt.md` | Persona: "You are Sol…" and the tone line that mentions Sol. This is shared by most advisor workflows. |
| `workflows/advisor.md` | MCP workflow `name`, description, and the instructions that say "You are Sol". This is what calling agents see. |
| `workflows/ask-for-advice.md` | Workflow title **Ask Sol** and any Sol wording in the description. |
| `tools/execution/advisor/ask-sol.yml` | Tool name `ask_sol` and description. If you rename the tool, also update the `tools:` list on `advisor.md` and `chat.md`. |
| `workflows/sol-research.md` | Opening line ("You are Sol researching…"). |
| `tools/execution/advisor/research.yml` / `briefing.yml` | Descriptions that say "Ask Sol…". |

The brain's product name in `brain-compose.yml` is **Advisor**. That is the instance label in the Telos Brain UI, not the persona. Change `name:` there only if you want a different brain title.

---

## Prerequisites the user must provide

Stop and ask the user for these. Do not skip ahead.

### 1. Telos Brain organisation and API key

Sign the user up through the public Management API.

1. Ask for an **account name** (organisation display name), their **full name**, and the **email** that should receive the invite.
2. Ask them to accept the Telos Brain terms and conditions. When they agree, send `termsAndConditions: true`.
3. Call the public signup endpoint once:

   ```bash
   curl -sS -X POST https://go.telosbrain.com/organisations/signup \
     -H "Content-Type: application/json" \
     -d '{
       "accountName": "<account name>",
       "personName": "<full name>",
       "email": "<email>",
       "termsAndConditions": true
     }'
   ```

4. A `201` response looks like `{ "organisationId": "…", "apiKey": "tbk_…" }`. Put `apiKey` in `brain/.env` as `TELOS_BRAIN_ORG_API_KEY` and ask the user to store it in a password manager. The key is shown **once**.
5. Tell the user to accept the invite email and sign in at **https://go.telosbrain.com**. That activates the organisation and grants **$10** welcome credit. Deploy can proceed with the returned key; workflow runs start once the organisation is Active.

Cloud deploy talks to `https://go.telosbrain.com` by default (`TELOS_BRAIN_API_URL` in `.env.example`).

### 2. An LLM key (Claude or Grok)

Sol's workflows need a model key or they will not run. Ask the user which they have, then collect **one**:

| Provider | Where to get a key | `.env` variable |
|---|---|---|
| Claude (Anthropic) | https://console.anthropic.com | `ANTHROPIC_API_KEY` |
| Grok (xAI) | https://console.x.ai | `XAI_API_KEY` |

Starter workflows pin Anthropic (`anthropic/claude-sonnet-4-6`). If the user only has a Grok key, set `XAI_API_KEY` **and** point the brain at an xAI model, for example in `.env`:

```
DEFAULT_LLM_MODEL=xai/grok-4-5
```

or ask them to set **Default LLM model** in the brain Settings after first deploy. A reachable brain default overrides the workflow pins.

Optional later:

- `VOYAGE_API_KEY` — https://dash.voyageai.com — semantic search (`voyage-3-lite`). Deploy works without it; skill and memory search will be weaker.
- `OPENAI_API_KEY` / `OPENROUTER_API_KEY` — only if you switch models to those providers.

---

## Deploy to the cloud (default)

Work from the `brain/` directory.

1. Install the CLI if it is missing:

   ```bash
   npm install -g @telos.ready/brain
   ```

2. Copy `.env.example` to `.env` (if `.env` does not already exist).

3. Fill in:

   ```
   TELOS_BRAIN_ORG_API_KEY=<key from signup, or the user's existing org key>
   TELOS_BRAIN_API_URL=https://go.telosbrain.com
   ANTHROPIC_API_KEY=<their Claude key>
   ```

   or `XAI_API_KEY` plus `DEFAULT_LLM_MODEL` as above. Leave unused key lines blank.

4. Deploy:

   ```bash
   brain deploy --instance advisor
   ```

5. **Capture the Brain API key from stdout immediately** on first deploy. It is printed **once**. Tell the user to store it in a password manager. Do not commit it. Do not delete `brain.lock`.

6. If you change model keys or `DEFAULT_LLM_MODEL` later, redeploy the same command so the brain stores the new values.

**Redeploy tip:** if a later deploy hits HTTP 409, run `brain snapshot` first so live version numbers come back to disk.

---

## Connect via MCP

After a successful cloud deploy:

1. Tell the user to sign in at **https://go.telosbrain.com**.
2. Open this brain (instance **advisor**, title **Advisor**).
3. Open **Workflows**. Find the MCP workflow named **Sol** (`WF-ADVISOR`).
4. Copy the **MCP URL** shown on that workflow. That is the URL the calling agent uses.
5. In Claude Cowork, Grokbot, Cursor, or the host they use, add an MCP server with that URL and complete authorisation (OAuth on the hosted MCP, or the organisation API key if the client asks for a bearer token).

After connecting, the calling agent should see Sol's tools (`briefing`, `research`, `create_frame_of_reference`, `ask_sol`, `ask_question`, `find_available_skills`, `get_skill`).

If tools do not appear, have the user toggle the MCP server off and on in the client so the tool list refreshes.

---

## Working with Sol

Use Sol the way you would use a human advisor.

**Start by briefing.** Call `briefing` with anything Sol should remember on later turns: how the user works, current projects, decisions already made, notes from books, constraints. Do not wait until you need advice. An unbriefed advisor is guessing.

**Give Sol craft when you have it.** This brain already has Advisory, Decision Making, and Business skill books. If the user has specific practices their agents should follow (how *this* company decides, writes, sells, or ships), create a skill book under `brain/skills/` and deploy it. One skill per file; a `skillbook.yml` lists them. Situation-specific facts still go through `briefing`, not into skills.

**Use Sol when reviewing work and decisions.** When the calling agent has a plan, a design, or a choice:

1. Frame a new problem with `create_frame_of_reference` *before* locking a solution.
2. Check a proposed approach with `ask_sol`.
3. Prefer Sol's reply over inventing your own critique.

Keep your own messages short. Put the substance in the tool arguments.

---

## Deploy locally (optional)

Use this only when the user wants a Brain stack on their machine. Cloud is the default.

Requirements: Node.js 25+, Docker.

```bash
npm install -g @telos.ready/brain
cd brain
brain start
```

`brain start` writes `.env.local` (if missing), starts SQL Server and the Brain server in Docker, and opens the admin UI at **http://127.0.0.1:60061** (no sign-in). It uses a well-known local organisation key that **must not** be used in production.

Put the Claude or Grok key in `.env.local` (same variable names as cloud). Then:

```bash
brain deploy --env local --instance advisor
```

Find the Sol MCP URL on the local workflows page the same way as in the cloud. Point the MCP client at that local URL. Localhost does not use hosted OAuth the same way — the client should send the local organisation API key if asked.

```bash
brain status
brain stop --project-id <id-from-status>
```

Full local-stack detail: skill **BRA106** (`skills/telos-brain/concepts/BRA106-local-development.md`).

---

## Repository hygiene

Do not commit:

- `.env`, `.env.local`
- `brain.lock` (if it contains keys)
- `node_modules/`, `dist/`

Commit `.env.example` with placeholders only. Never store org keys, LLM keys, or the Brain API key in git.

---

## Support

Copyright Telos IP Limited 2026  
https://www.telosbrain.com  
support@telosbrain.com
