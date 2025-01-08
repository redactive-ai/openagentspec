# Background
## What is an AI Agent?
Like a lot of terminology in tech, “agent” is a word that has been plucked from its prior common language usage, and elevated beyond its peers into a new meaning in the age of AI. With a nod to its humble beginnings in legal parlance, we can frame a modern definition for an AI agent as:  “an AI agent is an AI appointed and authorised to act on behalf of another person”.^[4]
[^4]: https://www.oxfordreference.com/display/10.1093/acref/9780195557558.001.0001/acref-9780195557558-e-0136?rskey=Wa6RdF&result=121

This definition helps us frame that an agent itself is never completely responsible for its actions, but rather only responsible for acting on a given instruction (on behalf of another person); within a well-defined set of rules. This “set of rules”, previously, may have been a legal clause, but in our new agentic age we require a new, flexible and tech-friendly way of defining these operational boundaries. From the perspective of an agent within an organisation; the user asking the agent to perform a task is responsible for the outcome of that task, irrespective of the agent's involvement.

Following on from this, we can conclude that any implementation of an AI agent must then satisfy the following constraints:
* An agent must have a clear and well-defined goal.
* The “set of rules” that an agent will operate within must be similarly clear and well-defined.
* Both of the above must be clearly communicated to any prospective user.

Providing the tools to clearly express this information is the key outcome of the specification  outlined in this RFC.

## Agent Implementation
Without limiting our definition of an agent too much, we can inform our specification by making some reasonable assumptions about how it is likely to run. Namely, we assume that the average agent will interface with an LLM for natural reasoning steps, and be able to leverage external tools by executing code. Let’s summarise these requirements as:
* An LLM; and,
* A series of Tools (code executions)

In addition to this, while generative AI is a fresh, formidable technology; we mustn't forget our valuable foundations from decades of traditional software engineering. We additionally require:
* Observability
* Auditability
* Access Management

Together, these form the requirements that we explicitly wish to address with the OpenAgentSpec.

## Prior Art
The space of generative AI agents and application development is vast and rapidly expanding. Some key contributions that we would like to call out:

* [LangChain](https://www.langchain.com/) & [Semantic Kernel](https://learn.microsoft.com/en-us/semantic-kernel/overview/)
  * Tooling to create multi-step generative AI workflows. The OG agent pipeline that allows you to build tools that can be used by LLMs. Langchain uses “tools” to provide context to the AI. The tooling is based around function calls that can reach out and “do something” in the outside world.  

* [MemGPT](https://memgpt.ai/)
  * Provides the ability to offload memory to an external data store. This expands the context that LLMs can use when constructing an answers. There is no concept of permissions or authentication in the MemGPT specification so this must be done using another mechanism.

* [Model Context Protocol](https://modelcontextprotocol.io)
  * Published by Anthropic as a standardised protocol for data interconnect. Used with Claude to provide additional context to LLMs. MCP currently doesn’t support permissions and relies on additional implementation of the MCP server to setup authentication.

* [AWS Bedrock Agents](https://aws.amazon.com/bedrock/agents/)
  * AWS Agents framework. Depending on the knowledge base used the permissions can vary but are based on a cached version from the last sync. The agents are assumed to have full access to all resources in the knowledge base.

* [Bee Agent Framework](https://github.com/i-am-bee/bee-agent-framework)
  * AI Platform @IBM used to create Agents. The specification is similar to CrewAI and is designed to create AI agents that connect to multiple LLMs. The framework doesn’t have a concept of permissions or authentication.

* [CrewAI](https://www.crewai.com/)
  * Platform for creating AI Agents, provides support for complex access of unstructured data and LLM integrations. CrewAI doesn’t have a concept of permissions when defining or using an agent but rather employs a plugin model called “tools” for accessing external functionality.

* [ChatGPT custom GPTs](https://openai.com/index/introducing-gpts/)
  * A custom layer of context and instructions that sit on top of the GPT API that hosts ChatGPT. Simple and elegant in nature it allows anyone to make a new customised version of ChatGPT by adding new files and specific instructions.

* ChatGPT Apps
  * An extension of custom GPTs where additional context and functionality can be added by creating function calls. These function calls can use any arbitrary code exposed as an API. There is no permissions or authentication system built on top of this and needs to be implemented individually per function.

All of this tooling (and more!) is useful for implementing the spec we outline here. Our key focus is formalising and standardising on a way of describing agents, building on the foundational blocks each of the above tools provides. Luckily, we are able to stand on the shoulders of giants, and leverage these tools in the reference implementation of the OpenAgentSpec, Agent OS.
