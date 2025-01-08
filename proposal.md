# Proposal

## Agents & Tools

A general definition of an AI agent is the combination of:
* A Generative AI model (for example, a Large Language Model or LLM) with;
* Tools that allow it to interface with the outside world, within
* A set of boundaries that it is authorised to act.

We define a tool as any algorithm or function that may be leveraged by an agent. An example is: a REST endpoint, a SQL database connector or any custom code. We define tools as being completely generic and flexible. A *tool*:
* Has some structured parameters
* Provides some structured data as output
* Has a natural language description of its function and parameters
* Has a well-defined relationship between its input and output[^1]
[^1]: Although this relationship is not necessarily deterministic, it is well defined in the sense that it is explainable, and auditable.

![Characteristics of a tool](/images/tool.png)

This is distinct from an *agent* which:
* Is prompted with some natural language
* Responds primarily in natural language
* Has a generative relationship between input and output

![Characteristics of an agent](/images/agent.png)

### Current Agents are High Risk

Current agents provide generative models the ability to use tools without any constraints or security guarantees. The current interaction between a user, an agent, and a suite of tools is:

![Unconstrained Agent Flow](/images/unconstrained_agent_flow.png)

1. A user query (prompt) is submitted to the agent.
2. The Agent uses the generative model to reason about what tools may be useful to fulfil the request.
3. The Agent uses the generative model to generate an appropriate input to provide to the first tool required.
4. The Agent invokes the tool.
5. The Agent uses the generative model to interpret the tool response, and determine if the user’s request has been fulfilled.
   * If so, the Agent responds to the original request.
   * If not, the Agent returns to step 2.

Providing agents the ability to use tools without constraints like this leaves the risk that the agent may use a tool inappropriately, and is a significant security concern. Agents need to handle authentication and provide strict guarantees about how tools will be used to overcome these concerns.

### Secure Agents need Constraints

A secure Agent needs to provide guarantees that it will only utilise tools within a well-defined set of constraints. We define a secure agent as having the following interaction:

![Secure Agent Flow](/images/secure_agent_flow.png)

1. A user query (prompt) is submitted to the agent.
2. The user query is validated by the input guardrail
3. The Agent uses the generative model to reason about what tools may be useful to fulfil the request.
4. The Agent uses the generative model to generate an appropriate input to provide to the first tool required.
5. The Agent validates that invoking the tool in the way the generative model describes falls within its given constraints.
6. The Agent invokes the tool.
7. The Agent uses the generative model to interpret the tool response, and determine if the user’s request has been fulfilled.
   * If so, the Agent responds to the original request.
   * If not, the Agent returns to step 2.
8. The agent response is validated against the output guardrail

The agent is constrained by *guardrails* (steps 2 & 8) and *tool restrictions* (steps 5 & 7). Guardrails provide guarantees on the agent’s natural language input and output. Tool restrictions provide guarantees on tool use, by referencing some well-defined state of the agent. We define this state and its lifecycle as an *engagement*.

## Engagements

In order to provide well-defined state to condition the tool restrictions on, we define an *engagement*. An engagement is a single task a user has asked of a given agent, and have the following lifecycle:
1. An engagement is created when a user first instructs an agent to perform a task.
2. An engagement is updated as an agent operates tools, and as the user provides additional input to perform the given task.
3. An engagement concludes when the agent completes the task, or reaches a failure state.

![The lifecycle of an Engagement](/images/engagement_lifecycle.png)

An engagement contains the well-defined state for the agent to condition against, including: 
* The inputs of all prior tools the agent has executed,
* The outputs received from all prior tools the agent has executed, and;
* The identity of the user.

## Identity & Authorisation

There are two identities involved in the execution of an agent workflow:
* The *engagement user* (the user interacting with the agent, for whom the agent is performing the task)
* The *agent owner* (the agent author, who defined the goals and operational boundaries of the agent)

The agent always invokes a tool using credentials belonging to either the engagement user, or the agent owner[^2]. The state of an engagement contains the engagement user, enabling the agent creator to constrain an agent based on who is interacting with the agent.
[^2]: The agent may invoke a tool using service account credentials as a proxy of the agent owner.

![Using a tool that understands user identities](/images/tool_use_with_user_identity.png)

Best practice is for the agent to always operate within the scope of the engagement user. When an agent needs to use a tool, it appears to the tool that the request is coming from the agent on behalf of the engagement user[^3].
[^3]: In OAuth2, this type of flow is known as 3-Legged OAuth.

![Using a tool that requires a static identity](/images/tool_use_with_static_identity.png)

The alternative is for the agent to execute tools using a *static identity*. When a tool doesn’t have an understanding of the engagement user identities, the agent owner can allow the agent to leverage a static identity to invoke the tool. Tool restrictions are particularly important in these cases.

The combination of engagement user and static identity also secures agent-to-agent workflows.

## Composing Agents

We can interpret an agent as a tool to provide the same security and safety boundaries to agent-to-agent communication that we have now defined for tool use. 

Although an agent accepts and answers with natural language, the engagement provides a deterministic state which an agent can utilise as structured output. Exposing a subset of this structure enables an agent to be leveraged like any other tool.

![Composing multiple agents together](/images/agent_composition.png)

Agents can be configured to return deterministic outputs, and other agents can constrain their use, both for agents that are co-located on the same server and those running across servers on the internet more broadly. To further aid in agent-to-agent composition, we define a discovery mechanism and RESTful communication protocol.

## Agent OS

We have now described all the key elements of the OpenAgentSpec. Agents have engagements; can use tools and communicate with other agents within a set of restrictions; and delegate identity. To bring all of these concepts together into a concrete form, we provide a server implementation of the OpenAgentSpec, which we call Agent OS. 

Agent OS is written in Python, primarily built upon semantic kernel, and available on github at:
	https://github.com/redactive-ai/agent-os 

Using Agent OS is not a requirement of the OpenAgentSpec, and any server that correctly orchestrates a single or multiple agent’s execution as described by their given OpenAgentSpec is said to be implementing the specification. Optionally, for an implementation to enable its agents to be composed by other agents running on different server implementations, the implementation may expose a series of endpoints as defined in the server section of the specification.
