# Module 1 - Building the agent

In this module you will deploy an Agent built with Strands on AWS Lambda and test it.

Navigate to the AWS Lambda Console and open the strands-travel-agent function
Paste the code below into your Lambda code editor
Deploy the code updates using the blue Deploy button on the left side of the editor.
The code is composed of the following:
Import the Strands Agents dependencies
The TRAVEL_AGENT_PROMPT sets up how the travel agent should behave and the tools it should use
The travel agent has a local "tool" simulating a simple flight search query. In a production scenario, this query would go to a real database system
The main part (called the handler) handles the incoming event payload for the Lambda function and returns a response. In this case, the event payload is the user's question and the response is the travel agent's response. The agent object is configured with the TRAVEL_AGENT_PROMPT and the flight_search tool
This Agent uses a Foundation model on Amazon Bedrock to generate a response for the user. The Strands SDK simplifies the experience of integrating with Amazon Bedrock. For customization options on the Amazon Bedrock integration, see the Strands documentation
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
42
43
44
45
46
47
48
49
50
from strands import Agent, tool
from typing import Dict, Any

# Define a travel-focused system prompt

TRAVEL_AGENT_PROMPT = """You are a travel assistant that can help customers book their travel.

Think step-by-step.

If a customer wants to book their travel, assist them with flight options for their destination.

Use the flight_search tool to provide flight carrier choices for their destination.

Provide the users with a friendly customer support response that includes available flights for their destination.
"""

@tool
def flight_search(city: str) -> dict:
"""Get available flight options to a city.

    Args:
        city: The name of the city
    """
    flights = {
        "Atlanta": [
            "Delta Airlines",
            "Spirit Airlines"
        ],
        "Seattle": [
            "Alaska Airlines",
            "Delta Airlines"
        ],
        "New York": [
            "United Airlines",
            "JetBlue"
        ]
    }
    return flights[city]

# The handler function signature `def handler(event, context)` is what Lambda

# looks for when invoking your function.

def handler(event: Dict[str, Any], \_context) -> str:
travel_agent = Agent(
model="us.amazon.nova-lite-v1:0",
system_prompt=TRAVEL_AGENT_PROMPT,
tools=[flight_search]
)

    response = travel_agent(event.get('prompt'))
    return str(response)

Foundation Model
Configuring your agent to use a specific Foundation Models is as easy as passing in the model id when you create the Agent. Use cases for using different Foundation Models include response quality, cost optimization, and latency. We suggest evaluating Foundation Models based on your desired behavior and cost requirements.

Testing
In order to test the agent with a basic prompt, let's use the Lambda test payload feature to pass a travel question to the agent and see its response.

Click on the Test tab above the code editor.
Event name: Test
Event JSON:
{
"prompt": "Can you tell me travel options to Seattle?"
}

Leave the rest of the fields as-is, click Save and Test.
You should see the agent's response about the flight options to Seattle show up.
The response will look similar to the following:
Hi there! I'm glad you're considering travel to Seattle. Here are some flight options available to you:

Alaska Airlines
Delta Airlines
