# IslamChat

An assistant on Islam in Wolof. A person sends a voice note in Wolof, and the answer comes back as a Wolof voice note and as text. It answers
only from trusted sources: the allowed websites and the books in the library. It never answers from memory and never gives a fatwa.

IslamChat is a thin layer over the **WaxalAgent library** (`../WaxalAgent`: the core `waxal_agent` and the web layer `waxal_server`). All
the code is there: the pipeline (speech recognition, translation, agent, voice), the per-person agent, S3 sync, the test page's server, the
WhatsApp webhook, links and browsing. **The library's README is the reference** for how all of that works and for every setting. This
folder only holds what makes the app its own:

| Path | What it is |
|---|---|
| `islamchat/` | the `islamchat` command (the library's server with this app's name) and the app's page, `static/index.html` |
| `data/instructions/INSTRUCTIONS.md` | the agent's general instructions: sources in order (websites, then library), no fatwa, name the source |
| `data/skills/` | the skills: `answer-from-the-sources`, `teach-the-basics`, `explain-a-text`, `personal-question` |
| `library/` | the books the agent answers from: the Noble Quran (French), Sahih al-Bukhari, Mukhtasar al-Akhdari (git-ignored) |
| `example.env` | the settings, with this app's values (`WAXAL_LINK_DOMAINS`, `WAXAL_MAX_WEB_OPEN`) |
| `docs/` | `INSTRUCTIONS.example.md` and `skills/`: the committed copies of the instructions and skills (`data/` is git-ignored), `SANDBOX.md` |
| `tests/test_app.py` | checks that the page, the command and the content fit the library |

`pyproject.toml` reads the library from the folder next to this one (`[tool.uv.sources]`, editable: a change in `../WaxalAgent` is seen at
once). For a release or Docker, pin a git commit
(`waxal-agent[server] @ git+https://github.com/DemePS/WaxalAgent.git@<commit>`; the Dockerfile already installs the library from git).

## Try it

```bash
uv sync
cp example.env .env                       # every setting, with DEVELOPER_MODE=1; fill in the keys
set -a; source .env; set +a               # the server reads the environment, not the file
uv run islamchat serve                    # http://127.0.0.1:8000/?token=<WAXAL_TOKEN>
```

Needs `ANTHROPIC_API_KEY` and `ELEVENLABS_API_KEY`, and ffmpeg. `DEVELOPER_MODE=1` keeps S3 and WhatsApp off, so only the test page is served.
Optional extra: `uv sync --extra browser` (the agent browses the allowed sites; then `playwright install chromium`). `WHATSAPP.md` has the commands to test on WhatsApp, `docs/SANDBOX.md` the Docker sandbox.

## What is specific to this app

- **Trusted sources.** `WAXAL_LINK_DOMAINS=islamqa.info=IslamQA, doctrine-malikite.fr=Doctrine Malikite`: the agent browses these sites
  (at most `WAXAL_MAX_WEB_OPEN` pages per question) and may share a link to them, shown under the answer and never spoken. With this list set,
  Anthropic's hosted web search is off, so the agent cannot read the rest of the web.
- **Library.** The books are long. The library's base prompt makes the agent find the pages with `search_pdf` and open only
  those pages.
- **Skills.** Loaded on demand: the base method, step-by-step lessons, explaining a verse, a hadith or a term, and sensitive personal questions
  (general knowledge, no personal ruling, refer to a qualified scholar).
- **Page.** `islamchat/static/index.html` is the library's test page with this app's name; it calls the library's `/api/...` routes, which the
  tests check.

To change the agent's behaviour, edit the instructions or a skill (no code). To change anything else (the pipeline, WhatsApp, S3, browsing),
change `../WaxalAgent`, not this folder.

## Tests

```bash
uv run pytest        # this app's tests; the library's own tests are in ../WaxalAgent
```
