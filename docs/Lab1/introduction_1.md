
Building and Scaling Agentic AI Workflows
Lab 1 - Building agentic workflows with AWS Step Functions
Lab 1 - Building agentic workflows with AWS Step Functions
Estimated Duration: 30 minutes
Introduction
In this lab, you will explore agentic workflows and how they represent a paradigm shift in AI application development. Agentic systems combine the reasoning capabilities of Large Language Models (LLMs) with the ability to interact with external tools and services, creating autonomous systems that can plan, decide, and act. By leveraging AWS Lambda, AWS Step Functions, and Amazon Bedrock, you'll learn how to design orchestration patterns that enable AI to make decisions and execute complex workflows across multiple services.

What you will build
You'll create a Social Media Agent designed to repurpose long-form content across multiple social media channels. This agent will:

Process technical blog posts and whitepapers that contain complex product announcements, technical specifications, or industry analyses
Dynamically determine the optimal transformation strategy based on content analysis using Amazon Bedrock's tool use capabilities
Orchestrate a specialized toolchain tailored for marketing content transformation:
Content Summarizer: Extracts key technical points and business value propositions from dense technical documentation
Tone Adapter: Transforms technical language into platform-appropriate messaging (LinkedIn professional tone, Twitter conversational style, etc.)
Post Writer: Creates platform-optimized content with appropriate character limits, hashtag strategies, and call-to-action elements
The agent will follow an intelligent loop pattern to reason about the content, decide which tools to use, and create engaging social media posts tailored to your specifications.

Agent Flow

Learning Objectives
After completing this module, you will be able to:

Implement tool use (function calling) with Amazon Bedrock models
Build orchestrated workflows using AWS Step Functions to coordinate AI and external systems
Design conversational AI solutions that can take actions on behalf of users
Create serverless applications that combine multiple AWS services for practical use cases
Understand the differences between code-based and workflow-based agent orchestration
Implement the agent loop pattern for continuous reasoning and tool execution
Previous
Next
© 2008 - 2026, Amazon Web Services, Inc. or its affiliates. All rights reserved.
Privacy policy
Terms of use
Cookie preferences