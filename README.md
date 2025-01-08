# Open Agent Spec

At Redactive, we have extensive experience in developing, deploying, and maintaining generative AI applications. In this RFC, we define what a Generative AI Agent, or “Agent” is, how they can be defined safely and securely, and outline a formal standard to provide consistent expectations for agentic workflows.

## Benefits

Agents implementing the OpenAgentSpec are able to:
* 👥 Be used by multiple authorised users (multi-tenant agents)
* 🔐 Interact with highly sensitive content (secure agents)
* 🦺 Autonomously perform actions safely (safe agents)
* 🤝 Work together with other agents:
  * 🏢 with differing authorisation boundaries
  * 📡 across organisational boundaries  

## The Standard

* 🤖 [Description of Standard](/proposal.md) - The OpenAgentSpec Standard - *Start here!*
* 📐 [Specification](/specification.md) - Technical Specification for implementing the OpenAgentSpec
* 📌 [Examples of Agents](/examples)
* ⛰️ [Background](/background.md) - A primer on some background concepts

By taking an opinionated stance on what an Agent is, we also aim to:
* Assist consumers and enterprises to onboard agents securely
* Establish best practises for platform providers to integrate with
* Build consistent expectations of how an agent should operate

## Why OpenAgentSpec?

The current approach to building AI agents is to write bespoke applications that combine LLMs with other explicitly programmed behaviours. This restricts the adoption of AI Agents to those that know how to program, makes it opaque to reason about how secure an Agent is, and means Agents are incompatible with each other. OpenAgentSpec democratises access to AI by providing a representation for non-programmers to understand and reason about what an Agent is allowed to do. Additionally OpenAgentSpec defines an interface for Agents to communicate with each other, greatly increasing their impact.

![The delta between the current security posture of agents and the proposed future state of OpenSpecAgents](/images/overview.png)

## Contributing

We are actively seeking community and industry feedback and involvement, please get involved by starting a [discussion](https://github.com/redactive-ai/openagentspec/discussions)! 💬

We aim to maintain this specification in an open fashion, including hosting requests for comments amongst industry leaders, and ensuring all future changes are versioned semantically as ideas evolve.