---
name: homework-reminder
description: Cast a homework reminder message to Chromecast after a 2-minute delay. Uses TTS with Google Translate.
---

# Homework Reminder Skill

Cast a homework reminder to Chromecast after 2 minutes.

## How It Works

When triggered, this skill:
1. Waits 2 minutes (120 seconds)
2. Generates TTS using Google Translate
3. Speeds up audio 1.1x using ffmpeg
4. Starts local HTTP server on port 8766
5. Casts the audio to Chromecast at 192.168.31.219
6. Cleans up the HTTP server

## Message

"Jayden, please stop playing game and watching youtube. Its time to do homework and pack your school bag."

## Usage

Simply trigger the skill - it runs in background with the 2-minute delay.

## Manual Run

To run manually:
```bash
~/bin/cast-homework
```

## Troubleshooting

- If Chromecast not found, check IP: `castnow --ml`
- Port 8766 must be free
- Ensure Mac and Chromecast are on same network
