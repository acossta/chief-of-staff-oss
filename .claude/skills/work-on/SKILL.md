---
name: work-on
description: Start a focused work session by selecting a todo, blocking calendar time, and marking it in progress
user-invocable: true
---

# Work On Skill

Start a focused work session by selecting a task to work on, blocking time on your calendar, and marking the task as "In Progress" in Notion.

## Usage

```
/work-on [task] [duration]
```

**Arguments:**
- `[task]` - Optional. Description or partial match of the task to work on
- `[duration]` - Optional. Duration of the work session: `30m`, `1h`, or `2h`

**Examples:**
- `/work-on` - Prompts for task selection and duration
- `/work-on investor update` - Works on matching task, prompts for duration
- `/work-on 2h` - Prompts for task, uses 2 hour duration
- `/work-on investor update 1h` - Works on matching task for 1 hour

## Execution Steps

### Step 1: Parse Arguments

Parse the provided arguments to identify:
- **Task description**: Any text that isn't a duration pattern
- **Duration**: Look for patterns like `30m`, `1h`, `2h`, `30min`, `1hr`, `2hr`, `1 hour`, `2 hours`

Convert duration to minutes:
- `30m` / `30min` / `30 minutes` → 30 minutes
- `1h` / `1hr` / `1 hour` → 60 minutes
- `2h` / `2hr` / `2 hours` → 120 minutes

### Step 2: Fetch Current Week's Page from Notion

The Weekly Goals database contains weekly pages with tasks as checkboxes in the page content.

```
Database ID: f72f8046847b4c23bc81a465576d7842
```

**Process:**
1. Fetch the database to find the current week's page (e.g., "Week of Jan 6, 2026")
2. Fetch that page's content using `mcp__notion__notion-fetch`
3. Parse the markdown content to extract checkbox items

**Checkbox format in Notion:**
```markdown
- [ ] Task name (unchecked/not started)
- [x] Task name (checked/completed)
- [ ] 🔄 Task name (in progress - marked by this skill)
- [ ] ⏳ Task name (waiting/blocked - can be set manually)
```

Tasks are stored as checkboxes within the weekly page content, NOT as separate Notion pages.

### Step 3: Select Task (if needed)

Parse the checkboxes from the page content:
- Look for lines matching `- [ ]` (unchecked tasks)
- Also include `- [ ] 🔄` tasks (currently in progress)
- Exclude `- [x]` tasks (already completed)

If a task description was provided, filter/match against the task text.

If no task was specified OR multiple matches found:
- Present the list of available tasks to the user
- Use `AskUserQuestion` to let them select which task to work on
- Show task name and whether it's already in progress (has 🔄)

### Step 4: Select Duration (if needed)

If no duration was specified:
- Use `AskUserQuestion` with options:
  - "1 hour (Recommended)" - default, good for deep work
  - "30 minutes" - quick task or getting started
  - "2 hours" - extended focus session

### Step 5: Get Current Time and Check Calendar

1. Get current date/time using `mcp__googlecalendar__get_current_date_time`
2. Calculate end time based on selected duration
3. Use `mcp__googlecalendar__find_free_slots` to check availability:
   - `time_min`: current time (RFC3339 format)
   - `time_max`: current time + 4 hours
   - `items`: ["primary"]

If there's a conflict within the requested duration:
- Inform the user about the conflict
- Suggest the next available slot
- Ask if they want to proceed anyway or pick a different time

### Step 6: Create Calendar Block

Use `mcp__googlecalendar__create_event` with:
- `calendar_id`: "primary"
- `summary`: "Working on: [task name]"
- `start_datetime`: current time in naive format (YYYY-MM-DDTHH:MM:SS)
- `event_duration_hour`: hours portion of duration
- `event_duration_minutes`: remaining minutes (0-59)
- `timezone`: user's timezone (from current time response)
- `description`: "Focus time for: [task name]"

### Step 7: Mark Task as In Progress in Notion

Use `mcp__notion__notion-update-page` with `replace_content_range` to add the 🔄 indicator:

**Before:**
```markdown
- [ ] Task name
```

**After:**
```markdown
- [ ] 🔄 Task name
```

Use:
- `page_id`: the weekly page's ID
- `command`: "replace_content_range"
- `selection_with_ellipsis`: "- [ ] Task name" (the original checkbox line)
- `new_str`: "- [ ] 🔄 Task name" (with the in-progress indicator)

**Important:** Match the exact text including indentation. If the task already has 🔄, skip this step.

### Step 8: Confirm to User

Display a summary:

```
Started work session:
- Task: [task name]
- Duration: [duration]
- Calendar: Blocked [start time] - [end time]
- Status: Marked with 🔄 in Notion
```

## Clearing In Progress Status

When a task is completed, the user should:
1. Check the checkbox in Notion (changes `- [ ]` to `- [x]`)
2. The 🔄 indicator will remain but the task shows as complete

Or manually remove the 🔄 if abandoning the task.

## Error Handling

### Notion Fetch Fails
- Offer to enter the task name manually
- Create the calendar block with the manual task name
- Note that Notion couldn't be updated

### Calendar Conflict
- Show the conflicting event(s)
- Suggest the next available slot that fits the duration
- Ask user: proceed anyway, pick suggested time, or cancel

### Notion Update Fails
- Still create the calendar block (don't fail the whole flow)
- Inform user that the calendar is blocked but Notion wasn't updated
- Suggest they manually add 🔄 to the task in Notion

### No Tasks Found
- If no tasks match the search or no unchecked tasks exist:
- Ask user if they want to enter a task name manually
- Create calendar block with manual task name (skip Notion update)

### Task Already In Progress
- If the selected task already has 🔄:
- Inform the user it's already marked as in progress
- Still create the calendar block
- Skip the Notion content update

## Defaults

| Setting | Default Value |
|---------|---------------|
| Todo source | Weekly Goals database (current week's page) |
| Calendar | Primary calendar |
| Event title | "Working on: [task name]" |
| Duration options | 30m, 1h, 2h |
| Recommended duration | 1 hour |
| In progress indicator | 🔄 emoji prefix |
