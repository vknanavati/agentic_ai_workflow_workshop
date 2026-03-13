# Module 5 - Using MCP
Strands Agents can use the MCP Client Tool available with Strands SDK to connect to external MCP Servers and dynamically load remote tools.

For this module, you will use Bedrock AgentCore Gateway to convert an existing Lambda Function into a fully managed MCP Server. Bedrock AgentCore Gateway also supports authentication model for Inbound Authentication for secure access. In this module, you will use the OAuth2 client credentials flow using Cognito to secure the MCP server that you create from the Lambda Function.

Architecture Pattern Diagram

The strands-travel-agent Lambda function will be the MCP Client that will make a call to the Attractions MCP Server that you will create using Agent Core Gateway. This MCP Server is secured using OAuth2 using Cognito. The Attractions Lambda function has various tools like List Attractions, Reserve Ticket, and Cancel ticket to allow you to use various methods that are defined depending on the tool that you need.

Creating the Amazon Bedrock AgentCore Gateway
In this section, you'll create a Amazon Bedrock AgentCore Gateway to convert an existing Lambda Function into a MCP Server

Step 1: Create the Amazon Bedrock AgentCore Gateway
Open the Amazon Bedrock AgentCore console . Make sure that you are in the correct region
Click on Gateways in the left sidebar under Build and Deploy
Click Create gateway and enter the Gateway name as TravelAttractionsGateway
Keep the default of Quick create configurations with Cognito selected
For IAM Permissions, select the Use an existing service role that starts with lab2-strands-AgentCoreGatewayExecutionRole
Target Details:

Enter a Target name TravelAttractionsLambdaTarget

Enter Target description as Lambda function which handles Travel Attractions

Select Target type as Lambda ARN

Navigate to AWS Lambda Console , open the function attractions and copy its Function ARN. Enter the Lambda ARN in the Agent Core Gateway configuration.

Choose Target schema as Define an inline schema

Paste the below in the In-Line schema editor

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
[
  {
    "description": "List available times and price for attractions",
    "inputSchema": {
      "properties": {
        "date": {
          "description": "Requested date for attraction",
          "type": "string"
        }
      },
      "required": [
        "date"
      ],
      "type": "object"
    },
    "name": "list_attractions"
  },
  {
    "description": "Reserve attraction ticket for a date and time",
    "inputSchema": {
      "properties": {
        "attraction": {
          "description": "The attraction to reserve",
          "type": "string"
        },
        "date": {
          "description": "The available date for the attraction",
          "type": "string"
        },
        "time": {
          "description": "The available time for the attraction",
          "type": "string"
        }
      },
      "required": [
        "attraction",
        "date",
        "time"
      ],
      "type": "object"
    },
    "name": "reserve_ticket"
  },
  {
    "description": "Cancel ticket reservation",
    "inputSchema": {
      "properties": {
        "reservationCode": {
          "description": "Reservation code for reserved attraction",
          "type": "string"
        }
      },
      "required": [
        "reservationCode"
      ],
      "type": "object"
    },
    "name": "cancel_ticket"
  }
]

Keep the rest as defaults and click on Create gateway

Update the agent to use MCP tool
Navigate to AWS Lambda Console , open the function strands-travel-agent and update with the following code to allow it to use MCP Client Tool with Strands.

It adds instructions to the agent prompt about the new tools and implements the authentication required by the MCP server. Notice the handler code now lists the MCP available tools and then provide them to the agent.

Make sure to click on Deploy after update the Lambda code.

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
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
98
99
100
101
102
103
104
from strands import Agent, tool
from strands_tools import http_request, current_time
from strands.session.s3_session_manager import S3SessionManager
from strands.tools.mcp.mcp_client import MCPClient
from mcp.client.streamable_http import streamablehttp_client
from typing import Dict, Any
import os
import requests

CLIENT_ID = os.environ['CLIENT_ID']
CLIENT_SECRET = os.environ['CLIENT_SECRET']
TOKEN_URL = os.environ['DOMAIN_URL'] + '/oauth2/token'
GATEWAY_URL = os.environ['GATEWAY_URL']

# Define a travel-focused system prompt
TRAVEL_AGENT_PROMPT = """You are a travel assistant that can help customers book their travel. 

Think step-by-step.

Use the flight_search tool to provide flight carrier choices for their destination.

Use list_attractions, reserve_ticket and cancel_ticket tools to provide attractions management service.

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

def fetch_access_token(client_id, client_secret, token_url):
  response = requests.post(
    token_url,
    data="grant_type=client_credentials&client_id={client_id}&client_secret={client_secret}".format(client_id=client_id, client_secret=client_secret),
    headers={'Content-Type': 'application/x-www-form-urlencoded'}
  )
  return response.json()['access_token']

def create_streamable_http_transport(mcp_url: str, access_token: str):
       return streamablehttp_client(mcp_url, headers={"Authorization": f"Bearer {access_token}"})


# The handler function signature `def handler(event, context)` is what Lambda
# looks for when invoking your function.
def handler(event: Dict[str, Any], _context) -> str:
    session_manager = S3SessionManager(
        session_id=event["user"]["session_id"],
        bucket=os.environ['SESSIONS_BUCKET'],
        prefix="agent-sessions"
    )

    access_token = fetch_access_token(CLIENT_ID, CLIENT_SECRET, TOKEN_URL)
    mcp_client = MCPClient(lambda: create_streamable_http_transport(GATEWAY_URL, access_token))
       
    with mcp_client:
        tools_mcp = mcp_client.list_tools_sync()
        #print(f"Found the following tools: {[tool.tool_name for tool in tools_mcp]}")

        tools_mcp += [flight_search, http_request, current_time]

        travel_agent = Agent(
            model="us.amazon.nova-lite-v1:0",
            system_prompt=TRAVEL_AGENT_PROMPT,
            tools= tools_mcp,
            session_manager=session_manager
        )
        response = travel_agent(event.get('prompt')) 

    return str(response)

Click on CloudShell on the bottom left of the console or click here . Now, run the following commands to retrieve the environment variable values for the next step:

aws s3 cp s3://ws-assets-prod-iad-r-pdx-f3b3f9f1a7d6a3d0/eb18d538-bf1f-49b9-9747-c474953deee1/lab2/agentcore_gateway_info.sh . && chmod +x agentcore_gateway_info.sh
./agentcore_gateway_info.sh

Click on Configuration, Environment variables and replace the environment variables values of CLIENT_ID, CLIENT_SECRET, DOMAIN_URL and GATEWAY_URL

Click on Save

Testing your Agent
In order to test the agent with a prompt, let's use the Lambda test payload feature to Reserve a Ticket for Space Needle for a time and date.

Test with the following event JSON:

{
    "prompt": "Reserve a ticket for Space needle for 9:00 AM tomorrow",
    "user": {
       "session_id": "123"
    }
}

You should see the agent's response with the Reservation Success similar to below. The response might differ but it should the confirm the reservation:

Hi there! Your ticket for the Space Needle at 9:00 AM tomorrow has been reserved. Here is your reservation code: W8VPWZ. Enjoy your visit!

Congratulations!
Congratulations, you were able to create a MCP Server from a Lambda function using Amazon Bedrock AgentCore Gateway and integrated your agent with it.

