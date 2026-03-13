# Module 1 - Understanding Tool Use
Tool Use  is an Amazon Bedrock feature that allows the agent model to use tools that can help respond to a message. When invoking Converse  or InvokeModel  APIs, you can include a list of available tools. If the model decides to use a tool, it pauses the response, returns the tool name and input data, and waits. Your agent application runs the tool, sends the result back to the model, and the model continues the conversation using that result.

Tool Use - Sequence Diagram

Example application
For example, you might have a chat application that lets users find out the most popular song played on a radio station. To answer a request for the most popular song, a model needs a tool that can query and return the song information.

API Call
The following is a code snippet showing how to use Tool Use with the AWS JavaScript SDK :

1
2
3
4
5
6
await bedrockClient.send(new ConverseCommand({
    messages: messages,
    modelId: MODEL_ID,
    system: SYSTEM_PROMPT,
    toolConfig: TOOLS
}));

The toolConfig parameter is what distinguishes this API invocation to leverage Tool Use. It contains the list of available tools and expected input properties in JSON Schema  definition:

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
{
    "tools": [
        {
            "toolSpec": {
                "name": "top_song",
                "description": "Get the most popular song played on a radio station.",
                "inputSchema": {
                    "json": {
                        "type": "object",
                        "properties": {
                            "sign": {
                                "type": "string",
                                "description": "The call sign for the radio station for which you want the most popular song. Example calls signs are WZPZ and WKRP."
                            }
                        },
                        "required": [
                            "sign"
                        ]
                    }
                }
            }
        }
    ]
}

Understanding the response
Let's say the user asks What is the most popular song on the WZPZ? and the model chooses to use a tool, it would produce the following response:

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
{
    "output": {
        "message": {
            "role": "assistant",
            "content": [
                {
                    "toolUse": {
                        "toolUseId": "tooluse_kZJMlvQmRJ6eAyJE5GIl7Q",
                        "name": "top_song",
                        "input": {
                            "sign": "WZPZ"
                        }
                    }
                }
            ]
        }
    },
    "stopReason": "tool_use"
}

Notice the name property matches the value top_song we set in toolConfig. The application uses this information to invoke the appropriate tool and send tool response to the model. When the model returns stopReason as end_turn, the conversation ends:

1
"stopReason": "end_turn"

