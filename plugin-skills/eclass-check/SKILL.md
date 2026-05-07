---
name: eclass-check
description: Check La Salle College eClass notices and updates using browser automation
---

# eClass Check

Check La Salle College eClass platform for notices and updates.

## Credentials
Stored in: macOS Keychain
- Service: eclass.lasalle.edu.hk
- Account: p25186

## How It Works

Uses browser automation with OpenClaw's openclaw profile.

### Steps:
1. Start browser (profile: openclaw)
2. Navigate to eClass login page
3. Sign in using credentials from Keychain
4. Check for new notices/notifications

### Key Commands:

```bash
# Open eClass in browser
open -a "Google Chrome" "https://eclass.lasalle.edu.hk/home/eService/notice/"

# Or use browser tool
browser action=open profile=openclaw targetUrl="https://eclass.lasalle.edu.hk/home/eService/notice/"
```

## Usage

Run the skill to:
- Check for new notices
- View recent updates
- Login to eClass platform

## Notes

- eClass is La Salle College's online learning platform
- Main URL: https://eclass.lasalle.edu.hk
- Notice board: /home/eService/notice/
