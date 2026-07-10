# How to set up Google OAuth

Goal: credentials on disk that let the agent read Gmail, modify labels, and read the calendar, refreshing silently forever under cron.

> The `gmail-brief-agent auth` subcommand referenced below is planned; until it exists, the same flow can be run with a short `google-auth-oauthlib` script.

## Prerequisites

- A Google Cloud project with the **Gmail API** and **Calendar API** enabled. If you already run a Google Workspace CLI (gws or similar), you have one.
- An OAuth client ID of type **Desktop app** in that project, and its `client_secret.json`. Reusing an existing desktop client is fine; each app still gets its own token.
- The project's OAuth consent screen must have publishing status **In production**, not Testing. See gotchas below for why this matters more than anything else on this page.

## Scopes

The agent requests exactly two:

| Scope | Grants | Withholds |
|-------|--------|-----------|
| `https://www.googleapis.com/auth/gmail.modify` | Read mail, change labels | Sending, permanent deletion |
| `https://www.googleapis.com/auth/calendar.readonly` | Read calendar events | Any calendar writes |

## Steps

1. Place `client_secret.json` at the path named in your config (outside the repo, `chmod 600`).
2. Run the one-time consent on a machine with a browser:
   ```
   gmail-brief-agent auth
   ```
   A browser opens Google's consent screen. Expect a "Google hasn't verified this app" interstitial for an unverified personal app requesting Gmail scopes; click Advanced, then proceed. Approve the two scopes.
3. The flow writes `token.json` (access token plus refresh token) next to your config. `chmod 600`, gitignored, never committed.
4. If the agent runs on a headless server, do step 2 on your laptop and copy the token over:
   ```
   scp token.json vps:~/.config/gmail-brief-agent/
   ```
   Alternatively run the flow on the server through an SSH port forward (`ssh -L 8080:localhost:8080 vps`) so the browser redirect lands.
5. Verify: run any read-only agent command twice. The second run must not open a browser; the refresh token is doing its job.

## Gotchas

- **Testing status kills cron weekly.** A consent screen left in Testing status issues refresh tokens that expire after 7 days. The job then fails silently every week until someone re-consents. Flip the app to In production (it stays unverified; that is fine for personal use).
- **Gmail scopes are "restricted"** in Google's classification. For a personal unverified app this only means the scary interstitial during consent. Verification is only required for public distribution.
- **Service accounts do not work here.** For Gmail they require domain-wide delegation, which only Workspace admins can grant. User-consent OAuth is the correct mechanism for a personal account.
- **`token.json` is the key to the inbox.** The client secret identifies the app; the token is the credential. Protect the token like a password.
