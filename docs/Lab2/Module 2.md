# Module 2 - Integration with external APIs

In this module, you'll learn how to update your agent to add new functionality. We'll update the agent in two ways:

Provide weather forecasts for the destination
Use a different Foundation Model in the Agent
Integrating your Strands Agent with external data sources via HTTP requests helps unlock a wide variety of use cases for your customers. You can use a similar approach to build agents for internal databases, APIs, or SaaS applications.

Updating the agent
We want to provide the users with weather information for their travel destination. We can retrieve the weather at the destinations with a call from our agent to the National Weather Service API
We have to update the TRAVEL_AGENT_PROMPT to describe the new behavior we want from our agent, in this case, including weather forecasts as part of the response
We add the built-in tool, http_request from the strands-tools library in the agent configuration
Deploy the code updates using the blue Deploy button on the left side of the editor.
We can test the new functionality with the same method as before
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
from strands import Agent, tool
from strands_tools import http_request
from typing import Dict, Any

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

def handler(event: Dict[str, Any], \_context) -> str:
travel_agent = Agent(
model="us.amazon.nova-lite-v1:0",
system_prompt=TRAVEL_AGENT_PROMPT,
tools=[flight_search, http_request]
)

    response = travel_agent(event.get('prompt'))
    return str(response)

Testing
In order to test the agent with a prompt, let's use the Lambda test payload feature to pass a travel question to the agent and see its response.

Click on the Test tab above the code editor.
Event name: Test
Event JSON:
{
"prompt": "Can you tell me travel options to Seattle?"
}

Leave the rest of the fields as-is, click Save and Test
You should see the agent's response about the flight options to Seattle and a weather forecast in the response similar to the following:
Hi there! I'm happy to assist you with your travel options to Seattle.

Available Flights:

Alaska Airlines
Delta Airlines Weather Forecast for Seattle:
This Afternoon (Oct 15): Mostly sunny with a high near 61°F. Northwest wind around 2 mph.
Tonight (Oct 15): Mostly clear with a low around 43°F. East southeast wind around 3 mph.
Thursday (Oct 16): Partly sunny then chance of light rain with a high near 58°F. South wind around 6 mph.
Thursday Night (Oct 16): Light rain likely with a low around 51°F. South wind around 7 mph.
Friday (Oct 17): Chance of light rain with a high near 60°F. West northwest wind around 5 mph. [..]
