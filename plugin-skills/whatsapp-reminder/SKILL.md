---
name: whatsapp-reminder
description: Schedule a reminder to send a WhatsApp message. Usage: "<message>, tmr 2pm" or "<message>, tomorrow 3pm"
---

# WhatsApp Reminder

Schedule a Telegram reminder to send a WhatsApp message later.

## Usage

When user gives input like:
- "doctor appointment, tmr 2pm"
- "meeting, tomorrow 3pm"
- "call mum, today 5pm"

The skill will:
1. Parse the message content and time
2. Create a cron job to remind via Telegram at the specified time

## Example

User: "doctor appointment, tmr 2pm"
→ Reminder: "Time to send WhatsApp: doctor appointment" (at tomorrow 2pm)

## Implementation

Use cron tool to schedule:
- schedule: { "kind": "at", "at": "2026-03-06T14:00:00" }
- payload: { "kind": "systemEvent", "text": "⏰ Reminder: Send WhatsApp - doctor appointment" }
- delivery: { "mode": "announce" }
- sessionTarget: "main"
