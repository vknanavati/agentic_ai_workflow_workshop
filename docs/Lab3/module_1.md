# Module 1 - Building your first agent

In this module, you'll create an AI agent to handle flight-related operations using Amazon Bedrock by leveraging the following capabilities:

Flight Search - Helps users search for available flights based on their preferences
Flight Booking - Helps users in booking a flight
Flight Cancellation - Helps users to cancel an existing booking
Agent architecture
The flight agent is composed by the following components:

Amazon Bedrock: Hosting the agent AI model
AWS Lambda: Execute the business rules and serve as an integration layer
Flight Service: An external REST API providing flight management capabilities
Flight agent architecture

You will create the Amazon Bedrock agent only. We will provide the other components needed.
Create the Agent
Let's start by building our first agent using Amazon Bedrock!

Navigate to the Amazon Bedrock console
In the left navigation pane, click Agents
Click Create agent
For Name, enter flight-agent
For Description, enter Enable flight management activities such as searching, booking, and canceling flights
Leave the rest as default and click Create
Customize the Agent
Under the Agent details section:
For Agent resource, select Use an existing service role and enter the role prefixed with lab3-bedrock-BedrockAgentRole
In the Select model dropdown, select Amazon - Nova Pro 1.0 and click Apply
For Instructions for the Agent, enter:
You are a flight booking specialist with expertise in finding and managing flights based on user preferences. Your task is to search for available flights, book flights, and cancel reservations as needed. When searching for flights, consider the user’s preferred origin, destination, departure, and return dates, and any specified preferences such as airline or price range. When booking a flight, ensure the correct flight details are used and confirm the reservation. When canceling a flight, verify the confirmation details before processing the request. Ensure all responses are clear, accurate, and optimized for a seamless travel booking experience.

Click Save on the top
Configure Action Group
In the Action groups section, click Add
For Action group name, enter FlightManagement
For Action group invocation, select Select an existing Lambda function
In the Lambda function dropdown, select bedrock-flight-agent
Create the following action group functions:
Action group function 1: Flight Search
Create the flight search operation in the existing Action group function
Switch to JSON Editor mode
Copy and paste the following JSON definition:
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
{
"name": "FlightSearch",
"description": "Search for available flights",
"parameters": {
"returnDate": {
"description": "return date in ISO 8601 format",
"required": "True",
"type": "string"
},
"origin": {
"description": "origin airport code",
"required": "True",
"type": "string"
},
"destination": {
"description": "destination airport code",
"required": "True",
"type": "string"
},
"departureDate": {
"description": "departure date in ISO 8601 format",
"required": "True",
"type": "string"
}
},
"requireConfirmation": "DISABLED"
}

Action group function 2: Flight Booking
Click Add action group function to create the flight booking operation
Switch to JSON Editor mode
Copy and paste the following JSON definition:
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
{
"name": "FlightBooking",
"description": "Book a flight",
"parameters": {
"flightId": {
"description": "unique flight identifier",
"required": "True",
"type": "string"
}
},
"requireConfirmation": "DISABLED"
}

Action group function 3: Flight Cancellation
Click Add action group function to create the flight cancellation operation
Switch to JSON Editor mode
Copy and paste the following JSON definition:
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
{
"name": "FlightCancellation",
"description": "Cancel a flight",
"parameters": {
"confirmation": {
"description": "flight confirmation identifier",
"required": "True",
"type": "string"
}
},
"requireConfirmation": "DISABLED"
}

Deploying the agent
Save the configuration and prepare the agent for testing:

Click Create to save the action group
Click Save and exit to save the agent
In the Test panel at the right, click Prepare alias
In the Aliases section, click Create
For Alias name, enter dev
Click Create alias alias
To deploy your agent, you must create an alias . Aliases are important because:

They provide a stable reference point for your agent
They allow you to manage different versions of your agent
They enable you to deploy updates without changing integration points
Understanding action fulfilment with AWS Lambda
Now let's review how to use Lambda to fulfill the agent actions. Note, this is the same function referenced when we configured the Action group.

When Amazon Bedrock invokes the Lambda function specified in the action group, it provides as input event the user prompt and relevant metadata to execute action. The detailed API schema can be found in Bedrock documentation.

Here is an example of input event for the FlightSearch action you have just created.

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
{
messageVersion: '1.0',
function: 'FlightSearch',
parameters: [
{ name: 'returnDate', type: 'String', value: '2025-04-02' },
{ name: 'origin', type: 'String', value: 'SFO' },
{ name: 'destination', type: 'String', value: 'LAX' },
{ name: 'departureDate', type: 'String', value: '2025-04-01' }
],
sessionId: '1461a049-e61a-4c9c-be1c-3df576712d69',
agent: {
name: 'flight-agent',
version: '1',
id: 'Z7FHURX5OS',
alias: 'QO3FYAYSXN'
},
actionGroup: 'FlightManagement',
sessionAttributes: {},
promptSessionAttributes: {},
inputText: 'Please search for round-trip flights from SFO to LAX departing on April 1st and returning April 2nd.'
}

To execute the action, we will first parse the input parameters reading event.parameters property.

1
2
3
4
const parameters = {};
for(const parameter of event.parameters) {
parameters[parameter.name] = parameter.value;
}

Next, we identify which action should be performed based on event.function property and route the request to the appropriate business method. Note that this is necessary as an action group can have multiple actions.

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
let result;

switch (event.function) {
case 'FlightSearch':
const search = {
origin: parameters.origin,
destination: parameters.destination,
departureDate: parameters.departureDate,
returnDate: parameters.returnDate
};
console.log('Searching flight: ', search);
result = await flightSearch(search);

      break;

case 'FlightBooking':
const flight = {
id: parameters.flightId
};
console.log('Reserving flight: ', flight);
result = await flightBook(flight);

      break;

case 'FlightCancellation':
const reserve = {
confirmation: parameters.confirmation
};
console.log('Cancelling flight: ', reserve);
result = await flightCancel(reserve);

      break;

default:
throw new Error('Function not supported');
}

Finally, we wrap the result in the format expected by the Amazon Bedrock agent and return the response.

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
console.log('Result: ', result);

const response = {
messageVersion: event.messageVersion,
response: {
actionGroup: event.actionGroup,
function: event.function,
functionResponse: {
responseBody: {
'TEXT': {
body: JSON.stringify(result)
}
}
}
}
};

return response;

In the business method implementation, you can execute any custom logic required to fulfill the action, such as integration with other AWS services or external services. For the Flight Agent implementation, we will invoke an external API. To review the full implementation, navigate to the Lambda console and check out the bedrock-flight-agent function code.

Testing the Agent with Amazon Bedrock console
In the Test panel at the right:

Enter the following prompt and click Run
What flight options do I have from SFO to JFK departing on June 5th and returning 10th?

The agent should respond with a list of available flight options, similar to the one below:
I found 3 flight options for your SFO to JFK round trip:

Delta Airlines - $500
United Airlines - $450
American Airlines - $400 Would you like to book one of these flights?
Let the agent know the know which one you want to proceed
Let's book the cheapest one

It should confirm the flight for you
Your flight has been successfully booked. The booking locator code is UXLF71. Safe travels!
