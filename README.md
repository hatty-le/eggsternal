# Eggsternal

Public reporting page for Lucky Egg partners. Two pages, nothing else:

- `index.html` — report a fake. Username plus a TikTok, Instagram or YouTube link.
- `leaderboard.html` — the partners who have reported the most.

Static site, no build step and no environment variables. Deploy on Vercel with
Framework Preset "Other".

## How it reaches the database

Both pages call a Postgres function on the Eggspose Supabase project through
PostgREST, using the publishable key. Neither page reads or writes a table
directly, so the key in the page grants no table access of its own:

- `eggsternal_submit(p_handle, p_url, …)` validates the link, normalises it to
  the post or video id, rate limits, checks for duplicates and inserts the
  listing as To do with `submitted_by = 'ext:@handle'`.
- `eggsternal_leaderboard()` returns aggregate counts per handle only.

Both are `security definer` with a fixed `search_path`.
