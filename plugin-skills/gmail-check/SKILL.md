---
name: gmail-check
description: Check Gmail inbox using browser automation. Use when user wants to see recent emails, unread messages, or inbox summary.
---

# Gmail Check

Check your Gmail inbox for recent emails and unread messages.

## Credentials
Stored in: macOS Keychain
- Service: google.com
- Account: brucekwok5@gmail.com

## How It Works

Uses browser automation with OpenClaw's openclaw profile.

### Steps:
1. Start browser (profile: openclaw)
2. Navigate to Gmail (may auto-login if credentials saved)
3. Parse inbox for recent/unread emails

### Key Commands:
```bash
# Start browser
browser action=start profile=openclaw

# Navigate to Gmail
browser action=navigate profile=openclaw targetUrl="https://mail.google.com/mail/u/0/"

# Get page content
browser action=snapshot profile=openclaw
```

## Output Format
```
=== Gmail Inbox ===

Unread: 5
- From: sender@example.com | Subject: Email subject | Time: 2h ago
- From: another@example.com | Subject: Another email | Time: Yesterday

Recent:
- From: recent@example.com | Subject: Recent subject | Time: 10m ago

[or]

⚠️ Please sign in manually
```

## Notes
- Shows unread count and recent emails
- Browser session persists after login
- May require 2FA verification on first login
