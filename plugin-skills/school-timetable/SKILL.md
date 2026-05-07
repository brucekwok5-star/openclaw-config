---
name: school-timetable
description: Get La Salle College Class 1F timetable by day, week, and season. Use when asked about Jayden's school schedule, classes, or timetable.
---

# School Timetable

## Week Pattern (2025-2026)

- **This week (Feb 23-27, 2026)**: Week 2
- **Next week (Mar 2-6)**: Week 1
- **Pattern**: Alternates Week 1, Week 2, Week 1...

## Quick Lookup

- **Current week**: Week 2 (as of Feb 27, 2026)
- **Next week**: Week 1 (starting Mar 2, 2026)
- **Current season**: Winter (use Winter Time column)
- **Today**: Check `memory/today.md` or infer from conversation date

## Usage

When asked about Jayden's classes:

1. Determine the day (Monday-Friday)
2. Determine the week (Week 1 or Week 2)
3. Look up the day in `references/timetable.md`
4. Return the classes for that day

## Response Format

Return a concise list:
- Time | Subject | Teacher

## Season Reference

- **Winter Time**: October - March (currently active)
- **Summer Time**: April - September

## Reference Files

- Full timetable: [timetable.md](references/timetable.md)
