---
name: crm
description: This skill should be used when the user mentions "met with", "had coffee with", "breakfast with", "lunch with", "dinner with", "meeting with", asks to "log event", "capture meetup", "add to CRM", "new contact", or describes an interaction with people that should be recorded. Handles natural language and voice transcripts to create structured CRM entries.
---

# Personal CRM Skill

Capture meetups, events, and contacts from natural language input into Notion.

## Quick Start

When user describes a meetup or interaction:

1. **Parse** the input for: event type, date, attendee names, notes
2. **Search** contacts using `mcp__plugin_Notion_notion__notion-search`
3. **Resolve** ambiguous matches by asking user to choose
4. **Create** event using `mcp__plugin_Notion_notion__notion-create-pages`
5. **Confirm** what was created

## Step 1: Parse Input

Extract from user's message:
- **Event name**: The activity (coffee, lunch, meeting, etc.)
- **Date**: Convert relative dates ("yesterday", "last Saturday") to YYYY-MM-DD format
- **Attendees**: All names mentioned
- **Notes**: Any additional context

Date conversion examples:
- "yesterday" → subtract 1 day from today
- "today" → today's date
- "last Saturday" → most recent past Saturday

## Step 2: Search Contacts

For EACH attendee name, search the contacts database:

```
Tool: mcp__plugin_Notion_notion__notion-search
Parameters:
  query: "<attendee name>"
  data_source_url: $PERSONAL_CRM_CONTACTS_DB
```

The environment variable `PERSONAL_CRM_CONTACTS_DB` contains the collection URL (e.g., `collection://2776e226-b9f6-8154-abb5-000b7b90f6f7`).

## Step 3: Handle Results

For each search result:

- **Single match**: Use that contact's page URL
- **Multiple matches**: Ask user which one using AskUserQuestion tool
- **No match**: Ask if user wants to create a new contact

When asking about ambiguous names:
```
Question: "Which Brian did you meet with?"
Options:
- Brian Pollard (Acme Corp)
- Brian Smith (TechStart)
- Both of them
```

## Step 4: Create Event

Once all attendees are resolved, create the event:

```
Tool: mcp__plugin_Notion_notion__notion-create-pages
Parameters:
  parent: {"data_source_id": "<ID from PERSONAL_CRM_EVENTS_DB without collection:// prefix>"}
  pages: [{
    "properties": {
      "Name": "<event description>",
      "date:Event Date:start": "<YYYY-MM-DD>",
      "date:Event Date:is_datetime": 0,
      "Attendees": "[\"https://www.notion.so/<contact-page-id>\", ...]"
    }
  }]
```

Extract the UUID from `$PERSONAL_CRM_EVENTS_DB` by removing the `collection://` prefix.

Format Attendees as a JSON array of Notion page URLs.

## Step 5: Create New Contacts (if needed)

When user confirms they want to add someone new:

```
Tool: mcp__plugin_Notion_notion__notion-create-pages
Parameters:
  parent: {"data_source_id": "<ID from PERSONAL_CRM_CONTACTS_DB without collection:// prefix>"}
  pages: [{
    "properties": {
      "Name": "<full name>"
    }
  }]
```

## Step 6: Confirm

After creating entries, tell the user:
- Event name and date
- Number of attendees linked
- Any new contacts created
- Link to the Notion page if available

## Environment Variables

Required:
- `PERSONAL_CRM_EVENTS_DB` - Events database collection URL
- `PERSONAL_CRM_CONTACTS_DB` - Contacts database collection URL

## Examples

Input: "had coffee with Alice and Bob yesterday"
→ Parse: event="coffee", date=yesterday, attendees=["Alice", "Bob"]
→ Search each name in contacts
→ Create event with resolved attendees

Input: "met with Brian for lunch last Friday"
→ If multiple Brians found, ask which one
→ Create event after user selects
