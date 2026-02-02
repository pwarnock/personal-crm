# Personal CRM Schema Reference

## Configuration

Set these environment variables before using the skill:

```bash
export PERSONAL_CRM_EVENTS_DB="collection://YOUR-EVENTS-DATABASE-ID"
export PERSONAL_CRM_CONTACTS_DB="collection://YOUR-CONTACTS-DATABASE-ID"
```

### Finding Your Database IDs

1. Open your Personal CRM page in Notion
2. Use `notion-fetch` to get the page content
3. Look for `<data-source url="collection://...">` tags
4. Copy the collection URLs for Events and Contacts databases

## Events Schema

| Property | Type | Description |
|----------|------|-------------|
| Name | title | Event name/description |
| Event Date | date | When the event occurred |
| Attendees | relation | Links to Contacts database |

### Creating an Event

Use `notion-create-pages` with:
```
parent: {data_source_id: $PERSONAL_CRM_EVENTS_DB (without collection:// prefix)}
pages: [{
  properties: {
    Name: "Event title here",
    "date:Event Date:start": "YYYY-MM-DD",
    "date:Event Date:is_datetime": 0,
    Attendees: ["https://www.notion.so/contact-page-id-1", "https://www.notion.so/contact-page-id-2"]
  }
}]
```

## Contacts Schema

| Property | Type | Description |
|----------|------|-------------|
| Name | title | Contact's full name |
| (other properties) | varies | Company, email, etc. |

### Looking Up Contacts

1. Use `notion-search` with `data_source_url: $PERSONAL_CRM_CONTACTS_DB`
2. Search by name (supports partial matching)
3. Extract page URL from results for use in Attendees relation

### Handling Ambiguous Names

When multiple contacts match a name (e.g., two "Brian"s):
1. Present options to user with full names
2. Use AskUserQuestion tool with contact options
3. Proceed with selected contact(s)

## MCP Tools Reference

### Search Contacts
```
Tool: mcp__plugin_Notion_notion__notion-search
Parameters:
  query: "contact name"
  data_source_url: $PERSONAL_CRM_CONTACTS_DB
```

### Create Event
```
Tool: mcp__plugin_Notion_notion__notion-create-pages
Parameters:
  parent: {data_source_id: <ID from $PERSONAL_CRM_EVENTS_DB>}
  pages: [{properties: {...}, content: "..."}]
```

### Create Contact
```
Tool: mcp__plugin_Notion_notion__notion-create-pages
Parameters:
  parent: {data_source_id: <ID from $PERSONAL_CRM_CONTACTS_DB>}
  pages: [{properties: {Name: "Full Name"}}]
```
