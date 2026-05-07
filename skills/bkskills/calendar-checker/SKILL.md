---
name: calendar-checker
description: Check today's calendar events from macOS Calendar app. Use when asked about today's schedule, appointments, or events. Supports querying specific calendars or all calendars at once.
---

# Calendar Checker

Query today's events from the macOS Calendar app.

## Usage

### Check all calendars for today's events

```applescript
tell application "Calendar"
    set output to ""
    set todayStart to (current date)
    set time of todayStart to 0
    set todayEnd to todayStart + 1 * days
    
    set calendarsToCheck to {"Jayden", "Calendar", "Family", "CloudHK", "Bruce Kwok", "Agile results", "Cloud roster"}
    
    repeat with calName in calendarsToCheck
        try
            tell calendar calName
                set todaysEvents to events whose start date ≥ todayStart and start date < todayEnd
                repeat with anEvent in todaysEvents
                    set eventTitle to summary of anEvent
                    set eventStart to start date of anEvent
                    set output to output & "[" & calName & "] " & (time string of eventStart) & ": " & eventTitle & "\n"
                end repeat
            end tell
        on error
            -- skip unavailable calendars
        end try
    end repeat
    
    if output is "" then
        return "No events scheduled for today."
    else
        return output
    end if
end tell
```

### Check specific calendar

Replace `calName` with the calendar name you want to query. Common calendars:
- Personal: "Jayden", "Calendar", "Family"
- Work: "CloudHK", "Bruce Kwok", "Agile results", "Cloud roster"

### Troubleshooting

If AppleScript hangs:
- Calendar app may have Exchange connectivity issues
- Force quit: `killall Calendar calendard`
- Re-authenticate Exchange account in System Settings

## Output Format

Returns formatted list:
```
[Calendar Name] HH:MM:SS AM/PM: Event Title
```

Or "No events scheduled for today" if empty.
