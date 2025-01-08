# Example - HR Agent

Consider an agent that answers questions about the internal processes of BusyCorp’s Human Resources department. To ensure relevancy of the information retrieved by the agent to answer the questions, we place restrictions on its use of tools:


```yaml
kind: "openagentspec:v1/agent"
name: hr-agent
description: An agent that can answer HR related questions
intent: You are an agent that answers questions for the Human Resources department of BusyCorp.
owner: Matthew Timms

capabilities:
    "search-knowledge-base":
        input_restriction:
            assertion: recent.search-knowledge-base.inputs["knowledge_base_id"].startsWith("Busycorp/HR/")
```
