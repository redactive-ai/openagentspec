# Open Agent Spec

At Redactive, we have extensive experience in developing, deploying, and maintaining generative AI applications. In this RFC, we define what a Generative AI Agent, or “Agent” is, how they can be defined safely and securely, and outline a formal standard to provide consistent expectations for agentic workflows.

## The Standard

* 🤖 [Description of Standard](/proposal.md) - The OpenAgentSpec Standard (*Start here*)
* 📐 [Specification](/specification.md) - Technical Specification for implementing the OpenAgentSpec

## Benefits

Agents that follow the OpenAgentSpec are able to:
* Be used by multiple authorised users (multi-tenant agents)
* Interact with highly sensitive content (secure agents)
* Autonomously perform actions safely (safe agents)
* Communicate with other agents:
  * with differing authorisation boundaries
  * across organisational boundaries

By taking an opinionated stance on what an Agent is, we also aim to:
* Assist consumers and enterprises to onboard agents securely
* Establish best practises for platform providers to integrate with
* Build consistent expectations of how an agent should operate

![The delta between the current security posture of agents and the proposed future state of OpenSpecAgents](/images/overview.png)

## Contributing

We are actively seeking community and industry feedback and involvement, please get involved by starting a [discussion](https://github.com/redactive-ai/openagentspec/discussions)! 💬

We aim to maintain this specification in an open fashion, including hosting requests for comments amongst industry leaders, and ensuring all future changes are versioned semantically as ideas evolve.