# Example - Web Browser

A simple Agent that is configured to browse the internet without authentication

```yaml
kind: "openagentspec:v1/agent"
name: web-browser
description: An agent that can browse the internet
intent: You are an agent that browses the internet on behalf of users. Please assist in fulfilling user requests.
owner: Lucas Sargent

capabilities:
    "generic-http-tool":
        user_identity: False
```
