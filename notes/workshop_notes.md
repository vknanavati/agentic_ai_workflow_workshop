Converse API-Amazon Bedrock API used to interact with AI models. Here's what the document reveals about it:
It's invoked with a structured request that includes:

ModelId — specifies which model to use (e.g., us.amazon.nova-lite-v1:0)
Messages — a list of conversation turns, each with a Role (e.g., "user") and Content (the text of the message)

So in essence, it's a conversational inference API that lets you send a message to a foundation model hosted on Amazon Bedrock and receive a response — similar in concept to a chat completion API.
The document demonstrates this by sending the question "What is Amazon Bedrock Converse API?" to the Nova Lite model via a Step Functions workflow, then inspecting the model's response in the State Output.

Lab 1.3:

- Create a simple Step Functions workflow from scratch
- Make a single Bedrock API call (Converse) with a hardcoded question
- Observe the raw output

Lab 1.4:

- is about full agent orchestration — it's significantly more complex:
- The workflow has multiple states and decision logic (e.g., "Use tool?" branching)
- It involves three specialized Lambda functions (Content Summarizer, Tone Adapter, Post Writer)
- It uses a feedback loop — after a tool runs, results are added back to conversation history and the model is invoked again
- It processes real-world, long-form content (a full blog post) and transforms it into a social media post

In short, Module 3 teaches the mechanics of Step Functions + Bedrock, while Module 4 shows a production-like agentic pattern where the model decides which tools to use and iterates until it reaches a final answer.
