# Module 2 - Building a Multi-Agent Solution

Introduction
In this module, you will build a comprehensive multi-agent solution for travel planning using Amazon Bedrock Agents. Multi-agent systems represent a powerful approach where specialized AI agents collaborate to solve complex problems that would be difficult for a single agent to handle effectively.

What You Will Build
You'll work with three specialized agents that collaborate to create a comprehensive travel planning solution:

Flight Agent - You'll leverage the existing Flight Agent that you built in the previous module, which handles flight searches, bookings, and cancellations
Weather Agent - You'll build a new Weather Agent that provides weather forecasts and historical data for travel destinations
Travel Agent - You'll create a supervisor agent that coordinates between the specialized agents and provides comprehensive travel recommendations
Multi-Agent Architecture

Benefits of the Multi-Agent Approach
Specialization: Each agent focuses on a specific domain, allowing for deeper expertise
Modularity: Agents can be developed, tested, and updated independently
Scalability: New capabilities can be added by creating additional specialized agents
Improved User Experience: Users interact with a single interface while benefiting from multiple specialized systems
Let's start by building the Weather Agent!

# Building a supervisor agent

In this module, you'll create a supervisor agent that orchestrates interactions between the flight and weather agents for comprehensive travel planning.

Agent architecture
Multi-agent

Prerequisite
Before starting this module, ensure you have completed:

Module 1 - Building your first agent
Module 2 - Building agent with IaC
Create the agent
Navigate to the Amazon Bedrock console
In the left navigation pane, click Agents
Click Create agent
For Name, enter travel-agent
For Description, enter Provide comprehensive travel planning through multi-agent collaboration
Check Enable multi-agent collaboration option
Click Create
Customize the agent
Under the Agent details section:
For Agent resource, select Use an existing service role and enter the role prefixed with lab3-bedrock-BedrockAgentRole
In the Select model dropdown, select Anthropic - Claude 3.5 Haiku and click Apply
For Instructions for the Agent, enter:
You are a travel agent organizing seamless trip arrangements by managing multiple specialized agents. Your task is to oversee the travel planning process by coordinating flight bookings and weather insights to ensure a smooth travel experience. When a user requests travel assistance, retrieve flight options based on their preferences, verify weather conditions for the destination, and provide recommendations accordingly. If extreme weather conditions may impact travel, suggest alternative dates or destinations. Ensure that all flight bookings and cancellations are handled efficiently while keeping the traveler informed of any potential disruptions. Your goal is to provide a well-coordinated and stress-free travel experience through effective agent collaboration.

Click Save on the top
Configure multi-agent collaboration
In the Multi-agent collaboration section, click Edit
Enable the toggle switch Multi-agent collaboration
In the Agent collaborator panel add the agents created:
Flight Agent
In the Collaborator agent dropdown, select flight-agent
In the Agent alias dropdown, select dev
For Collaborator name, enter flight-agent
For Instructions for the Agent, enter Use flight-agent for flight management activities like search, book or cancel a flight
Weather Agent
In the Collaborator agent dropdown, select weather-agent
In the Agent alias dropdown, select dev
For Collaborator name, enter weather-agent
For Instructions for the Agent, enter Use weather-agent for weather data retrieval such as forecasts and historical weather data
Saving the changes
Now let's save the configuration and prepare the agent for testing:

Click Save and exit to save the multi-agent collaboration
Click Save and exit again to save the travel agent
In the Test panel at the right, click Prepare
In the Aliases section, click Create
For Alias name, enter dev
Click Create alias
Testing the travel agent
Next, you'll use a chatbot application to test the travel agent from a user's perspective.

Configuring the chatbot application
The chatbot uses a Lambda function called appsync-bedrock-resolver to communicate with the agent, as shown in the architecture. You’ll need to provide your agent’s identification so the function can invoke it properly:

Take note of the Agent ID: agent id
Take note of the Alias ID: agent id
Open the AWS Lambda console
Locate and open the function named appsync-bedrock-resolver
Navigate to the Configuration tab and find Environment variables
Set the environment variables:
AGENT_ID: Use your Travel Agent's ID
AGENT_ALIAS_ID: Use Travel Flight Agent's alias ID
Navigate to the chatbot application:
On the workshop page , locate the WebSiteUrl output and open the URL provided in a new browser tab to frontend client application:
Self-hosted event instructions
If you are following this workshop in the self-hosted mode, the WebSiteUrl output can be found in your CloudFormation Stack Outputs.

If the sign-in form asks you to verify your contact information, select skip Skip account verification
You will authenticate using the pre-created credentials:
Username: workshop_user
Password: Password1!
Interacting with the agent
Now, in the chatbot application select the Chatbot option under Bedrock Agents from the left hand pane and let's interact with the Travel agent.

chatbot application

Ask for a trip leaving on day 1st of any month, like:
I need help planning a trip from San Francisco to Los Angeles, departing December 1st and returning the following day.

Our weather service always issues an alert on the 1st of each month. Let's try to schedule a trip on that day to see how our agent behaves
As expected our agent found a potential disruption and provided recommendations:
I've found flight and weather information for your San Francisco to Los Angeles trip:

✈️ Flight Options:

Delta Airlines - $775
United Airlines - $660
American Airlines - $370
⚠️ Weather Alerts:

Severe Thunderstorm Warnings are in effect for both San Francisco and Los Angeles on December 1st
Temperatures will range from 50-71°F
December 2nd shows no weather alerts for Los Angeles
Recommendations:

Consider the American Airlines flight for the best price
Be prepared for potential flight delays due to thunderstorm warnings
Pack layers and a light waterproof jacket
Check for real-time flight updates closer to your travel date
Would you like to proceed with booking a flight or explore alternative dates?

Test and troubleshoot agent behavior
One neat feature about Amazon Bedrock Agent is the ability to provide the agent reasoning, which significantly enhances testability and troubleshooting. Developers can inspect how the agent interprets inputs, selects actions, and executes API calls. This makes it easier to identify issues, debug unexpected behaviors, and refine prompts or configurations. Additionally, structured reasoning allows for step-by-step validation, helping teams isolate failures, optimize responses, and ensure the agent behaves reliably in different scenarios.

We activate this feature by setting the enableTrace flag when calling the Amazon Bedrock Agent API from the appsync-bedrock-resolver Lambda function. To view the trace output in the chatbot, click on the Expand Reasoning link.

Travel agent sample trace

## Building agent with IaC

In this module, you'll create an agent to handle weather-related operations using Amazon Bedrock by leveraging the following capabilities:

Weather Forecast - Provide users with real-time or future weather forecast
Weather History - Provide users with historical weather data
Agent architecture
The weather agent is composed by the following components:

Amazon Bedrock: Hosting the agent AI model
AWS Lambda: Execute the business rules and serve as an integration layer
Weather Service: An external REST API providing weather information
Weather agent

You will create the Amazon Bedrock agent only. We will provide the other components needed.
Infrastructure as Code (IaC)
For the Weather Agent, you will use AWS CloudFormation to create the agent with Infrastructure as Code (IaC) instead of setting it up manually through the console. This approach offers several advantages over using the AWS Console:

Reproducibility: Easily recreate agents across different environments
Version control: Track changes to your agent configurations
Automation: Integrate agent creation into CI/CD pipelines
Consistency: Ensure consistent configuration across deployments
Understanding the CloudFormation Template
Let's examine the core parts of CloudFormation template that recreates the weather agent you created before using the console.

Parameters
The template accept as parameter the foundation model ID to use for th agent:

1
2
3
4
5
Parameters:
ModelId:
Type: String
Default: us.amazon.nova-pro-v1:0
Description: The foundation model ID to use for the agent

Amazon Bedrock agent
The core of the template is the agent definition:

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
BedrockAgent:
Type: AWS::Bedrock::Agent
Properties:
AgentName: weather-agent
AgentResourceRoleArn: !ImportValue BedrockAgentRoleArn
Description: Allow weather data retrieval such as forecasts and historical data
FoundationModel: !Ref ModelId
IdleSessionTTLInSeconds: 1800
Instruction: |
You are a weather specialist providing clear weather information to help users make informed travel decisions.
You can assume the airport code based on a city name.
For weather forecasts, you can provide expected temperature and any extreme weather alerts that could impact travel.
For historical weather analysis, you can provide average minimum and maximum temperatures and rainfall volume per month.
ActionGroups: - ActionGroupName: WeatherData
Description: Action group for weather-related operations
ActionGroupExecutor:
Lambda: !ImportValue WeatherAgentFunctionArn
FunctionSchema:
Functions: - Name: WeatherForecast
Description: Retrieve real-time or future weather forecast for a given location and date
Parameters:
location:
Type: string
Description: Airport code for the location
date:
Type: string
Description: Date of forecast in ISO 8601 format - Name: WeatherHistory
Description: Retrieve historical weather data for a given location
Parameters:
location:
Type: string
Description: Airport code for the location

Key components of the agent definition:

Basic Properties:

Name, description, and foundation model
IAM role for permissions
Session timeout (30 minutes)
Instructions:

Positions the agent as a weather insights specialist for travel planning
Focuses on providing actionable weather data for specific use cases
Specifies the types of information to include in forecasts and historical analysis
Action Groups:

Defines the capabilities of the agent
Specifies the Lambda function that will execute the actions
Defines the function schema with available operations
Function Schema:

Defines two functions: WeatherForecast and WeatherHistory
Specifies the parameters each function requires
Provides descriptions to help the agent understand when to use each function
Agent alias
The template also creates an alias for the agent:

1
2
3
4
5
6
BedrockAgentAlias:
Type: AWS::Bedrock::AgentAlias
Properties:
AgentAliasName: dev
AgentId: !GetAtt BedrockAgent.AgentId
Description: "Development version of the weather agent"

Deploying the agent
To deploy the agent using the CloudFormation template, follow these steps:

Follow this link to launch the CloudFormation stack in the US West 2 (Oregon) region

On the CloudFormation console:

Confirm that you're in the US West 2 (Oregon) region
The template URL should be pre-populated
Click Next
Stack details:

Stack name: lab3-weather-agent (or choose your preferred name)
Review the parameters but don't change anything
Click Next
Configure stack options:

Leave the default settings
Check the acknowledgment box for IAM resource creation
Click Next
Review and create:

Click Submit
Wait for deployment:

The stack creation will take less than a minute
You can monitor the progress in the CloudFormation console
Once the stack creation is completed you can navigate to the Amazon Bedrock console, and you'll find a new agent named weather-agent.

agent-list

Testing the weather agent
In the Test panel at the right:

Enter the following prompt and click Run:
When’s the best time of year to enjoy the beaches in MIA?

The agent should respond with a detailed forecast, including:
The best time of year to enjoy the beaches in Miami is during the spring (March to May) and fall (September to November) seasons. During these months, the weather is warm and dry, with average temperatures ranging from the mid-60s to mid-80s Fahrenheit. The lower rainfall during these periods also makes for ideal beach conditions. I would recommend planning your beach vacation in Miami during these times of the year to take advantage of the optimal weather and beach-going experience.

Congratulations!
You've successfully created a Weather Agent using Infrastructure as Code (IaC)!
