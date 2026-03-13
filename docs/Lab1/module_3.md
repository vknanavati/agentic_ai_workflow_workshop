# Module 3 - Understanding Step Functions
Code-based vs workflow-based Agent orchestration
When building AI agents, you have two main ways to control how they work:

The first is code-based orchestration where you write all the logic in your application code that handles the agent's thinking, tool use, and memory. This gives you full control and keeps everything simple since all the logic stays in one place. You can implement exactly what you need without learning new tools or services. However, you must build everything yourself - including error handling, state tracking, and monitoring. Additionally, using a compute service like Lambda might not be ideal since you're paying for compute time while the agent function sits idle waiting for tool responses.

Many teams start with code-based orchestration and transition to workflow-based solutions as their agents become more sophisticated. In workflow-based orchestration, rather than writing imperative code that dictates agent behavior, you define the agent's actions at each step and let the workflow engine manage the execution details. This declarative approach offers several key advantages. The workflow engine automatically manages state transitions and provides built-in error handling and retry mechanisms. It can efficiently pause execution while waiting for external responses, leading to reduced compute costs. This approach improves scalability and simplify maintenance of your agent.

Why Step Functions?
AWS Step Functions  is the perfect solution for orchestrating interactions between models, tools, and users. It provides a visual workflow service that simplifies complex request-response patterns through an intuitive interface. By allowing you to build and visualize workflows graphically, Step Functions makes it easier to manage complex chains of interactions and monitor their execution in real-time. As a serverless service, it eliminates concerns about infrastructure maintenance, version management, and scalability. The platform offers robust capabilities to handle sophisticated business logic and conditional branching in your workflows. Step Functions directly integrates with over 11,000 AWS APIs across 200+ services, including Amazon Bedrock.

Create the Step Functions workflow
In this section, you will build and run a Step Functions workflow to invoke Amazon Bedrock API.

Open the Step Functions console 
Create a State Machine using Blank template.
Enter State machine name as lab1-sfn-hello-world
Leave the workflow type as Standard and click Continue
Explore the visual designer studio. On the left, you see Actions, Flow and Patterns. You can search for actions, drag and drop them in the middle section, graphical designer layout. On the right, you see the options to configure the steps as you work on them. You can switch to different views such as code, config and design using the tab at the top.

In the left panel, under Actions tab search for Converse
Drag and drop the Amazon Bedrock Runtime Converse to the designer canvas
In the right panel, under Arguments & Output tab update the below code under Arguments section
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
{
  "ModelId": "us.amazon.nova-lite-v1:0",
  "Messages": [
    {
      "Role": "user",
      "Content": [
        {
          "Text": "What is Amazon Bedrock Converse API?"
        }
      ]
    }
  ]
}

stepfn-bedrock

Important
As you are creating the workflow, you will see a red banner at the top. Ignore it. It will disappear as you make progress and save the workflow

Navigate away from the right side of the screen to the top section.
Select Config at the top. This page allows you to configure workflow name, IAM role, and logging etc.
Under Permissions, choose the execution role prefixed with lab1-sfn-StepFunctionsExecutionRole
Click Create on the top right
Run the workflow
Click Execute on the top right
Do not change anything in the input and click Start execution. stepfn-bedrock
Select Converse task in the execution and explore the State Output
Congratulations!
You have successfully invoked a converse API in Amazon Bedrock using Step Functions.

