# Lab 2 - Building agents with Strands Agents SDK

- Purpose:
  - learn common patterns for agents on AWS using open source agent frameworks and managed services.
  - build Agents with Strands Agents.

**Strands Agents**
is a simple-to-use, code-first framework for building agents.
It is lightweight and production-ready, supporting many model providers and deployment targets.

As seen in the diagram below, an agent interacts with a Foundation Model and tools in a loop until it completes the task provided by the prompt.

**Strands Agentic Loop**
To define an agent with the Strands Agents SDK, you define these three components in code:

- Model: Strands offers flexible model support. You can use any model in Amazon Bedrock that supports tool use and streaming, a model from Anthropic’s Claude model family through the Anthropic API, a model from the Llama model family via Llama API, Ollama for local development, and many other model providers such as OpenAI through LiteLLM. You can additionally define your own custom model provider with Strands.
- Tools: You can choose from thousands of published Model Context Protocol (MCP) servers to use as tools for your agent. Strands also provides 20+ pre-built example tools, including tools for manipulating files, making API requests, and interacting with AWS APIs. You can easily use any Python function as a tool, by simply using the Strands @tool decorator.
- Prompt: You provide a natural language prompt that defines the task for your agent, such as answering a question from an end user. You can also provide a system prompt that provides general instructions and desired behavior for the agent.

**What you'll build**
In this Lab you are going to build a travel agent that will suggest flight options and provide weather forecasts for your trip. You're going to build an agent step-by-step with the following features:

Defining a system prompt to shape the agent's behavior
Adding a local tool to your agent, allowing it to retrieve flights from a mock database
Deploying your Agent on AWS Lambda
That's enough to get started, but after that, you'll add more features and learn Strands concepts such as:

Integrate agent with external API to provide real-time data
Adding session memory so your agent is aware of all the past messages in your conversation
Using Amazon Bedrock KnowledgeBases to perform Retrieval Augmented Generation (RAG) on your data sources
Create a MCP Server from Lambda functions and consume them
Expose the agent using a REST API through API Gateway
