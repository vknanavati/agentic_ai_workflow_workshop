# Running workshop in your own AWS account

Self-hosting the workshop will incur costs related to the provisioned and used resources. Follow the clean up module instructions after finishing the workshop to avoid unnecessary costs.

This section provides instructions for participants who are running this workshop in their own AWS account rather than in an AWS-provided event environment.

Prerequisites
Before you begin, ensure you have:

An AWS account with administrative permissions
Access to the AWS Management Console
Familiarity with AWS CloudFormation
Deployment Steps
Deploy the CloudFormation Stack

Click the link below to launch the CloudFormation stack in the US West 2 (Oregon) region:

Launch Stack

On the CloudFormation console:

Confirm that you're in the US West 2 (Oregon) region
The template URL should be pre-populated
Click Next
Stack details:

Stack name: agentic-ai-workshop (or choose your preferred name)
Review the parameters but don't change anything
Click Next
Configure stack options:

Leave the default settings
Check the acknowledgment box for IAM resource creation
Click Next
Review:

Click Create stack
Wait for deployment:

The stack creation will take approximately 5-15 minutes
You can monitor the progress in the CloudFormation console
Next Steps
After successfully deploying the CloudFormation stack, go to Introduction to Agentic AI and proceed with the workshop.
