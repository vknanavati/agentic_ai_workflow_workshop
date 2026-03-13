# Summary

In this workshop, you explored three approaches to implementing agentic AI workflows. While all methods facilitate agentic AI development, they diverge in key aspects:

Aspect AWS Step Functions with Tool Use Strands Agents SDK Amazon Bedrock Agents
Architecture Custom orchestration using Step Functions to coordinate between foundation models and tools Code-first framework with flexible agent loop implementation on serverless infrastructure Managed service with built-in orchestration between models and action groups
Development Model More granular control with explicit workflow definition Code-first approach with decorators and Python functions Higher-level abstraction with declarative agent definition
Flexibility Highly customizable with ability to integrate any AWS service or external API Highly flexible with support for multiple model providers, MCP servers, native and custom tools Low flexibility with predefined patterns for tools and knowledge bases
Complexity Moderate configuration and explicit state management Low to moderate complexity with code-based configuration and deployment Simplified setup with managed agent lifecycle and built-in capabilities
Monitoring Custom monitoring through CloudWatch and Step Functions execution history Framework based with Open Telemetry support Built-in tracing through Amazon Bedrock console
Skills acquired
Through this workshop, you've developed the following skills:

Designing and implementing AI agents for real-world applications
Orchestrating agentic workflows using workflow engines and state machines
Building code-first agents with Strands SDK
Integrating agents with external APIs and dynamic tool discovery through Model Context Protocol
Implementing persistent memory and conversation state management
Applying Retrieval Augmented Generation (RAG) patterns for private data access
Connecting foundation models with external tools and services
Exposing agents through secure API endpoints with authentication
Building multi-agent systems that collaborate to solve complex problems
Using Infrastructure as Code to deploy and manage AI agents
