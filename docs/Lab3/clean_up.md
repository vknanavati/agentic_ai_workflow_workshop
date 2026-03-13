# Clean up

Only clean up if you are running this on your own AWS account. If you are running this workshop at an AWS event, the resources will be automatically destroyed when the workshop ends.
In this section, we'll clean up all the resources created during the workshop to ensure you don't incur any unexpected charges.

CloudFormation Stacks
The easiest way to clean up most resources is to delete the CloudFormation stacks we created:

Open the AWS CloudFormation console
Delete the following stacks in this order:
agents-bedrock (if you deployed it)
weather-agent (if you deployed it)
bedrock-workshop-resources
Open the Amazon Bedrock console
Navigate to Agents and delete any agents you created
