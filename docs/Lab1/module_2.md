# Module 2 - Coding an Agent with Tool Use

In this module, you will learn how to create an agent using code and Tool Use. The Social Media Agent transforms long-form content—blog posts, announcements, or articles—into engaging social media posts. It serves as your personal assistant for creating content that's ready to share.

Agent architecture
The agent architecture consists of these components:

AWS Lambda: Runs the agent code and hosts the individual tools
Amazon Bedrock: Provides the AI model for reasoning and decision-making
Social Media Agent

The Agent Loop
The agent loop is the core concept that enables intelligent, autonomous behavior through continuous reasoning, tool use, and response generation. This loop continues until the agent gathers all information needed for a complete answer.

Agent Loop

Agent Code Walkthrough
Tool Declaration
To achieve its objective, the agent will have the following tools available:

Tool Description
Content Summarizer Extracts key points from long text
Tone Adapter Changes writing style to match your needs (professional, casual, etc.)
Post Writer Crafts perfect content for social platforms
These tools are declared using JSON Schema as the follow:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
const TOOLS = [
{"toolSpec": {
"name": CONTENT_SUMMARIZER_TOOL_NAME,
"description": "Summarize a text extracting key takeaways",
"inputSchema": {
"json": {
"type": "object",
"properties": {
"text": {"type": "string", "description": "Text to be summarized"}
},
"required": ["text"]
}
}}
},
{"toolSpec": {
"name": TONE_ADAPTER_TOOL_NAME,
"description": "Tailor a text to a specific tone",
"inputSchema": {
"json": {
"type": "object",
"properties": {
"tone": {"type": "string", "description": "Text tone to be used(e.g., professional, casual, fantasy)"},
"text": {"type": "string", "description": "Text to be adapted"}
},
"required": ["tone","text"]
}
}}
},
{"toolSpec": {
"name": POST_WRITER_TOOL_NAME,
"description": "Create social media posts",
"inputSchema": {
"json": {
"type": "object",
"properties": {
"text": {"type": "string", "description": "Text to used as reference to create the post"}
},
"required": ["text"]
}
}}
}];

Agent Loop Implementation
The heart of our agent is the loop that implements the reasoning cycle. For each iteration the following logic is executed:

AI Reasoning: The agent thinks about the current situation and decides what to do next
Decision Point: Two things can happen:
End Turn: The agent has everything it needs and provides the final answer
Tool Use: The agent needs more information and calls a tool
Tool Execution: If tools are needed, we execute them and add results back to the conversation
Continue Loop: The cycle repeats with new information until complete
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
while(true) {
// Use Converse API to complete the prompt
const converse = await invokeConverseAPI(messages);
const assistantMessage = converse.output.message;
// Enforce one tool at a time, where 2 == thinking + toolUse
assistantMessage.content.splice(2);
// Add the assistant response in the message history
messages.push(assistantMessage);

    // Collect the model final response and return to the user
    if (converse.stopReason === 'end_turn') {
        const responseContent = assistantMessage.content[0].text;
        logModelThinking(responseContent);
        const response = responseContent.replace(THINKING_PATTERN, '').trim();
        return {response: response};
    }

    // Invoke the tool specified by the model
    if (converse.stopReason === 'tool_use') {
        const toolResponseMessage = { content: [], role: ConversationRole.USER };
        for (const content of assistantMessage.content) {
            const tool = content.toolUse;
            if (tool) {
                const toolResponse = await invokeTool(tool);
                toolResponseMessage.content.push({
                    toolResult: {
                        toolUseId: tool.toolUseId,
                        content: [{json: toolResponse}]
                    }
                });
            }
        }
        messages.push(toolResponseMessage);
    }

}

Tool Execution
When the agent decides to use a tool, this function routes the request to the appropriate tool and returns the results back to the agent.

The beauty of this approach is that the agent makes intelligent decisions about which tools to use and when. It might summarize content first, then adapt the tone, and finally create the social media post. Or it might take a different path based on your specific request.

1
2
3
4
5
6
7
8
9
10
11
12
13
async function invokeTool(tool) {
console.log(`Invoking tool: ${tool.name} (ID: ${tool.toolUseId})`);
switch (tool.name) {
case CONTENT_SUMMARIZER_TOOL_NAME:
return await invokeLambda(CONTENT_SUMMARIZER_FUNCTION, tool.input);
case TONE_ADAPTER_TOOL_NAME:
return await invokeLambda(TONE_ADAPTER_FUNCTION, tool.input);
case POST_WRITER_TOOL_NAME:
return await invokeLambda(POST_WRITER_FUNCTION, tool.input);
default:
throw new Error(`Unknown tool: ${tool.name} (ID: ${tool.toolUseId})`);
}
}

In this particular implementation, all tools are available as Lambda functions. However, they could be any resource accessible from your code, such as databases, third-party APIs, or AWS APIs.
Testing
To test the agent you will ask it to create a social media post using a peculiar tone (mythical) from the Amazon Bedrock announcement blog.

Navigate to the Lambda console
Open the function lab1-social-media-agent
In the Test tab, create a new event:
For Event Name, enter a Test or a name of your preference
For Event JSON, enter:
1
{"prompt": "Please create a social media post using a mythical tone: This April, we announced Amazon Bedrock as part of a set of new tools for building with generative AI on AWS. Amazon Bedrock is a fully managed service that offers a choice of high-performing foundation models (FMs) from leading AI companies, including AI21 Labs, Anthropic, Cohere, Stability AI, and Amazon, along with a broad set of capabilities to build generative AI applications, simplifying the development while maintaining privacy and security.\n\nToday, I’m happy to announce that Amazon Bedrock is now generally available! I’m also excited to share that Meta’s Llama 2 13B and 70B parameter models will soon be available on Amazon Bedrock.\n\nAmazon Bedrock’s comprehensive capabilities help you experiment with a variety of top FMs, customize them privately with your data using techniques such as fine-tuning and retrieval-augmented generation (RAG), and create managed agents that perform complex business tasks—all without writing any code. Check out my previous posts to learn more about agents for Amazon Bedrock and how to connect FMs to your company’s data sources.\n\nNote that some capabilities, such as agents for Amazon Bedrock, including knowledge bases, continue to be available in preview. I’ll share more details on what capabilities continue to be available in preview towards the end of this blog post.\n\nSince Amazon Bedrock is serverless, you don’t have to manage any infrastructure, and you can securely integrate and deploy generative AI capabilities into your applications using the AWS services you are already familiar with.\n\nAmazon Bedrock is integrated with Amazon CloudWatch and AWS CloudTrail to support your monitoring and governance needs. You can use CloudWatch to track usage metrics and build customized dashboards for audit purposes. With CloudTrail, you can monitor API activity and troubleshoot issues as you integrate other systems into your generative AI applications. Amazon Bedrock also allows you to build applications that are in compliance with the GDPR and you can use Amazon Bedrock to run sensitive workloads regulated under the U.S. Health Insurance Portability and Accountability Act (HIPAA).\n\nGet Started with Amazon Bedrock:\n You can access available FMs in Amazon Bedrock through the AWS Management Console, AWS SDKs, and open-source frameworks such as LangChain.\n\nIn the Amazon Bedrock console, you can browse FMs and explore and load example use cases and prompts for each model. First, you need to enable access to the models. In the console, select Model access in the left navigation pane and enable the models you would like to access. Once model access is enabled, you can try out different models and inference configuration settings to find a model that fits your use case.\n\nData Privacy and Network Security:\n With Amazon Bedrock, you are in control of your data, and all your inputs and customizations remain private to your AWS account. Your data, such as prompts, completions, and fine-tuned models, is not used for service improvement. Also, the data is never shared with third-party model providers.\n\nYour data remains in the Region where the API call is processed. All data is encrypted in transit with a minimum of TLS 1.2 encryption. Data at rest is encrypted with AES-256 using AWS KMS managed data encryption keys. You can also use your own keys (customer managed keys) to encrypt the data.\n\nYou can configure your AWS account and virtual private cloud (VPC) to use Amazon VPC endpoints (built on AWS PrivateLink) to securely connect to Amazon Bedrock over the AWS network. This allows for secure and private connectivity between your applications running in a VPC and Amazon Bedrock.\n\nGovernance and Monitoring:\n Amazon Bedrock integrates with IAM to help you manage permissions for Amazon Bedrock. Such permissions include access to specific models, playground, or features within Amazon Bedrock. All AWS-managed service API activity, including Amazon Bedrock activity, is logged to CloudTrail within your account.\n\nAmazon Bedrock emits data points to CloudWatch using the AWS/Bedrock namespace to track common metrics such as InputTokenCount, OutputTokenCount, InvocationLatency, and (number of) Invocations. You can filter results and get statistics for a specific model by specifying the model ID dimension when you search for metrics. This near real-time insight helps you track usage and cost (input and output token count) and troubleshoot performance issues (invocation latency and number of invocations) as you start building generative AI applications with Amazon Bedrock.\n\nBilling and Pricing Models:\n Here are a couple of things around billing and pricing models to keep in mind when using Amazon Bedrock:\n\nBilling – Text generation models are billed per processed input tokens and per generated output tokens. Text embedding models are billed per processed input tokens. Image generation models are billed per generated image.\n\nPricing Models – Amazon Bedrock oﬀers two pricing models, on-demand and provisioned throughput. On-demand pricing allows you to use FMs on a pay-as-you-go basis without having to make any time-based term commitments. Provisioned throughput is primarily designed for large, consistent inference workloads that need guaranteed throughput in exchange for a term commitment. Here, you specify the number of model units of a particular FM to meet your application’s performance requirements as deﬁned by the maximum number of input and output tokens processed per minute. For detailed pricing information, see Amazon Bedrock Pricing."}

Create test event
Click Test
Around 30 seconds later you should see a success message, click on Details: Test Success
Scroll down and maximize the Log Output and you will find an output similar to: Log Output
Observe how the agent first plans its steps and selects the right tools for each task. After completing all steps, it delivers the final response to the user.

Do you want to try a different tone? Just change "mythical" for another in the prompt and run the test again.
Congratulations!
You've successfully created a Social Media Agent!
