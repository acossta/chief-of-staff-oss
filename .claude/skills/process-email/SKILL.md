---
name: process-email
description: Process received emails by categorizing them into action items (create todos) and quick responses (list for immediate reply)
---

# Process Email Skill

Process incoming emails by analyzing them and categorizing into:
1. **Action Required**: Emails that need follow-up tasks (creates todos in Notion)
2. **Quick Response**: Emails that can be replied to immediately (listed for quick handling)
3. **Informational**: Emails that are FYI only (no action needed)

## Execution Steps

### Step 1: Get Date Range

Use `mcp__googlecalendar__get_current_date_time` to get the current date. Calculate the date range for emails to process:
- Default: Last 24 hours (or since last processing)
- Gmail format: `YYYY/MM/DD` (e.g., `2026/01/08`)

### Step 2: Fetch Received Emails

Use `mcp__gmail__fetch_emails` to fetch emails received:

```
Query: "to:me after:YYYY/MM/DD" (excluding sent emails)
Parameters:
- max_results: 50
- include_payload: true
- verbose: true
```

For each email, extract:
- Message ID
- Subject line
- Sender (name and email)
- Received timestamp
- Body preview/content
- Thread ID (for context)
- Labels (to filter out already processed)

### Step 3: Categorize Emails

Analyze each email and categorize based on content signals:

**Action Required** (create todo):
- Contains questions directed at the user
- Requests for deliverables, documents, or information
- Meeting requests that need scheduling decisions
- Mentions deadlines or due dates
- Contains phrases like "please", "can you", "need you to", "by [date]"
- Follow-up requests

**Quick Response** (respond immediately):
- Simple yes/no questions
- Confirmation requests
- Short informational requests
- Thank you acknowledgments that warrant a brief reply
- Introductions that need a quick acknowledgment
- Scheduling confirmations

**Informational** (no action):
- Newsletters and subscriptions
- Automated notifications
- CC'd emails where user is not primary recipient
- FYI forwards
- Marketing emails

### Step 4: Ask for User Input

Use `AskUserQuestion` tool to:

1. **Confirm categorization**: "I've categorized [X] emails. Here's how they break down:
   - Action Required: [count]
   - Quick Response: [count]
   - Informational: [count]

   Would you like to review any category before I proceed?"

2. **Clarify ambiguous emails**: For emails that are unclear, ask:
   "This email from [sender] about '[subject]' could be either action-required or quick-response. Which would you prefer?"

3. **Confirm todo creation**: "I'm about to create [X] todos in Notion. Proceed?"

### Step 5: Create Todos for Action Items

For each "Action Required" email, create a todo in Notion:

Use `mcp__notion__notion-create-pages` with the Weekly Goals database (ID: f72f8046847b4c23bc81a465576d7842) or a dedicated Tasks database.

Todo format:
```
Title: [Email subject or extracted action]
Properties:
  - Status: Not Started
  - Source: Email from [sender]
  - Due Date: [if mentioned in email]
  - Context: [Brief summary of what's needed]
```

Include in the page content:
- Original email subject
- Sender information
- Key action items extracted
- Link to email thread (if available)

### Step 6: Generate Summary Output

Create a markdown summary:

```markdown
# Email Processing Summary - {YYYY-MM-DD HH:MM}

## Quick Response Needed ({count})
*These emails can be handled with a brief reply*

| # | From | Subject | Suggested Response |
|---|------|---------|-------------------|
| 1 | [sender] | [subject] | [1-line suggestion] |
| 2 | [sender] | [subject] | [1-line suggestion] |

## Action Items Created ({count})
*Todos have been created in Notion for these*

| # | From | Subject | Action Required | Due |
|---|------|---------|-----------------|-----|
| 1 | [sender] | [subject] | [extracted action] | [date if any] |
| 2 | [sender] | [subject] | [extracted action] | [date if any] |

## Informational / No Action ({count})
*These emails were reviewed and don't require action*

- [sender]: [subject] - [reason skipped]
- [sender]: [subject] - [reason skipped]

## Processing Stats
- Total emails processed: [X]
- Todos created: [X]
- Quick responses queued: [X]
- Skipped (informational): [X]
```

### Step 7: Offer Quick Response Assistance

After displaying the summary, offer to help draft quick responses:

"Would you like me to help draft responses for any of the Quick Response emails? I can:
1. Draft all quick responses for your review
2. Draft responses one at a time
3. Skip - I'll handle them myself"

If user chooses option 1 or 2, use `mcp__gmail__create_email_draft` to create drafts for each quick response email.

### Step 8: Save Processing Log (Optional)

Write a log to `updates/{YYYY-MM-DD}-email-processing.md` for reference.

## Email Analysis Heuristics

### High-Priority Signals (Action Required)
- Sender is in contacts or frequently communicated with
- Email is addressed directly to user (not CC'd)
- Contains action verbs: "please review", "need your input", "approve", "sign off"
- Contains time-sensitive language: "ASAP", "urgent", "by EOD", "deadline"
- Has attachments that need review
- Part of an ongoing thread where user's response is expected

### Quick Response Signals
- Short email body (< 100 words)
- Single question format
- Scheduling related ("does Tuesday work?")
- Confirmation requests ("can you confirm receipt?")
- Simple acknowledgments needed

### Skip Signals (Informational)
- Unsubscribe link present (newsletter)
- Sender is "noreply@" or similar
- High recipient count (mass email)
- Labels indicate promotional/social
- Automated system notifications

## Important Notes

- If Gmail API fails or times out, report the error and suggest manual email review
- Never auto-send emails - only create drafts for user review
- Respect email privacy - don't log full email contents, only metadata and summaries
- If Notion fails, display todos in the summary for manual creation
- Process emails in batches to avoid overwhelming the user
- Skip emails already labeled as processed (if using labels)
