---
name: minimax-usage
description: Check MiniMax API usage/credits. Shows current usage percentage and reset time.
---

# MiniMax Usage Check

Check your MiniMax API usage credits and quota.

## Credentials
If login required, credentials stored in: `~/.openclaw/credentials/minimax.json`
```json
{
  "phone": "13143973803",
  "password": "Z@vpxQV6vKwKS.M"
}
```

## Important URL
**Use this page for usage info:** `https://platform.minimaxi.com/user-center/payment/coding-plan`

This shows your Coding Plan quota and usage percentage.

## How It Works

This skill uses **browser automation** to log into MiniMax and scrape the usage page.

### Steps:
1. Start OpenClaw browser (profile: openclaw)
2. Navigate to: `https://platform.minimaxi.com/user-center/payment/coding-plan`
3. Take snapshot and parse the usage data (no login needed if already authenticated)

### Key Commands:
```bash
# Navigate directly to Coding Plan page
browser action=navigate profile=openclaw targetUrl="https://platform.minimaxi.com/user-center/payment/coding-plan"

# Get page content
browser action=snapshot profile=openclaw
```

## For Other Tasks

To automate any login-protected website:

1. **Find login page URL** - e.g., `/login?redirect=/target-page`

2. **Get element refs** - Take snapshot to find textbox/button refs (e.g., e29 for username, e36 for password, e60 for submit)

3. **Type → Click → Navigate**:
   - Type into textbox: `{"kind":"type", "ref":"e29", "text":"username"}`
   - Click button: `{"kind":"click", "ref":"e60"}`
   - Navigate after login: `{"kind":"navigate", "targetUrl":"..."}`

4. **Parse snapshot** - Look for the data in the snapshot tree (e.g., "25% 已使用")

## Notes
- Browser must be started with `profile=openclaw` (isolated, no extension needed)
- Session persists - can navigate after login
- Use `snapshot` to see page content and find element refs
