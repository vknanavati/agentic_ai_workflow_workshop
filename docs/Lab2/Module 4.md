# Module 4 - Using Bedrock Knowledge Bases for RAG

Strands Agent can use Amazon Bedrock Knowledge Bases to give agents contextual information from your company's private data sources. This is achieved using the retrieve tool that is predefined in Strands SDK. It uses the knowledge base Id to retrieve the data depending on your prompt.

Preparing the Amazon Bedrock Knowledge Base
In this section, you'll create a Knowledge Base on Amazon Bedrock using Amazon S3 Vectors as the vector store. This will enable your agent to perform Retrieval Augmented Generation (RAG) with your private data sources.

Step 1: Upload Sample Data to S3
Open AWS Cloud Shell and close the welcome window.

Copy the sample document to the Cloud Shell environment and print its content to see a list of tour operators:

BUCKET=$(aws s3api list-buckets --query "Buckets[?starts_with(Name, 'lab2-strands-travelagents3bucketrag')].Name" --output text)
aws s3 cp s3://ws-assets-prod-iad-r-pdx-f3b3f9f1a7d6a3d0/eb18d538-bf1f-49b9-9747-c474953deee1/seattletouroperators.txt .
cat seattletouroperators.txt

Copy the sample document into the S3 bucket that will be used as a data source by the knowledge base:

aws s3 cp seattletouroperators.txt s3://$BUCKET/

Step 2: Create the knowledge Base
Open the Amazon Bedrock console
Navigate to Knowledge bases in the left sidebar
Click Create and select Knowledge Base with vector store
Knowledge base details:

Enter a knowledge base name StrandsTravelAgentKB
Select the existing IAM role that starts with lab2-strands-BedrockKnowledgeBaseRole
Select S3 as the data source type
Click Next
Data source configuration:

Enter a data source name TravelAgentS3DataSource
Click Browse S3 and select the bucket with prefix lab2-strands-travelagents3bucketrag
Click Next
Data storage and processing:

Click Select model and Select Amazon - Titan Text Embeddings v2 as the embedding model
Keep the default of Quick create a new vector store selected
Select Amazon S3 Vectors as the vector store
Click Next
Review and create:

Review your configuration
Click Create knowledge base
Creating the knowledge base will take a few seconds

Step 3: Sync the data source
After the knowledge base is created, you'll be redirected to the knowledge base details page
In the Data sources section, find your data source named TravelAgentS3DataSource
Click Sync to start the ingestion process
Wait for the sync to complete (status will change to "Available")
Bedrock Sync

Sync the data source will:
Parse your documents from the S3 data bucket
Split them into chunks using the default chunking strategy
Generate embeddings using Amazon Titan Text Embeddings v2
Store the embeddings in Amazon S3 Vectors for efficient retrieval
Step 4: Collect the knowledge base ID
At the top of the page, you'll see the Knowledge base overview section
Copy the Knowledge base ID (it will look like ABCDEFGHIJ)
KB ID

Update the agent to use retrieve tool
Now you'll update the Lambda function to use the retrieve tool with your Knowledge Base. This involves both configuring the environment variable and updating the code.

Update the environment variable
First, configure the Lambda function to reference your Knowledge Base:

Open the AWS Lambda console
Find and click on the strands-travel-agent function
Select the Configuration tab and Environment variables in the left sidebar
Edit the environment variable KNOWLEDGE_BASE_ID and paste the Knowledge Base ID you copied in Step 4
Click Save to apply the changes
This environment variable will allow your Lambda function to reference the correct Knowledge Base when using the retrieve tool.

Update the Lambda code
Navigate to AWS Lambda Console , open the function strands-travel-agent and update with the following code to allow it to use Retrieval Augmented Generation (RAG)

It adds instructions to the agent prompt about the new retrieve tool and provide it to the agent along with the other tools.

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
from strands import Agent, tool
from strands_tools import http_request, retrieve
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

Use the retrieve tool to get validated list of tour operators for different kinds of tours and activities that we support.

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
session_manager = S3SessionManager(
session_id=event["user"]["session_id"],
bucket=os.environ['SESSIONS_BUCKET'],
prefix="agent-sessions"
)
travel_agent = Agent(
model="us.amazon.nova-lite-v1:0",
system_prompt=TRAVEL_AGENT_PROMPT,
tools=[flight_search, http_request, retrieve],
session_manager=session_manager
)
response = travel_agent(event.get('prompt'))
return str(response)

Testing your Agent
In order to test the agent with a prompt, let's use the Lambda test payload feature to pass a travel question to the agent and see its response.

Test with the following event JSON:
{
"prompt": "What are some tour operators with water activities?",
"user": {
"session_id": "123"
}
}

You should see the agent's response with the "approved tour operators" that are available in the knowledge base. In the Log Output you will also see that it uses the retrieve tool to get this information.
