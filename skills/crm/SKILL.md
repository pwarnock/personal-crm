---
name: Personal CRM
description: This skill should be used when the user mentions "met with", "had coffee with", "breakfast with", "lunch with", "dinner with", "meeting with", asks to "log event", "capture meetup", "add to CRM", "new contact", or describes an interaction with people that should be recorded. Handles natural language and voice transcripts to create structured CRM entries.
version: 0.1.0
---

# Personal CRM Skill

Capture meetups, events, and contacts from natural language input. Process voice transcripts, quick notes, or conversational descriptions into structured Notion CRM entries.

## Prerequisites

Set environment variables (see `references/schema.md` for setup):
- `PERSONAL_CRM_EVENTS_DB` - Events database collection URL
- `PERSONAL_CRM_CONTACTS_DB` - Contacts database collection URL

## Workflow Overview

1. **Parse** natural language for event details and attendees
2. **Resolve** attendee names to existing contacts
3. **Clarify** ambiguous matches with user
4. **Create** event with linked attendees
5. **Optionally** create new contacts for unknown people

## Step 1: Parse Input

Extract from user's input:
- **Event name**: The activity (coffee, lunch, meeting, etc.)
- **Date**: Relative ("yesterday", "last Saturday") or explicit
- **Attendees**: Names mentioned
- **Notes**: Any context about what was discussed
- **New contacts**: People not yet in CRM

Handle messy voice transcripts gracefully - filter filler words ("um", "uh"), extract meaningful content.

## Step 2: Resolve Contacts

For each attendee name:

1. Search the Contacts database using `notion-search` with:
   ```
   query: "contact name"
   data_source_url: $PERSONAL_CRM_CONTACTS_DB
   ```

2. Handle results:
   - **Single match**: Use that contact
   - **Multiple matches**: Ask user to choose (see Ambiguity Handling)
   - **No match**: Offer to create new contact or search differently

## Step 3: Handle Ambiguity

When multiple contacts match a name, use AskUserQuestion:

```
Question: "Which Brian?"
Options:
- Brian Pollard
- Brian Arizaga
- Both
```

Provide enough context (company, recent interaction) to help user choose quickly.

## Step 4: Create Event

Use `notion-create-pages` with the Events database:

```
parent: {data_source_id: <ID from $PERSONAL_CRM_EVENTS_DB>}
pages: [{
  properties: {
    Name: "event name",
    "date:Event Date:start": "YYYY-MM-DD",
    "date:Event Date:is_datetime": 0,
    Attendees: "[\"https://notion.so/contact-id\", ...]"
  }
}]
```

Note: Extract just the UUID from the collection URL (remove `collection://` prefix).

## Step 5: Create New Contacts (Optional)

When user mentions someone not in CRM:

1. Ask if they want to add them as a contact
2. Extract any mentioned details (company, role)
3. Create contact using `notion-create-pages` with Contacts database:
   ```
   parent: {data_source_id: <ID from $PERSONAL_CRM_CONTACTS_DB>}
   ```

## Date Parsing

Convert relative dates to absolute:
- "yesterday" → subtract 1 day from today
- "last Saturday" → most recent past Saturday
- "this morning" → today
- "Tuesday" → next occurrence (or current week if not passed)

## Quick Reference

| Task | MCP Tool | Data Source |
|------|----------|-------------|
| Search contacts | `notion-search` | `$PERSONAL_CRM_CONTACTS_DB` |
| Create event | `notion-create-pages` | `$PERSONAL_CRM_EVENTS_DB` |
| Create contact | `notion-create-pages` | `$PERSONAL_CRM_CONTACTS_DB` |

## Confirmation Pattern

After creating entries, confirm with user:
- Event created: name, date, attendee count
- Link to Notion page
- Any new contacts created

## Additional Resources

### Reference Files
- **`references/schema.md`** - Configuration, database schema, MCP tool details

### Examples
- **`examples/sample-inputs.md`** - Natural language parsing examples and expected outputs
