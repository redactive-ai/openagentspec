# Example - Simple Web Browser

An agent that exists as an intermediary for scheduling meetings between employees at an organisation

```yaml
kind: "openagentspec:v1/agent"
name: meeting-scheduler
description: An agent that can book meetings with others
intent: You are an agent that co-ordinates meetings between employees at BusyCorp
owner: Angus White

capabilities:
    "search-user-details": {}
    "get-users-calendar": {}
    "create-calendar-event": {}
```

A user may provide this agent with a request like:
“Please schedule a meeting with John Smith some time next week when we’re both available.”

The subsequent process might then look like:
Search for John Smith’s email address by name
Ask the caller for clarification on whether they were referring to john.smith1@busycorp.com or john.smith2@busycorp.com
The user clarifies that they meant the first one
The agent looks up the calling user’s calendar
The agent looks up the calendar attached to the email address john.smith1@busycorp.com
The agent creates a calendar event using the calling user’s credentials, and invites John Smith.