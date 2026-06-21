# hermes-plugin-teams-voice

Microsoft Teams **voice/video (Conversational Video Interface)** for **Hermes Agent**,
packaged as a standalone, pip-installable plugin — install it *on top of* a normal
Hermes install, no fork required.

The plugin (name **`teams_voice`**) hosts the HMAC-authenticated WebSocket bridge that
a media worker dials into, and drives the call: realtime (OpenAI/Azure
speech-to-speech) **or** streaming (STT→agent→TTS), camera/screen vision, the avatar
driver cues (expression / visemes / show-to-caller), group-call etiquette, DTMF,
bilingual EN/AR, meeting recap/minutes, and SharePoint (OneDrive) file send.

## Install on Hermes

Install into the **same Python environment as Hermes** — it discovers the plugin via
the `hermes_agent.plugins` entry-point and imports it in-process. Target the python
that runs `hermes` (Linux/macOS `…/venv/bin/python`, Windows `…\venv\Scripts\python.exe`),
or activate that venv first and drop `--python`.

**A — from PyPI (recommended):**

```bash
uv pip install --python /path/to/hermes/venv/bin/python hermes-plugin-teams-voice
# or, with the Hermes venv activated:  pip install hermes-plugin-teams-voice
```

**B — from GitHub (latest / pre-release):**

```bash
uv pip install --python /path/to/hermes/venv/bin/python \
  "git+https://github.com/alaamh/hermes-plugin-teams-voice.git"
```

**C — from a local checkout (development):**

```bash
git clone https://github.com/alaamh/hermes-plugin-teams-voice.git
uv pip install --python /path/to/hermes/venv/bin/python -e ./hermes-plugin-teams-voice
```

> Installing into the wrong environment means Hermes won't see the plugin.
> Faster audio (optional): add the `numpy` extra, e.g. `hermes-plugin-teams-voice[numpy]`.

## Enable + run

```bash
hermes plugins list                            # confirm: teams_voice   (source: entrypoint)
hermes plugins enable teams_voice              # entry-point plugins are opt-in
hermes teams-voice serve --handler realtime    # voice bridge; also: streaming | echo | logging
hermes gateway run                             # (separately) the Teams chat plane + cron
```

## Configure

Config lives in Hermes's own files (not in this package):

- `~/.hermes/config.yaml` → `plugins.entries.teams_voice.config` — `host`/`port` (voice WS), the `realtime:` block (OpenAI/Azure), `allowlist`, `meeting_recap`, `share_point_site_id`, …
- `~/.hermes/.env` → secrets — `TEAMS_VOICE_SHARED_SECRET`, `AZURE_FOUNDRY_API_KEY`, `TEAMS_SHAREPOINT_SITE_ID`, …

`shared_secret` **must equal** the media worker's shared secret. Full reference:
[`hermes_teams_voice/README.md`](hermes_teams_voice/README.md).

## Upgrade / uninstall

```bash
uv pip install --upgrade hermes-plugin-teams-voice
uv pip uninstall hermes-plugin-teams-voice     # then it disappears from `hermes plugins list`
```

## How it loads

Hermes discovers pip plugins via the `hermes_agent.plugins` entry-point group. This
package exposes:

```toml
[project.entry-points."hermes_agent.plugins"]
teams_voice = "hermes_teams_voice"
```

Hermes imports `hermes_teams_voice` and calls its `register(ctx)` — registering the
`teams-voice` CLI, the status tool, and the session hook. Entry-point plugins are
opt-in, so `teams_voice` must be in `plugins.enabled` (`hermes plugins enable` does this).

## Requirements

- A working **Hermes Agent** install (the host; not a PyPI package).
- Python ≥ 3.10 and `aiohttp`; `ffmpeg` on PATH for streaming-mode TTS decode.
- A media worker that bridges the live Teams call audio/video into this plugin over the HMAC WebSocket (open-source, separate repo).

## Relationship to the bundled plugin

This is the same code as the in-tree `plugins/teams_voice` plugin, repackaged for pip
distribution so you don't have to fork Hermes. Install it on **vanilla** Hermes; don't
also keep a bundled `teams_voice` (same name → the entry-point would shadow it).

- **Voice/CVI** works fully on vanilla Hermes.
- **Chat-plane governance + SharePoint file attach** depend on the enhanced
  `plugins/platforms/teams` adapter; without it the plugin **degrades gracefully**
  (e.g. meeting minutes post as text instead of a SharePoint file card).

## License

MIT (matches Hermes Agent). Created by Alaaeldin Elhenawy — Dubai, UAE.
