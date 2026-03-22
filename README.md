# auto_sop

Cross-border/shop automation SOP repository.

## Structure

- `docs/strategy/`
  - upstream strategy and overall automation framing
- `docs/community-collection/`
  - daily community discovery and queue-building SOPs
- `docs/reddit-comment/`
  - Reddit comment automation docs
- `docs/reddit-followup/`
  - Reddit reply follow-up / review docs
- `assets/workbook/`
  - workbook snapshots / tracked copies

## Important note

The current production runtime still uses files under:

`/Users/chengkun/.openclaw/workspace`

This repo is the managed documentation / version-control copy.
Do not assume cron/runtime references have already been switched to this repo path.

## Current source-of-truth split

- Runtime / automation execution path:
  - `/Users/chengkun/.openclaw/workspace`
- Version-managed documentation path:
  - `/Users/chengkun/Documents/personal/shop_sop/auto_sop`

## Next recommended steps

1. Review folder structure
2. Add `.gitignore` if needed
3. Create first commit
4. Create GitHub private repo
5. Add remote and push
6. Later, decide whether to migrate runtime references from workspace into this repo
