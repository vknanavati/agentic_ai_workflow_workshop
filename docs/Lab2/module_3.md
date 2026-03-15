# Module 3 - Adding agent memory
To create more natural conversations, your Strands Agent needs to remember past interactions with users. This is achieved through session management, where each conversation is tracked using a unique session ID.

Here's how it works:

Each conversation gets a unique session ID that you specify when calling the agent
Using the same session ID across multiple requests maintains continuity in the conversation
The agent can access previous context to provide more relevant responses
Store conversation history in an external data store rather than keeping it in memory. This ensures:

Data persistence across chat sessions
Scalability for multiple conversations
Better data recovery options
The stored conversation history helps the agent understand context and respond more naturally, similar to how humans remember previous parts of a conversation.

You can update the Lambda function with the following code to store and retrieve session memory from Amazon S3 using Strands built-in S3 Session Manager .

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
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
from strands import Agent, tool
from strands_tools import http_request
from strands.session.s3_session_manager import S3SessionManager
from typing import Dict, Any
import boto3
import os
import json

# Define a travel-focused system prompt
TRAVEL_AGENT_PROMPT = """You are a travel assistant that can help customers book their travel. 

Think step-by-step.

If a customer wants to book their travel, assist them with flight options for their destination and provide them with information about the weather.

Use the flight_search tool to provide flight carrier choices for their destination.

You can provide information about the weather with the following:
1. Make HTTP requests to the National Weather Service API
2. Process and display weather forecast data
3. Provide weather information for locations in the United States
4. The Seattle zip code value is 98101 and the latitude and longitude coordinates are 47.6061° N, 122.3328° W
5. First get the coordinates or grid information using https://api.weather.gov/points/{latitude},{longitude} or https://api.weather.gov/points/{zipcode}
6. Then use the returned forecast URL to get the actual forecast
When displaying responses:
- Format weather data in a human-readable way
- Highlight important information like temperature, precipitation, and alerts
- Handle errors appropriately
- Convert technical terms to user-friendly language

Always explain the weather conditions clearly and provide context for the forecast.

Provide the users with a friendly customer support response that includes available flights and the weather for their destination.

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
def handler(event: Dict[str, Any], _context) -> str:
    session_manager = S3SessionManager(
        session_id=event["user"]["session_id"],
        bucket=os.environ['SESSIONS_BUCKET'],
        prefix="agent-sessions"
    )
    travel_agent = Agent(
        model="us.amazon.nova-lite-v1:0",
        system_prompt=TRAVEL_AGENT_PROMPT,
        tools=[flight_search, http_request],
        session_manager=session_manager
    )
    response = travel_agent(event.get('prompt'))
    return str(response)

Testing your Agent's memory
In order to test the agent with a prompt, let's use the Lambda test payload feature to pass a travel question to the agent and see its response.

Test with the following event JSON:
{
    "prompt": "Can you tell me travel options to Seattle?",
    "user": {
        "session_id": "123"
    }
}

You should see the agent's response about the flight options to Seattle and weather forecast show up.
Test with the following event JSON:
{
    "prompt": "Can you tell me some local things to do?",
    "user": {
        "session_id": "123"
    }
}

You should see the agent's response include local attractions in Seattle. Note, we did not specify that Seattle was the destination. Providing the agent with memory of the session enables it to answer follow up queries that reference previous messages in the conversation.
You can find the conversation history stored in the S3 bucket  prefixed with lab2-strands-travelagents3sessionsbucket Each session will have a subdirectory under agent-sessions containing the complete conversation history.

s3-memory

Congratulations!
Congratulations, you were able to integrate your agent with a data store to track session history. This enables your agent to remember the context of previous user questions in a session. You then tested with a mock payload to test the response.

