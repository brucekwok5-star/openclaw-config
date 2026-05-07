---
name: calendar-check
description: Check Google Calendar events using browser automation
---

# Google Calendar Check

Check your Google Calendar for today's events.

## Credentials
Stored in: macOS Keychain
- Service: google.com
- Account: brucekwok5@gmail.com

To add/modify credentials:
```bash
security add-internet-password -r "http" -s "google.com" -a "your@email.com" -w "yourpassword"
```

## How It Works

Uses browser automation with OpenClaw's openclaw profile.

### Steps:
1. Start browser (profile: openclaw)
2. Navigate to Google Calendar
3. Sign in if needed (credentials in Keychain may auto-fill)
4. Parse today's events from calendar view

### Key Commands:
```bash
# Start browser
browser action=start profile=openclaw

# Navigate to Calendar
browser action=navigate profile=openclaw targetUrl="https://calendar.google.com/"

# Get page content
browser action=snapshot profile=openclaw
```

## Output Format
```
=== Google Calendar ===

Today: Friday, Feb 27
Events:
- 10:00 AM - Meeting
- 2:00 PM - Dentist

[or]

⚠️ Please sign in manually
```

## Notes
- Browser session persists after login
- Shows today's date and events
- May require 2FA verification
