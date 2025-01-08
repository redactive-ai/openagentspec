# OpenAgentSpec Specification

The core component of the OpenAgentSpec is the specification of an agent. An agent is defined by its OpenAgentSpec, and a server that implements the OpenAgentSpec is responsible for parsing each OpenAgentSpec it serves, and ensuring that the following specification is met.

## Names & References

* A tool is uniquely identified within a server by its `tool_name`. 
* An agent is uniquely identified within a server by its `agent_name`.
* Both `tool_name`s and `agent_name`s MUST fit the definition of a DNS subdomain as defined in [RFC 1123](https://tools.ietf.org/html/rfc1123). This means each name MUST:
  * contain no more than 253 characters
  * contain only lowercase alphanumeric characters, '-' or '.'
  * start with an alphanumeric character
  * end with an alphanumeric character.
* A tool is only ever referred to by its complete `tool_name`.
* A reference to an agent may be one of 3 forms:
  * The server MUST parse references to agents that are solely of the form:
    * `agent_name`
  * The server MUST parse references to agents that are of the form:
    * `oagent://{agent_name}`
  * The server MAY interpret references of the following form as remote agent references:
    * `oagent://{hostname}[/{path}]/{agent_name}`
* Where the server contains a tool and an agent of the same name, and where a reference may refer to either a tool or an agent, the server MUST interpret the reference as referring to the tool. To specify an agent exclusively, use the form: 
  * `oagent://{agent_name}`
* A server MAY support remote agent references by interpreting these references as fully qualified urls, replacing `oagent://` with `https://` and issuing an HTTPS GET request to fetch the OpenAgentSpec remotely.

## Agent

An Agent is defined by a yaml file with the following schema:

```yaml
kind: "openagentspec:v1/agent"
name: str
description: str
intent: str
owner: str	

capabilities: dict
	keys: str
    values: object
        collect_results: bool
        user_identity: bool
        static_identity: str | None
        input_restriction: object | None
            require_review: str | None
            assertion: OAGENT_ASSERTION | None
        output_restriction: object | None
            require_review: str | None
            assertion: OAGENT_ASSERTION | None

guardrails: object | None
	input: object | None
        tool_name: str
        assertion: OAGENT_ASSERTION
    output: object | None
	    tool_name: str
        assertion: OAGENT_ASSERTION

exposes: object | None
	keys: str
    values: OAGENT_SELECTION

lifespan: object | None
	retain_memory: bool
	short_circuit: int
```

1. The `kind` field MUST contain the value `openagentspec:v1/agent` for files conforming to this version of the spec.
1. The `name` field MUST contain a string that uniquely identifies this agent within the server.
1. The `description` field MUST contain a description of the agent, written in the third person.
1. The `intent` field MUST contain a description of the goal of the agent, written in the second person. The server MAY pass this to a generative model to contextualise the user request. The server MAY pass additional information beyond this field.
1. The `owner` field MUST contain a unique identifier corresponding to the owner of the agent.

1. The keys of the `capabilities` field MUST be a reference to a tool or an agent. The server MUST be able to describe the tool and its parameters to a generative model, and the server MUST be able to invoke the tool and parse its response. The values of the capabilities field will be referred to as a **capability**.
1. If the `collect_results` field of a capability is absent or `True`, the server MUST append the output from the tool to the Engagement object.
1. If the `user_identity` field of a capability is absent or `True`, the server MUST ensure that the identity of the calling user is used to access the tool. The server MAY translate the identity into credentials specific to the combination of tool and user.
1. The `static_identity` field of a capability MAY contain an identifier the server understands as uniquely identifying a set of credentials to use with the tool. If this value is populated, and either `user_identity` is `False` or the agent has been invoked without an identity, the server MUST use these credentials to access the tool.
1. A capability MAY contain an `input_restriction` section. The objects of this section will be referred to as a **restriction**. An `input_restriction` defines the conditions that are validated before the capability is invoked, and can validate the agent’s input to a given capability. 
1. A capability MAY contain an `output_restriction` section. The object of this section is a restriction. An `output_restriction` defines the conditions that are validated after the capability is invoked, but prior to giving the result to the generative model, and can validate the given capability’s output.
1. The `require_review` section of a restriction MAY be populated with an identifier the server understands, either a user identity or an agent identifier. If populated, the server MUST ask the specified user or agent if the executing agent agent is able to continue, given the current Engagement object. The server MUST receive confirmation from the specified user or agent before continuing the agent flow.
1. The `assertion` field of a restriction MAY contain one assertion in CEL (Common Expression Language). If populated, the server MUST evaluate the expression using the current Engagement object as the first and only input.
   1. If belonging to the `input_restriction`, the server MUST evaluate the assertion prior to invoking a tool.
   1. If belonging to the `output_restriction`, the server MUST evaluate the assertion after invoking the tool, prior to using the result in any other way.  
   1. The server MUST NOT allow the agent to access the capability or its response if any assertion fails.

1. The `guardrails` section MAY contain one or both of an input object and an output object. These objects will be referred to as a **guardrail**.
1. If defined, a guardrail MUST contain a `tool_name` field and an `assertion` field.
   1. The `tool_name` MUST reference a particular tool to the server.
   1. The `assertion` MUST contain an assertion in CEL.
   1. For the input guardrail, the server MUST invoke the corresponding tool prior to passing any user input to a generative model. For Engagements that recieve multiple user inputs, the server MUST invoke the tool each time.
   1. For the output guardrail, the server MUST invoke the corresponding tool prior to returning any generative model response to the calling user. For Engagements that yield back to the user multiple times, the server MUST invoke the tool each time.
   1. For every guardrail tool invocation, the server MUST evaluate the given assertion using the tool output as the first and only input. 
   1. If any assertion fails, the server MUST fail the execution without executing any more tools, and return an error to the calling user.

1. The `exposes` section MAY be populated. If populated, the server MUST respond to the calling user with the key-value pairs detailed in the exposes section. The values MUST refer to values from the Engagement object in CEL.

1. The `lifespan` section MAY be populated with an object
   1. If the `retain_memory` field is set to `False`, the server MUST NOT retain past context for the generative model across multiple Engagements. If absent or `True`, it MAY do so. The mechanism for doing so is left open to the implementation.
   1. The `short_circuit` field MAY contain an integer corresponding to the maximum length of a repeated subsequence of tool calls from the agent. The server MUST terminate the execution of the agent if it attempts to violate this condition. For example, if set to 3, the following would be terminated:
      * Tool 1 ✅ -> Tool 1 ✅ -> Tool 1 ❌
      * Tool 1 ✅ -> Tool 2 ✅ -> Tool 1 ✅ -> Tool 2 ✅ -> Tool 1 ✅ -> Tool 2 ❌

## Engagement

1. The server MUST use this object and only this object to evaluate any `input_restriction` and `output_restriction` `assertion`s that are defined as it orchestrates each agent.
1. The server MUST make any parts of this object that have been specified in the `exposes` section  available alongside the generated text results of the agent.
1. The server MUST store the following fields in an Engagement object:
   * `started_at`: Timestamp of when the Engagement was created.
   * `user.id`: An implementation specific id that uniquely identifies the Engagement user to the server.
   * `user.email`: The email address of the Engagement user, if known.
   * `recent._name`: The name of the last tool or agent the agent invoked.
   * `recent.{tool_name | agent_name}.inputs`: The inputs for the most recent invocation of the referenced tool or agent.
   * `recent.{tool_name | agent_name}.outputs`: The outputs of the most recent invocation of the referenced tool or agent.
   * `history._list`: Ordered list containing the name of each tool or agent the agent has invoked for this Engagement in order of invocation.
   * `history._total`: Integer count of tool and agents invoked.
   * `history.{tool_name | agent_name}.[x].inputs` The inputs for the `x`th invocation of the referenced tool or agent. 
   * `history.{tool_name | agent_name}.[x].outputs` The outputs of the `x`th invocation of the referenced tool or agent. 



## Server

1. The server MUST support a method of surfacing the Open Agent Spec definition for each and every Agent it hosts.
1. The server MAY accept well-formed agent documents conforming to the above specification as a method of allowing users to define Agents.
1. The server MAY choose to make it's agents available for remote servers and agents to consume. If so:
   1. The server MUST expose each of it's Open Agent Spec definitions as a distinct HTTPS GET endpoint. These endpoints MUST return the agent definition in yaml format.
   1. The HTTPS GET endpoints that return Open Agent Spec specifications MAY be advertised for others to consume by replacing `https://` with `oagent://` in the protocol of the url.
   1. The server MUST accept HTTPS POSTs to each of these endpoints, and:
      1. These POST endpoints MUST accept the input to begin an Engagement with an agent in json format.
      1. A POST request to one of these endpoints will create a new engagement with the agent defined in the equivalent GET request.
      1. The server MUST respond to the POST requests with a 303 and a redirection to a location that can be queried to fetch the state of the Engagement in json format.