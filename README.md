# Israel Post

Agent skills for Israel Post (דואר ישראל, *doar*) operations, driven against the live Hebrew site at `israelpost.co.il`.

## Skills

| Skill | Status | What it does |
|---|---|---|
| `israel-post-appointment` | working | Check the next free slot, book, or cancel a counter appointment at a branch (package pickup, general counter, forex, vehicle transfer). Resolves the branch from a bundled dataset of 314 booking-capable branches. |
| `israel-post-branch-search` | planned | Find branches by city, address or nearest point; opening hours and services offered. |
| `israel-post-tracking` | working | Look up a tracking code (e.g. `RR123456789IL`): status, history, collecting branch and shelf number, customs due, days left to collect. Drives the tracking page in Chrome and reads the JSON it receives (`reference/itemtrace-api.md`). |
| `israel-post-calendar-event` | working | After a booking, add the appointment to the user's calendar: branch address, time, and the parcels to collect with shelf numbers and customs due. Uses any available calendar tool, else writes an `.ics` file. |

Planned skills are developed live in a real browser first, then written up. See `docs/roadmap.md`.

## Configuration

User values live outside the plugin in `~/.config/israel-post/preferences.yaml`:

```yaml
israel_post:
  default_branch_no: 101
  default_service: package_pickup   # general | forex | package_pickup | vehicle_transfer
  default_phone: "05XXXXXXXX"
```

## Installation

```bash
claude plugins install israel-post@danielrosehill
```

## History

`israel-post-appointment` moved here on 2026-10-25 from the `israel-agent-skills` plugin ([Claude-Israel-Agent-Skills-Plugin](https://github.com/danielrosehill/Claude-Israel-Agent-Skills-Plugin)). Its config path changed from `~/.config/israel-agent-skills/preferences.yaml` to `~/.config/israel-post/preferences.yaml`.

## License

MIT
