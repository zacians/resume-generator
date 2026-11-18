# resume-gen
Offline resume builder; optional one AI call to tailor bullets/summary. Python 3.9+, no deps.

    python resume_gen.py init
    python resume_gen.py build
    python resume_gen.py build --job job.txt --ai --template modern --out cv

Edit `resume.json` (single source of truth). Open the HTML and print to PDF (A4, styled); `weasyprint`/`wkhtmltopdf` are used if installed.

Web UI (`python resume_gen.py serve`): add a photo, or import an old PDF resume from the menu. PDF import uses one AI call and needs selectable text; scanned/image PDFs need OCR first. Review every imported field.
AI is opt-in (`--ai --job`), cached in `.cache/`, falls back to your text on any error, 20s timeout.
Env: `AI_BASE_URL` (default https://openrouter.ai/api/v1), `AI_API_KEY`, `AI_MODEL` (default meta-llama/llama-3.3-70b-instruct:free).
Any OpenAI-compatible free tier works: OpenRouter `:free` models, Groq (https://api.groq.com/openai/v1), Gemini (https://generativelanguage.googleapis.com/v1beta/openai).

PDF import can use a separate provider. Set all three of `AI_SCAN_BASE_URL`, `AI_SCAN_API_KEY`, `AI_SCAN_MODEL`; if any is missing it uses the tailoring provider. The same fields are in `.settings.json` (`scan_base_url`, `scan_api_key`, `scan_model`) and in the Settings dialog.
Example: Gemini for tailoring, Mistral for PDF import:

    AI_BASE_URL=https://generativelanguage.googleapis.com/v1beta/openai
    AI_MODEL=gemini-2.5-flash
    AI_API_KEY=<google-ai-studio-key>
    AI_SCAN_BASE_URL=https://api.mistral.ai/v1
    AI_SCAN_MODEL=open-mistral-nemo
    AI_SCAN_API_KEY=<mistral-key>

Test either side with `python resume_gen.py check-ai` or `python resume_gen.py check-ai --scan`.

## Share it over Tailscale

The app has no built-in accounts, so set a password before exposing it beyond localhost:

    $env:RESUME_PASSWORD = 'choose-a-strong-password'
    python resume_gen.py serve --no-browser

Login user defaults to `resume`; change it with `--user`. `--password` also works, but an env var keeps the password out of the process list.

Pick one: tailnet-only HTTPS (private) or public Funnel.

    # private, only your Tailscale devices
    tailscale serve --bg 8765
    tailscale serve status

    # public internet
    tailscale funnel --bg 8765
    tailscale funnel status

Both proxy `https://<machine>.<tailnet>.ts.net/` to `http://127.0.0.1:8765`.

**Do not use Funnel without `RESUME_PASSWORD`.** Anyone with the URL could read/edit your resume, change AI settings, and spend your AI quota. Disable Funnel with:

    tailscale funnel --https=443 off

The local UI is still at `http://127.0.0.1:8765/`.
