# Personal CRM

A Claude Code plugin for capturing meetups, contacts, and interactions from natural language into your Notion CRM.

## Features

- **Natural language input**: "met Brian and Greg for coffee yesterday"
- **Voice transcript support**: Handles messy transcribed audio
- **Smart contact resolution**: Asks for clarification when names are ambiguous
- **Automatic date parsing**: "last Saturday", "yesterday", "this morning"
- **New contact creation**: Add people you just met

## Installation

### From Marketplace

```
/plugin marketplace add pwarnock/cc-plugins
/plugin install personal-crm@cc-plugins
```

### Direct Install

```
/plugin marketplace add https://github.com/pwarnock/personal-crm.git
```

## Configuration

Set these environment variables with your Notion database IDs:

```bash
export PERSONAL_CRM_EVENTS_DB="collection://YOUR-EVENTS-DATABASE-ID"
export PERSONAL_CRM_CONTACTS_DB="collection://YOUR-CONTACTS-DATABASE-ID"
```

### Finding Your Database IDs

1. Install the [Notion plugin](https://github.com/makenotion/claude-code-notion-plugin) for Claude Code
2. Connect the MCP: `/mcp` and authenticate with Notion
3. Ask Claude to fetch your Personal CRM page using the Notion MCP
4. Look for `<data-source url="collection://...">` tags in the response
5. Copy the collection URLs for your Events and Contacts databases

### Expected Notion Structure

Your Personal CRM should have:

**Events Database**
| Property | Type |
|----------|------|
| Name | title |
| Event Date | date |
| Attendees | relation (to Contacts) |

**Contacts Database**
| Property | Type |
|----------|------|
| Name | title |
| (optional) | Company, Email, etc. |

## Usage

Just describe your meetup naturally:

```
"had coffee with Alice and Bob yesterday"
"breakfast meeting with the team last Saturday"
"met Sarah Chen at the conference - she's a PM at Stripe"
```

The skill will:
1. Parse the event details
2. Search your contacts
3. Ask for clarification if needed (e.g., "Which Brian?")
4. Create the event in Notion with linked attendees

## License

MIT
