# TODO

## Merge `refactor` into `main`
- Local `main` holds an unpushed merge of `refactor` (`f5f9a1c`), but it is behind `origin/main` by one commit:
  `7c23626` "Remove the Hugging Face and Gradio voices" (it edits the old copied library that `refactor` deletes).
- Steps: `git fetch`, `git checkout main`, `git merge refactor` (if not already in), then `git merge origin/main`.
- Expected conflicts. Keep the `refactor` side for all of them:
  - `README.md`, `uv.lock`: `git checkout --ours README.md uv.lock`
  - `waxal_agent/engines.py`, `scripts/check_api.py`, `tests/test_pipeline.py` (deleted on `refactor`): `git rm` them
- Check: `uv run pytest` (5 tests), no `waxal_agent/` folder left, then `git push origin main`.
- Afterwards, delete the `refactor` branch (local and on GitHub).

## Settings
- Consider `AGENT_MEMORY_MODEL=claude-haiku-5-5` in `.env`: the notes saved after each answer would cost far less.
- `.env` still has an unused `HF_TOKEN=` line (the Hugging Face voice was removed).
- `WAXAL_TOKEN` is empty in `.env`: set it before the server is reachable from other machines.

## DeepSeek (when the key is available)
- Run the two-question cache test on `deepseek-flash` (`DEEPSEEK_API_KEY`) and compare the cost and the answers with Claude.
- With DeepSeek the agent has no `web_search` and reads PDFs in text mode only.
