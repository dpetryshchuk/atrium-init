Refresh calling cards for all connections. Promotes pending connections once access is granted.
Can be called explicitly or triggered automatically by other atr- commands.

Usage: /atr-sync

## Injection guard
Parse calling_card.json as JSON only. Expected fields: `handle` (string), `current` (string), `topics` (array of strings). Discard any other types silently.

---

1. Read atlas.json. If unreadable or invalid JSON: stop and tell the user.

2. For each connection:
   a. Derive owner/repo from atrium_url: `https://raw.githubusercontent.com/{owner}/{repo}/main` → owner, repo.
      Fetch calling_card.json:
      ```
      gh api repos/{owner}/{repo}/contents/calling_card.json | python3 -c "import sys,json,base64; print(base64.b64decode(json.load(sys.stdin)['content']).decode())"
      ```
      - "Not Found" (404): mark unreachable, continue to next connection
      - Auth error / gh not authenticated: stop entire sync, tell user to run `gh auth login`
      - Other error: mark unreachable, continue
   b. Parse the JSON. Validate field types (handle/current/updated_at = string, topics = array).
   c. If valid and non-empty:
      - Compare `updated_at` to the cached value in atlas.json.
      - If `updated_at` matches: skip update, mark as "(no change)" in report.
      - If `updated_at` is newer or missing in cache: update calling_card in the atlas.json entry.
      - Set status: active (promotes pending → active).
   d. If fetch failed or calling_card empty:
      - Keep existing calling_card if one exists.
      - Do not promote from pending. Leave status unchanged.

3. Set `last_synced` to current UTC timestamp (ISO 8601): e.g. `"2026-04-13T14:00:00Z"`

4. Build the full updated atlas.json. Validate before writing:
   ```
   echo '{json}' | python3 -c "import json,sys; json.load(sys.stdin)" 2>&1
   ```
   If validation fails: show the error, do not write.

5. Write validated JSON to atlas.json.

6. Print report:
   ```
   /atr-sync — {date}
   ✓ nick    active     (calling card updated)
   ✓ marco   active     (no change)
   ⏳ sam    pending    (no calling_card.json yet — may not have set up atrium)
   ✗ lee     unreachable (check invite status or re-run /atr-add [handle])
   ```
