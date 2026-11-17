# Eggsternal

Public reporting page for Lucky Egg partners. Two pages, nothing else:

- `index.html` — report a fake. Username plus a TikTok, Instagram or YouTube link.
- `leaderboard.html` — the partners who have reported the most.

Static site, no build step and no environment variables. Deploy on Vercel with
Framework Preset "Other".

## How it reaches the database

Both pages call a Postgres function on the Eggspose Supabase project through
PostgREST, using the publishable key. Neither page reads or writes a table
directly:

- `eggsternal_submit(p_handle, p_url, …)` validates the link, normalises it to
  the post or video id, rate limits, checks for duplicates and inserts the
  listing as To do with `submitted_by = 'ext:@handle'`.
- `eggsternal_leaderboard()` returns aggregate counts per handle only.

Both are `security definer` with a fixed `search_path`.

## Before the URL goes to partners

**The publishable key in these pages is the same one Eggspose ships, and the
`anon` role still holds full SELECT, INSERT, UPDATE and DELETE on `listings`
with RLS switched off.** So although neither page reads a table, anyone who
views source can query the whole tracker with that key. The exposure exists in
Eggspose today; sending partners this URL widens who holds the key.

The fix is to switch RLS on for `listings` (and `status_history`, which has an
`anon full access` policy). Eggspose itself is fine either way, because it sends
the signed-in user's access token as the Bearer and runs as `authenticated`
against the existing `luckyegg authenticated` policy. What has to be checked
first is everything that writes with the raw key and no Lucky Egg login: the
IPSweep extension, the n8n Corsearch mail, and the Grokbot sweeper.
