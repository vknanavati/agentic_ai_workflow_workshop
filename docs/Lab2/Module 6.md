# Module 6 - Exposing the agent via API Gateway

In this module, you will expose the Strands agent through Amazon API Gateway, making it accessible via HTTP requests. The API will be secured using Amazon Cognito for authentication and proxy the requests to the Lambda.

Architecture Diagram

Step 1: Create the API Gateway REST API
Navigate to the Amazon API Gateway Console
Click Create API
Under REST API, click Build
Choose New API
Enter the following details:
API name: strands-travel-agent
Description: API for Strands Travel Agent
Endpoint Type: Regional
Click Create API
Step 2: Create the Chat Resource
Click Resources in the left sidebar
Select the root resource (/) and Click on Create Resource
Enter the following details:
Resource Name: chat
Resource Path: /
Leave CORS (Cross Origin Resource Sharing) unchecked for now
Click Create Resource
API Gateway Chat Resource

Step 3: Create the POST Method
Select the /chat resource you just created
Click Create Method
Select POST from the dropdown for Method type
Configure the method with these settings:
Integration type: Lambda Function
Lambda Proxy integration: Enabled
Lambda Function: strands-travel-agent
Keep the rest as defaults and Click Create Method
If prompted about giving API Gateway permission to invoke your Lambda function, click OK
API Gateway Post Method

Step 4: Create the Cognito Authorizer
Click Authorizers in the left sidebar
Click Create authorizer
Enter the following details:
Name: CognitoAuthorizer
Type: Cognito
Cognito User Pool: Select the existing user pool named StrandsAgentUserPool
Token Source: Authorization
Click Create Authorizer
API Gateway Authorizer

Step 5: Add Authorization to the Method
Click Resources in the left sidebar
With the POST method selected under /chat, click Method Request
Click Edit on Method request settings
Select CognitoAuthorizer from the Authorization dropdown
Click Save
Step 6: Deploy the API
Click Deploy API
Create a new deployment stage:
Deployment stage: [New Stage]
Stage name: prod
Click Deploy
Step 7: Update Lambda Function for API Gateway
Now update the Lambda function to handle API Gateway requests.

Navigate to AWS Lambda Console , open the function strands-travel-agent and update with the following code to allow it to handle API Gateway requests.

It parses the prompt from body and cleans the final response to avoid any reasoning tags in the output. Additionally, the authenticated username is now used as the session_id providing a proper user session isolation.

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
105
106
107
108
109
110
111
112
113
from strands import Agent, tool
from strands_tools import http_request, current_time
from strands.session.s3_session_manager import S3SessionManager
from strands.tools.mcp.mcp_client import MCPClient
from mcp.client.streamable_http import streamablehttp_client
from typing import Dict, Any
import os
import requests
import json
import re

THINKING_PATTERN = r'<thinking>.\*?</thinking>'
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

def handler(event: Dict[str, Any], \_context) -> str:
session_manager = S3SessionManager(
session_id=event["requestContext"]["authorizer"]["claims"]["cognito:username"],
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
        body = json.loads(event['body'])
        response = travel_agent(body['prompt'])

    return {
        'statusCode': 200,
        'body': json.dumps({
            'response': re.sub(THINKING_PATTERN, '', str(response), flags=re.DOTALL).strip()
        })
    }

Step 7: Test Your API
Before test the API you will create an user and define its credentials on Cognito. Then, you will authenticate to obtain a token that will be provided in the Authorization header to API Gateway.

Click CloudShell at the bottom of the AWS Console or open CloudShell

Run the following commands to get your User Pool details and create a test user:

# Get User Pool ID and Client ID

USER_POOL_ID=$(aws cognito-idp list-user-pools --max-results 20 --query 'UserPools[?Name==`StrandsAgentUserPool`].Id' --output text)
CLIENT_ID=$(aws cognito-idp list-user-pool-clients --user-pool-id $USER_POOL_ID --query 'UserPoolClients[0].ClientId' --output text)

Create a test user and set up credentials:

# Create test user

aws cognito-idp admin-create-user \
 --user-pool-id $USER_POOL_ID \
 --username testuser \
 --user-attributes Name=email,Value=test@example.com Name=email_verified,Value=true \
 --temporary-password "TempPass123!" \
 --message-action SUPPRESS > /dev/null

# Set permanent password for the user

aws cognito-idp admin-set-user-password \
 --user-pool-id $USER_POOL_ID \
 --username testuser \
 --password "MyPassword123!" \
 --permanent > /dev/null

# Get access token

TOKEN_RESPONSE=$(aws cognito-idp admin-initiate-auth \
 --user-pool-id $USER_POOL_ID \
 --client-id $CLIENT_ID \
 --auth-flow ADMIN_NO_SRP_AUTH \
 --auth-parameters USERNAME=testuser,PASSWORD="MyPassword123!")

ID_TOKEN=$(echo $TOKEN_RESPONSE | jq -r '.AuthenticationResult.IdToken')
echo "ID Token: $ID_TOKEN"

Test the API Endpoint
Get your API Gateway URL:

Click Stages in the left sidebar
Expand the prod stage
Expand the / (root) resource
Expand the /chat resource
Click on POST method
Copy the Invoke URL shown Invoke URL
Back in CloudShell, export the API URL with the Invoke URL you just copied:
API_URL=REPLACE_BY_INVOKE_URL

Test the API using CloudShell:

curl -X POST $API_URL \
 -H "Authorization: $ID_TOKEN" \
 -H "Content-Type: application/json" \
 -d '{"prompt": "Can you tell me travel options to Seattle?"}' | jq

You should receive a response similar to:

{ "response": "Hi there! I found that you can fly to Seattle with Alaska Airlines or Delta Airlines. Would you also like to know about attractions in Seattle? Also, do you need the weather forecast for your trip?" }

The API Gateway provides additional features like throttling, caching, and request/response transformations that you can configure based on your specific requirements.

Congratulations!
You have successfully exposed your Strands agent through API Gateway with Cognito authentication. Your agent is now accessible via HTTP requests and can be integrated into web applications, mobile apps, or other services.
