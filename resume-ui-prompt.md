# Task: Add a friendly web UI to `resume-gen`

Builds on the existing `resume_gen.py` + `resume.json`. Reuse its render and AI functions. Do not rewrite them.

## Agent rules (save tokens)
- Be terse. Code only, no explanations, no recap. Final reply: `done`.
- Read only `resume_gen.py` function signatures (grep `def `), not the whole file.
- Write each file once, complete. No refactors, no tests, no extra docs.
- No build tools, no npm, no frameworks, no CDN. Vanilla HTML/CSS/JS.
- Ask nothing. Pick the simplest option when unclear.

## Files
1. `ui.html`: the whole front end (HTML + CSS + JS inline), about 400 lines max.
2. `resume_gen.py`: add a `serve` command only (about 40 lines), using `http.server` from the stdlib.
   - `python resume_gen.py serve` starts on `127.0.0.1:8765` and opens the browser automatically.
   - Routes: `GET /` returns `ui.html`; `POST /api/render` takes resume JSON + template and returns HTML; `POST /api/ai` takes resume JSON + job text and returns the improved JSON (reuses the cached single-call AI function); `GET/POST /api/settings` reads and saves the API key, base URL and model to `.settings.json` (the key is never returned in full, only the last 4 characters).

## Layout
- **Desktop:** two columns. Left = editor (45%), right = live preview (55%, A4 paper look, sticky).
- **Mobile (<800px):** single column with a bottom tab bar: `Edit | Preview | AI`.
- Top bar: app name, template picker (Classic / Modern), `Download PDF`, `...` menu (Import JSON, Export JSON, Reset, Settings).
- Preview updates live, debounced 300ms, by calling `/api/render` and setting an `iframe srcdoc`.

## Editor (friendly by design)
- Collapsible section cards in this order: Basics, Summary, Experience, Education, Skills, Projects. The first card is open and the rest are collapsed. Each header shows a completeness check mark.
- Progress bar at the top: "4 of 6 sections done".
- Add/remove entries with big, clear buttons (`+ Add job`). Arrow buttons move entries up and down (no drag-and-drop).
- Bullets: one textarea per job, one bullet per line. Show a live character count and a soft warning at 120 characters ("Shorter reads better").
- Skills: chip input. Press Enter or comma to add a chip, click x to remove it.
- Every field has a label and a short placeholder example (e.g. "Led a team of 4 to cut load time by 35%").
- Month pickers for dates, plus a "Present" checkbox.
- Inline validation: a friendly message beside the field (e.g. "Add an email so recruiters can reach you"). Never block the user or show alert popups.

## AI panel (opt-in, transparent)
- A collapsible card: "Tailor to a job (AI)". It has a textarea ("Paste the job description") and a `Tailor my resume` button.
- Before sending, show a one-line estimate: "Sends ~N keywords, 1 request." Cache hits show "Used cache, free".
- Result is shown as a **diff review**: original vs suggested text per bullet, with `Accept` / `Reject` buttons per item and `Accept all`. Nothing changes until the user accepts.
- Loading state: the button turns into a spinner with "Working... (max 20s)". On failure, show a plain-language message and keep the user's data intact.
- If no API key is set, the button opens Settings with a 3-line guide to getting a free key (OpenRouter / Groq / Gemini).
- Settings dialog: the API key field is masked, with a `Test connection` button that shows a green or red status.

## Data safety
- Autosave to `localStorage` on every change (debounced 500ms). Show a subtle "Saved" indicator.
- On load, if saved data exists, restore it silently. If not, load the sample with a banner: "Sample data loaded. Replace it with yours."
- Undo/Redo (Ctrl+Z / Ctrl+Y) via a 30-step in-memory history of JSON snapshots.
- Reset requires a confirm dialog. Import validates the JSON shape and reports errors in plain words.

## Visual design
- Clean and calm: system font stack, 16px base, 8px spacing grid, one accent color (indigo), generous whitespace, 12px rounded cards, soft shadows.
- CSS variables for colors. Auto light/dark through `prefers-color-scheme`.
- Preview stays on white paper in both themes.
- Buttons: a clear primary (`Download PDF`), others secondary. Minimum 44px tap targets.
- Subtle transitions (150ms) only. Respect `prefers-reduced-motion`.

## Accessibility & polish
- Semantic HTML, real `<label>` elements, visible focus rings, full keyboard use, `aria-live="polite"` for the Saved and AI status messages.
- Contrast of at least 4.5:1. No color-only meaning.
- Empty states with guidance ("No jobs yet. Add your most recent one first.").
- Keyboard shortcuts: `Ctrl+S` forces a save, `Ctrl+P` downloads the PDF (via `window.print()` on the preview iframe).

## Done when
- `python resume_gen.py serve` opens a working UI, and editing any field updates the preview.
- The resume prints cleanly to one A4 page.
- It works offline with `--ai` unused, and the AI flow works with a free key.

Start now. Create/modify the files, then reply with only: `done`.
