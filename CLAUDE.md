# CLAUDE.md — operating instructions for Claude Code

## Project
Home Assistant config-as-code for the Lang residence (repo: `langhome`). Phase 1 goal: one-button **Activities** (e.g. "Watch TV", "Play Pandora Outside") plus supporting zone lighting — simple enough for guests and a dog sitter. See `README.md` for full context and roadmap.

## Target
- Home Assistant host: `10.100.10.10` (UI at `:8123`)
- Config lives in `/config` on that host.

## Deploy
- This repo maps into `/config`: `packages/` → `/config/packages/`; `dashboards/` → `/config/dashboards/` (or a YAML-mode dashboard).
- Deploy over **SSH to 10.100.10.10** (credentials live in the local SSH config — NEVER stored in this repo).
- After any change: run `ha core check`, then reload only the affected domain (Reload Scenes / Reload Scripts). Restart HA only when adding a brand-new package file or editing `configuration.yaml`.

## Conventions
- All Phase 1 logic lives in HA **packages** under `packages/`. `configuration.yaml` includes:
  `homeassistant: { packages: !include_dir_named packages }`
- **Activities = scripts** (ordered sequences). **Lighting moments = scenes.**
- Keep entity IDs predictable and stable — the dashboard and future voice intents depend on them.

## Guardrails
- NEVER edit `/config/scenes.yaml` or `/config/automations.yaml` — those are UI-managed and separate from this repo.
- NEVER commit secrets, tokens, or credentials. HA secrets belong in `/config/secrets.yaml` on the host, not in the repo.
- Ask before restarting Home Assistant or reloading anything that could interrupt media playback.
- Work in small, verifiable steps: edit → `ha core check` → reload → confirm entity/state → commit.

## Current state
- Deployable now: `packages/lighting_phase1.yaml` (6 scenes), `dashboards/family.yaml` (HACS + Mushroom installed).
- Pending: `packages/activities_phase1.yaml` — must first discover the `media_player` entities and their `source_list`. Known gotchas to flag for review: Samsung TV HDMI-input switching may not work via the HA integration (route via the OREI matrix or IR instead); Pandora station selection depends on the Yamaha's saved presets.
