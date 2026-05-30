# langhome

Home Assistant configuration (infrastructure-as-code) for the Lang residence.
Deploys to Home Assistant @ `10.100.10.10`.

## Goal — Phase 1: Activities

One-button **Activities** anyone can use without thinking — wife, guests, a dog sitter.
Same model as Harmony / Control4 Activities. Examples:

- **Watch TV (Living Room)** → Frame on → correct input → Yamaha + sub on → correct input → a remote/transport card appears on the tablet to navigate content.
- **Play Pandora Outside** → patio + garden speakers on → pick a station.
- Plus simple per-zone **lighting** (Living, Pub, Patio, Garden) that activities can fold in.

Technically: **Activities are HA scripts** (ordered sequences). Lighting "moments" are **scenes** that scripts can call. The lighting scenes are done; the activity scripts are next (pending the AV entity export — see Status).

## Repo structure

```
langhome/
├── packages/
│   ├── lighting_phase1.yaml      # zone lighting scenes (building blocks)  [ready]
│   └── activities_phase1.yaml    # Watch TV / Play Pandora Outside, etc.   [pending AV export]
├── dashboards/
│   └── family.yaml               # tablet dashboard (lighting now; activities-first once scripts exist)
└── README.md
```

## Prerequisites — manual, one-time (Claude Code CANNOT do these)

1. **Add-ons** (Add-on Store, UI only): Studio Code Server [done]. Optionally **Advanced SSH & Web Terminal** so Claude Code can run `ha core check` / reload on the host.
2. **HACS** — this is NOT an add-on. Install once:
   - In a terminal (Studio Code Server's terminal, or SSH): `wget -O - https://get.hacs.xyz | bash -`
   - Restart HA → Settings → Devices & Services → Add Integration → **HACS** → authorize with GitHub.
3. **HACS frontend cards** (via the HACS UI): **Mushroom** now; browser_mod, mini-media-player later.
4. **Integrations with a login/auth flow** (Settings → Devices & Services): confirm Samsung TV + Yamaha are connected; add **Meross** (garden plug); any cloud logins.
5. **Device pairing / OAuth** — anything needing a button press or browser login.

Everything that is a **file** — packages, activity scripts, automations, scenes, dashboard YAML, `configuration.yaml`, card-resource registration — is Claude Code's job.

## Deploy workflow

- `configuration.yaml` must include:
  ```yaml
  homeassistant:
    packages: !include_dir_named packages
  ```
- Sync this repo to `/config` on `10.100.10.10` via: **Git pull add-on** (versioned), Claude Code over **SSH/rsync**, or Studio Code Server / Samba.
- Apply: Developer Tools → YAML → **Check Configuration** → **Reload Scenes** / **Reload Scripts** (or restart).
- Leave UI-managed `scenes.yaml` / `automations.yaml` out of scope.

## Using Claude Code

Run Claude Code on **your computer**, in a clone of this repo — not inside Studio Code Server. On the LAN with SSH, the loop is: Claude Code edits the repo → commit → deploy via the Git pull add-on, or have Claude Code SSH/rsync into `/config` and reload. Use VS Code on your computer as the editor (Claude Code runs in its terminal); keep Studio Code Server for quick manual peeks.

> The Claude chat that designs this and Claude Code are separate tools: the chat generates config; Claude Code applies and iterates it against your live HA.

## Status

**Lighting (building blocks) — ready:**

| Zone | Light(s) | Status |
|------|----------|--------|
| Living Room | `light.living_room_main_lights` (dimmer) + Pico | ✅ |
| Pub | `light.pub_main_lights` (dimmer) | ✅ |
| Patio | `switch.exterior_back_porch_lights` (switch) + `light.pentair_08_d5_de_lights` (pool) | ✅ |
| Garden | Meross 2-port plug | ⏳ add via `meross_lan`, then fill scenes |
| Gemstone (eaves) | proprietary app | ❓ research — likely app-only |

**Activities (the Phase 1 headline) — pending:**

| Activity | Needs | Status |
|----------|-------|--------|
| Watch TV (Living Room) | Frame + Yamaha entities & source names | ⏳ pending media_player export |
| Play Pandora Outside | Yamaha Zone 2/3 + Pandora presets | ⏳ pending media_player export |

**Next input needed:** run the `media_player` export (entity IDs + `source_list`) so activities can be written against real source strings. Known caveats: Samsung's HA integration may not switch HDMI inputs directly (we'll route via the OREI matrix or IR instead), and Pandora station-picking depends on the Yamaha's saved presets.

## Notes

- **Garden:** Meross plug isn't in HA yet — add via the `meross_lan` HACS integration (local), then reference its two outlet switches in the Garden scenes + party moments.
- **Patio dimming:** back porch light is a Caséta on/off *switch* (PD-8ANS), so patio scenes are on/off only. Swap for a PD-6WCL dimmer later if you want dimming.
- **Gemstone:** permanent eaves lighting, currently app-controlled; no confirmed HA integration. Check the controller for a local API or Matter before counting on it.
- **Spare Pico** ("Unassigned Chandelier Remote 1") can be bound to an Activity or scene — see the commented automation in `lighting_phase1.yaml`.

## Roadmap

- **Phase 1:** Activities (Watch TV, Play Pandora Outside, etc.) + supporting zone lighting + tablet dashboard.
- **Phase 2:** distributed audio (WiiM + Music Assistant grouping) and video routing via the OREI matrix.
- **Phase 3:** locks (Schlage — already integrated; evaluate Matter), Ring doorbell → camera pop-up on tablet (browser_mod), garage via ratgdo (MyQ API is blocked).
- **Phase 4:** bedrooms (Master, Gavin, Elsie, Bailey).
- **Phase 5:** "Skippy" private voice agent (HA Assist + harness via MCP).
