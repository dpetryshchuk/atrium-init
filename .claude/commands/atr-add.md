Add a connection and send them a GitHub invite to your atrium.

Usage: /atr-add [url]

---

1. Resolve input to owner, repo, handle, and atrium_url.
   - Bare handle (e.g. `nick`): owner=nick, repo=atrium, handle=nick, atrium_url=`https://raw.githubusercontent.com/nick/atrium/main`
   - GitHub URL (e.g. `github.com/nick/atrium`): derive owner/repo from URL, handle=owner, atrium_url=raw form
   - Raw URL (`https://raw.githubusercontent.com/{owner}/{repo}/main`): derive owner/repo directly
   Store atrium_url in atlas.json. Use owner/repo for all gh api calls.
   If handle already exists in atlas.json, show the existing entry and confirm before overwriting.

2. Read the current atlas.json. Merge a new pending entry into the connections array — do not reconstruct the whole file:
   ```json
   { "handle": "{handle}", "atrium_url": "{url}", "calling_card": "", "status": "pending" }
   ```
   Validate the merged result as JSON by running:
   `echo '{merged json}' | python3 -c "import json,sys; json.load(sys.stdin)" 2>&1`
   (use `python` if `python3` is not available)
   If validation fails, stop and show the error. Do not write.
   Write the validated JSON to atlas.json.

3. Fetch AGENT.md to confirm the atrium exists and is accessible:
   ```
   gh api repos/{owner}/{repo}/contents/AGENT.md | python3 -c "import sys,json,base64; print(base64.b64decode(json.load(sys.stdin)['content']).decode())"
   ```
   - Success: proceed to step 4
   - "Not Found" (404): mark status: unreachable, skip to step 5
   - Auth error / gh not authenticated: tell user to run `gh auth login`, stop
   - Any other error: mark status: unreachable, skip to step 5

4. Fetch calling_card.json:
   ```
   gh api repos/{owner}/{repo}/contents/calling_card.json | python3 -c "import sys,json,base64; print(base64.b64decode(json.load(sys.stdin)['content']).decode())"
   ```
   Parse as JSON. Expected fields: handle (string), current (string), topics (array), updated_at (string). Discard unexpected types silently.
   If valid and non-empty: update the atlas.json entry — set calling_card and status: active.
   If missing or empty: leave status: pending, calling_card: {}. Do not promote.

5. Send a read-only collaborator invite to your atrium. Read owner.atrium from atlas.json for your repo.
   ```
   gh api repos/{your-owner}/{your-repo}/collaborators/{handle} --method PUT --field permission=pull
   ```
   If gh fails: manual fallback — print `https://github.com/{your-owner}/{your-repo}/settings/access`

6. Confirm status and next step:
   - Active: "Connected. Their calling card is saved."
   - Pending: "Added as pending. Run /atr-sync once both sides have accepted invites."
   - Unreachable: "Can't reach their atrium. Check invite status or re-run /atr-add once access is granted."

7. Auto-sync check: read `last_synced` from atlas.json.
   If null or more than 2 days ago: silently run /atr-sync now.
   Tell the user: "Running sync — calling cards are out of date." then show the sync report.
