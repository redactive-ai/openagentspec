# Example - Multiple Agents Communicating Internally

Consider an agent at BusyCorp that functions as an executive assistant to the Human Resources department. It is able to schedule meetings between members of the team, and access the team’s knowledge base to be able to automatically populate meeting descriptions. It makes use of existing agents to achieve this..

```yaml
kind: "openagentspec:v1/agent"
name: hr-executive-assistant
description: An EA agent for the HR department
intent: You are an agent that functions as an executive assistant to the Human Resources department at BusyCorp.
owner: Example User 1

capabilities:
    "oagent://hr-agent": {}
    "oagent://meeting-scheduler": {}
```

If a member of the HR department were to contact this agent with the prompt “Please organise an hour-long meeting next week for the team to organise our roadmap for the next quarter”, the agent may respond like the following:

The hr-executive-assistant contacts the hr-question-answerer with the prompt “What are the key tasks that the team needs to achieve in the next quarter?”
The hr-question-answerer responds with “The following bullet points describe the key tasks for the HR team at BusyCorp in the next quarter: …”
The hr-executive-assistant contacts the hr-question-answerer with the prompt “Who are the key stakeholders in determining the roadmap for the next quarter?”
The hr-question-answerer responds with “Here is a list of the names of team members who should be involved in determining next quarter’s roadmap: …”
The hr-executive-assistant contacts the meeting-scheduler agent with the prompt “Please organise an hour-long meeting next week with the following attendees: … Please include the following information in the meeting description: …”
The meeting-scheduler creates the corresponding calendar invitation.
The hr-executive-assistant responds to the original user: “The meeting has been created. The following people were invited: … Is there anything else I can help you with?”
Note that all operations in this workflow will be logged against the original calling user for audit purposes.