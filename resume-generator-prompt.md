# Task: Build `resume-gen` (CLI, AI-assisted, free-tier friendly)

## Agent rules (follow strictly, save tokens)
- Be terse. No explanations, no summaries, no recap after finishing. Only code + one-line status.
- Do NOT read files you don't need. Do NOT re-read files you just wrote.
- Write each file once, complete. Avoid rewrites and refactors.
- No extra features, no tests, no docs beyond a 10-line README.
- Ask nothing. If unclear, pick the simplest option.

## Stack
- Python 3.9+, **stdlib only** (urllib, json, argparse, pathlib, hashlib). No pip installs.
- One file: `resume_gen.py` (~250 lines max) + `resume.json` (sample data) + `README.md`.

## Flow
1. `resume.json` = single source of truth (name, contact, summary, experience[], education[], skills[], projects[]).
2. `python resume_gen.py init` -> writes sample `resume.json`.
3. `python resume_gen.py build [--job job.txt] [--ai] [--template classic|modern] [--out resume]`
   - Renders `resume.html` (print-ready CSS, A4, `@media print`) and `resume.md`.
   - User exports PDF via browser print (or `wkhtmltopdf`/`weasyprint` if installed; auto-detect, never require).
4. Templates = Python string templates inside the file (2 small ones, pure CSS, no JS, no external fonts).

## AI usage (this is where tokens are saved)
Use AI **only** for: (a) rewriting experience bullets to match a job description, (b) writing a 2-sentence summary. Everything else (layout, sorting, formatting) is local code.

Token-saving requirements:
- **One AI call per build**, not per section. Send a single compact JSON payload, expect a single compact JSON reply.
- Send only fields AI must change (`summary`, `bullets`) plus job keywords. Never send contact info, dates, education.
- Pre-trim job description locally: strip boilerplate, keep top ~25 keywords/phrases (frequency-based, stopword list in code) -> send keywords, not the full text.
- Cap input: max 6 bullets per role, 3 most recent roles. Truncate bullets to 200 chars.
- Cap output: `max_tokens=700`, `temperature=0.3`.
- Terse system prompt (below). No chat history, no few-shot examples.
- **Cache**: key = sha256(payload) -> `.cache/<hash>.json`. Identical input = zero API calls.
- `--ai` is opt-in. Without it, build is fully offline and free.
- On API failure/timeout (20s): fall back to original text silently, print one warning line.

### System prompt to embed (exact)
```
Resume editor. Rewrite given bullets/summary for the job keywords. Rules: action verb first, quantify only if numbers exist in input, never invent facts, max 18 words/bullet, ATS-friendly plain text. Reply JSON only: {"summary":"","roles":[["bullet",...],...]}
```

### Provider config (free tiers)
OpenAI-compatible endpoint via env vars, so any free provider works:
- `AI_BASE_URL` (default `https://openrouter.ai/api/v1`)
- `AI_API_KEY`
- `AI_MODEL` (default a `:free` model, e.g. `meta-llama/llama-3.3-70b-instruct:free`)
- Examples in README: OpenRouter free models, Groq free tier, Gemini OpenAI-compatible endpoint.
- Single function `ai_call(payload) -> dict` using `urllib.request`. Strip code fences before `json.loads`.

## Output quality (local, no AI)
- One page target: if content overflows, auto-drop oldest bullets beyond the cap, then reduce font via CSS variable.
- Sections ordered: Header, Summary, Experience, Projects, Skills, Education.
- Escape HTML. Dates as given. No images, no icons.
- Skills: put skills matching job keywords first.

## Deliverables
1. `resume_gen.py`
2. `resume.json` (sample, fake person)
3. `README.md` (usage + env vars, max 10 lines)

Start now. Create the files, then reply with only: `done`.
