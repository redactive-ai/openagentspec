# Example - Multiple Agents Communicating Externally

Provided both agents implement the OpenAgentSpec standard, the server will be able to transparently co-ordinate communication between them by retrieving each agent’s spec document. However, the risk posture of this case is a little different.

Imagine that BusyCorp contracts their IT support to an independent organisation called CompuHelp. CompuHelp might expose an agent to their clients that is able to provide automatic tech support. Its OpenAgentSpec document might look like:

```yaml
kind: "openagentspec:v1/agent"
name: tech-support
description: Our externally facing tech support agent
intent: You are a tech support agent at CompuHelp. You respond to requests from external clients. Try to answer questions directly, and if unable to do so, create a support ticket to be addressed by our staff.
owner: Angus White

capabilities:
    "search-faqs":
        input_restriction:
            assertion: recent.search-faqs.inputs["path"].startsWith("public")
    "create-ticket":
        user_identity: False
        static_identity: API-KEY

exposes:
    ticket_id: recent.create-ticket.outputs["ticket_id"]
    checked_pages: recent.search-faqs.outputs["pages"]
```

CompuHelp hosts this spec on their website at: `https://www.compuhelp.com/agents/tech-support`.

Note the addition of the exposes key: these are the two values that will be returned to the caller. We want to ensure no other data is leaked to the calling agent about the information used to fulfil the request.

BusyCorp might have an agent that reports on the status of internal services, and raises a support ticket if they appear to be down:

```yaml
kind: "openagentspec:v1/agent"
name: service-report
description: The agent in charge of responding to service outages
intent: You are an agent at BusyCorp. Respond to user queries about the status of IT services. Services are either DOWN or UP. If a service is DOWN, please raise a ticket with CompuHelp.
owner: Angus White

capabilities:
    "get-service-status":
        user_identity: False
    "oagent://www.compuhelp.com/agents/tech-support":
        user_identity: False
        static_identity: API-KEY

exposes:
    ticket_id: recent.create-ticket.outputs["ticket_id"]
    checked_pages: recent.search-faqs.outputs["pages"]
```

An interaction with this agent might look like this:

A user, Alice, asks the service-report agent “what is the status of the Foo service?”
The service-report agent checks, and discovers that the Foo service is down.
The service-report agent contacts the CompuHelp client-support agent and asks “I work for BusyCorp, and have noticed our Foo service isn’t functioning. Is there a fix?””
The CompuHelp client-support agent is unable to answer the question directly, and so creates a ticket.
The CompuHelp agent responds to the service-report agent: “I was unable to find an answer to your question, but have created a support ticket to resolve this. The tracking number is: #####”
The service-report agent responds to Alice: “The Foo service is currently down. A ticket has been raised with CompuHelp to resolve. The tracking number is: ####”
