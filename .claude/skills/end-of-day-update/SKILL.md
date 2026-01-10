---
name: end-of-day-update
description: Generate an end-of-day summary by aggregating today's work from Notion, Gmail, Slack, Calendar, and GitHub
---

# End of Day Update Skill

Generate an evening wrap-up by aggregating work from today across multiple sources. Unlike the morning standup (which summarizes yesterday), this skill focuses on TODAY's accomplishments and includes reflection sections.

## Execution Steps

### Step 1: Get Today's Date

Use `mcp__googlecalendar__get_current_date_time` to get the current date. You'll need different formats for different APIs:
- Gmail: `YYYY/MM/DD` (e.g., `2026/01/08`)
- Slack: Standard date format
- Calendar: RFC3339 timestamps (e.g., `2026-01-08T00:00:00Z` to `2026-01-08T23:59:59Z`)
- GitHub (`gh` CLI): `YYYY-MM-DD` (e.g., `2026-01-08`)
- File output: `YYYY-MM-DD` (e.g., `2026-01-08`)

Also calculate tomorrow's date for the Gmail query end bound and tomorrow's calendar context.

### Step 2: Fetch Data in Parallel

Spawn 5 subagents using the Task tool with `run_in_background: true` to fetch data in parallel:

#### Notion Subagent
```
Fetch completed work from TODAY AND current weekly goals status:

**Completed work:**
1. Use mcp__notion__notion-fetch to get the Weekly Goals database (ID: f72f8046847b4c23bc81a465576d7842)
2. Look for items with status changes or completions from today
3. Also search for any Notion pages modified today using mcp__notion__notion-search with date filter

**Current weekly goals status:**
1. From the Weekly Goals database, identify all items and their current status
2. Note which items are "Complete", "In Progress", or "Not Started"

Return:
- A list of completed items with title, description, and context
- A list of all weekly goals with their current status
```

#### Gmail Subagent
```
Fetch emails sent TODAY:
1. Use mcp__gmail__fetch_emails with query "from:me after:YYYY/MM/DD before:YYYY/MM/DD"
   (Replace with today's date and tomorrow's date)
2. Extract for each email:
   - Subject line
   - Recipient(s)
   - Brief context of what the email was about

Return a list of emails with subject, recipients, and context.
```

#### Slack Subagent
```
Fetch Slack messages sent TODAY:
1. Use mcp__slack__search_messages with query "from:@me" and appropriate date filter for today
2. For each message found, extract:
   - Channel name
   - Brief summary of the message content
   - Who was involved in the conversation

Return a list of key Slack communications with channel, summary, and participants.
```

#### Calendar Subagent
```
Fetch meetings from TODAY AND tomorrow:

**Today's meetings (for recap):**
1. Use mcp__googlecalendar__find_event with timeMin and timeMax set to today's date range
2. For each meeting, extract:
   - Meeting title
   - Attendees
   - Duration
   - Any notes if available

**Tomorrow's meetings (for context):**
1. Use mcp__googlecalendar__find_event with timeMin and timeMax set to tomorrow's date range
2. For each meeting, extract:
   - Meeting title
   - Start time
   - Attendees

Return:
- A list of today's meetings with title, attendees, and duration
- A list of tomorrow's scheduled meetings with title, time, and attendees
```

#### GitHub Subagent
```
Fetch GitHub activity from TODAY in the BrainGridAI organization:

**Using gh CLI via Bash tool:**
1. Fetch commits:
   `gh search commits --author=@me --committer-date=">=${TODAY}" --owner=BrainGridAI --limit=50 --json repository,sha,commit`

2. Fetch PRs created:
   `gh search prs --author=@me --created=">=${TODAY}" --owner=BrainGridAI --limit=20 --json repository,title,state,url,createdAt`

3. Fetch PRs merged:
   `gh search prs --author=@me --merged=">=${TODAY}" --owner=BrainGridAI --limit=20 --json repository,title,url`

4. Fetch issues created:
   `gh search issues --author=@me --created=">=${TODAY}" --owner=BrainGridAI --limit=20 --json repository,title,state,url`

Parse the JSON output and return:
- List of commits grouped by repository with commit messages
- List of PRs with title, status (opened/merged), and repository
- List of issues created with title and repository
```

### Step 3: Aggregate Results

Wait for all subagents to complete using TaskOutput tool, then collect all results into a unified list of work items.

### Step 4: Ask Clarifying Questions

Use the `AskUserQuestion` tool to gather additional context. Ask about:

1. **Blockers & Challenges**: "Did you encounter any blockers or challenges today?"
   - Example: Waiting on dependencies, technical issues, unclear requirements

2. **Key Wins**: "What was your biggest win or accomplishment today?"
   - Example: Shipped a feature, resolved a difficult bug, closed a deal

3. **Reflection**: "Any notes or reflections to capture?"
   - Example: Learnings, things to remember, thoughts on process

4. **Tomorrow's Focus**: "What are your top priorities for tomorrow?" (optional)
   - Example: Specific tasks to tackle first thing

Keep questions concise and batch related questions together when possible.

### Step 5: Generate Markdown Summary

Create a markdown summary organized by categories:

```markdown
# End of Day Update - {YYYY-MM-DD}

## Key Accomplishments
- [Main things completed today]

## Meetings
- [Meeting 1]: [Brief outcome/notes]
- [Meeting 2]: [Brief outcome/notes]

## Communications
- Emails: [Summary of important emails sent]
- Slack: [Key conversations/decisions]

## GitHub Activity
- **Commits**: [Summary of commits across repositories]
  - [repo-name]: [Key changes from commit messages]
- **Pull Requests**:
  - [Merged/Opened] [PR Title] in [repo-name]
- **Issues**:
  - [Opened] [Issue Title] in [repo-name]

## Other Work Items
- [Any other relevant work from Notion or elsewhere]

## Blockers & Challenges
- [Issues faced today]
- [Things needing attention or follow-up]

## Reflection / Notes
- [Learnings, thoughts, notes to self]
- [Things to remember]

## Tomorrow's Context

### Scheduled Meetings
- [Time] [Meeting title] with [Attendees]

### Priorities
- [Priority 1]
- [Priority 2]

### Weekly Goals Status
- [Goal 1] (Status: Complete/In Progress/Not Started)
- [Goal 2] (Status: Complete/In Progress/Not Started)
```

### Step 6: Save to File

Write the summary to `updates/{YYYY-MM-DD}-evening-update.md` using the Write tool.

Example filename: `updates/2026-01-08-evening-update.md`

### Step 7: Display Summary

Show the generated summary to the user and confirm the file was saved.

## Important Notes

- If any data source fails or times out, continue with the available data and note what was unavailable
- If `gh` CLI fails or isn't authenticated, note that GitHub data was unavailable and continue with other sources
- Prioritize quality over quantity - focus on meaningful work items, not every small action
- Keep the summary concise and actionable
- The Blockers & Challenges section should capture anything that slowed progress or needs escalation
- The Reflection section is for personal notes and learnings - encourage honest self-assessment
- The summary should be readable in 2-3 minutes
