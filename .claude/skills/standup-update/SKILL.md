---
name: standup-update
description: Generate a daily standup update by aggregating work from Notion, Gmail, Slack, and Calendar
---

# Standup Update Skill

Generate a morning standup update by aggregating work from the previous day across multiple sources.

## Execution Steps

### Step 1: Get Yesterday's Date

Use `mcp__googlecalendar__get_current_date_time` to get the current date, then calculate yesterday's date. You'll need different formats for different APIs:
- Gmail: `YYYY/MM/DD` (e.g., `2026/01/08`)
- Slack: Standard date format
- Calendar: RFC3339 timestamps (e.g., `2026-01-08T00:00:00Z` to `2026-01-08T23:59:59Z`)
- File output: `YYYY-MM-DD` (e.g., `2026-01-08`)

### Step 2: Fetch Data in Parallel

Spawn 4 subagents using the Task tool with `run_in_background: true` to fetch data in parallel:

#### Notion Subagent
```
Fetch completed work from yesterday AND incomplete weekly goals:

**Completed work:**
1. Use mcp__notion__notion-fetch to get the Weekly Goals database (ID: f72f8046847b4c23bc81a465576d7842)
2. Look for items with status changes or completions from yesterday
3. Also search for any Notion pages modified yesterday using mcp__notion__notion-search with date filter

**Incomplete weekly goals (for priorities):**
1. From the Weekly Goals database, identify items with status "In Progress" or "Not Started"
2. These represent ongoing priorities that should be worked on today

Return:
- A list of completed items with title, description, and context
- A list of incomplete goals with title and current status
```

#### Gmail Subagent
```
Fetch emails sent yesterday:
1. Use mcp__gmail__fetch_emails with query "from:me after:YYYY/MM/DD before:YYYY/MM/DD"
   (Replace dates with yesterday and today)
2. Extract for each email:
   - Subject line
   - Recipient(s)
   - Brief context of what the email was about

Return a list of emails with subject, recipients, and context.
```

#### Slack Subagent
```
Fetch Slack messages sent yesterday:
1. Use mcp__slack__search_messages with query "from:@me" and appropriate date filter
2. For each message found, extract:
   - Channel name
   - Brief summary of the message content
   - Who was involved in the conversation

Return a list of key Slack communications with channel, summary, and participants.
```

#### Calendar Subagent
```
Fetch meetings from yesterday AND today:

**Yesterday's meetings:**
1. Use mcp__googlecalendar__find_event with timeMin and timeMax set to yesterday's date range
2. For each meeting, extract:
   - Meeting title
   - Attendees
   - Duration
   - Any notes if available

**Today's meetings (for priorities):**
1. Use mcp__googlecalendar__find_event with timeMin and timeMax set to today's date range
2. For each meeting, extract:
   - Meeting title
   - Start time
   - Attendees

Return:
- A list of yesterday's meetings with title, attendees, and duration
- A list of today's scheduled meetings with title, time, and attendees
```

### Step 3: Aggregate Results

Wait for all subagents to complete using TaskOutput tool, then collect all results into a unified list of work items.

### Step 4: Ask Clarifying Questions

Use the `AskUserQuestion` tool to disambiguate and clarify work items. Ask about:

1. **Related items**: "Are these items about the same project?"
   - Example: An email to John and a Slack message to John might be about the same topic

2. **Vague items**: "What was the outcome of this meeting?"
   - Example: A meeting titled "Team Sync" needs context about what was discussed

3. **Grouping**: "Should I group these related activities together?"
   - Example: Multiple commits/emails about the same feature

4. **Missing context**: "What was the main accomplishment from this work?"
   - Example: A long Slack thread might need a summary

Keep questions concise and batch related questions together when possible.

### Step 5: Generate Plain Text Summary

Create a markdown summary organized by categories:

```markdown
# Standup Update - {YYYY-MM-DD}

## Key Accomplishments
- [Main things completed yesterday]

## Meetings
- [Meeting 1]: [Brief outcome/notes]
- [Meeting 2]: [Brief outcome/notes]

## Communications
- Emails: [Summary of important emails sent]
- Slack: [Key conversations/decisions]

## Other Work Items
- [Any other relevant work]

## Today's Priorities

### Scheduled Meetings
- [Time] [Meeting title] with [Attendees]

### Goals to Work On
- [Goal 1] (Status: In Progress)
- [Goal 2] (Status: Not Started)
```

### Step 6: Save to File

Write the summary to `updates/{YYYY-MM-DD}-morning-update.md` using the Write tool.

Example filename: `updates/2026-01-08-morning-update.md`

### Step 7: Display Summary

Show the generated summary to the user and confirm the file was saved.

## Important Notes

- If any data source fails or times out, continue with the available data and note what was unavailable
- Prioritize quality over quantity - focus on meaningful work items, not every small action
- Keep the summary concise and actionable for a team standup context
- The summary should be readable in 1-2 minutes
