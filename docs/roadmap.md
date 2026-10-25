# Roadmap

Status as of 2026-10-25.

## Planned skills

### `israel-post-branch-search`
- Source of truth candidate: the same upstream CDN branch list that `skills/israel-post-appointment/data/branches.json` was generated from (~1,524 branches total; only the 314 booking-capable ones are bundled today).
- Needs: opening hours, services per branch, nearest-branch by coordinates.
- Open question: ship the full branch list, or query live each time.

### `israel-post-tracking`
- Prior art (third-party, not reviewed yet): https://github.com/WolfikOz/israel-post-tracking — read it before building.
- Develop by driving the tracking page in a real browser; capture the underlying request if there is one, as with the appointment flow.

## Maintenance gaps
- `refresh-branches.py` (referenced by `branches.json` → `generated_from`) is not in this repo. Recreate it when building branch search, and commit it under `scripts/`.
- Cross-session appointment cancel (by phone) is unsupported; investigate whether the site exposes it.
