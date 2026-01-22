# Apple PIM Plugin for Claude Code

Native macOS integration for Calendar, Reminders, and Contacts using EventKit and Contacts frameworks.

## Features

- **Calendar Management**: List calendars, create/read/update/delete events, search by date/title
- **Reminder Management**: List reminder lists, create/complete/update/delete reminders, search
- **Contact Management**: List groups, create/read/update/delete contacts, search by name/email/phone

## Prerequisites

- macOS 13.0 or later
- Swift 5.9 or later
- Node.js 18+ (for MCP server)
- Calendar, Reminders, and Contacts permissions granted to Terminal/Claude Code

## Installation

### 1. Build Swift CLI Tools

```bash
cd swift
swift build -c release
```

This creates three binaries in `.build/release/`:
- `calendar-cli`
- `reminder-cli`
- `contacts-cli`

### 2. Install MCP Server Dependencies

```bash
cd mcp-server
npm install
```

### 3. Grant Permissions

On first run, macOS will prompt for Calendar, Reminders, and Contacts access. Grant these permissions.

## Commands

### `/apple-pim:calendars`

Manage calendar events.

```
/apple-pim:calendars list                    # List all calendars
/apple-pim:calendars events --from today --to "next week"
/apple-pim:calendars search "team meeting"
/apple-pim:calendars create --title "Lunch" --start "tomorrow 12pm" --duration 1h
```

### `/apple-pim:reminders`

Manage reminders.

```
/apple-pim:reminders lists                   # List all reminder lists
/apple-pim:reminders items --list "Personal"
/apple-pim:reminders create --title "Buy groceries" --due "tomorrow 5pm"
/apple-pim:reminders complete --id <id>
```

### `/apple-pim:contacts`

Manage contacts.

```
/apple-pim:contacts groups                   # List contact groups
/apple-pim:contacts search "John"
/apple-pim:contacts get --id <id>
/apple-pim:contacts create --name "Jane Doe" --email "jane@example.com"
```

## MCP Tools

The plugin exposes these MCP tools:

| Tool | Description |
|------|-------------|
| `calendar_list` | List all calendars |
| `calendar_events` | List/search events |
| `calendar_create` | Create a new event |
| `calendar_update` | Update an event |
| `calendar_delete` | Delete an event |
| `reminder_lists` | List reminder lists |
| `reminder_items` | List/search reminders |
| `reminder_create` | Create a reminder |
| `reminder_complete` | Mark reminder complete |
| `reminder_update` | Update a reminder |
| `reminder_delete` | Delete a reminder |
| `contact_groups` | List contact groups |
| `contact_list` | List/search contacts |
| `contact_get` | Get contact details |
| `contact_create` | Create a contact |
| `contact_update` | Update a contact |
| `contact_delete` | Delete a contact |

## Agent

The `pim-assistant` agent triggers proactively when you mention:
- Scheduling, appointments, meetings, events
- Reminders, tasks, todos, "remind me"
- Contacts, people, email/phone lookups

## Troubleshooting

### Permission Denied

If you get permission errors, check System Settings > Privacy & Security:
- Calendars: Ensure Terminal/Claude Code has access
- Reminders: Ensure Terminal/Claude Code has access
- Contacts: Ensure Terminal/Claude Code has access

### CLI Not Found

Ensure you've built the Swift package:
```bash
cd swift && swift build -c release
```

## License

MIT
