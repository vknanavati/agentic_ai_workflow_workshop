**March 16**
Lab 3 Module 2:
_Objective_:

- The weather agent is composed by the following components:
  - Amazon Bedrock: Hosting the agent AI model
  - AWS Lambda: Execute the business rules and serve as an integration layer
  - Weather Service: An external REST API providing weather information

Lab 2 Module 5
_Objective_:

- create a Amazon Bedrock AgentCore Gateway to convert an existing Lambda Function into a fully managed MCP Server

- When making Gateway we set the target as the Lambda Function

**March 15**
Lab 2
_Learned about_:

- different than previous model because instead of the state machine step functions to handle the agentic loop we use code
- Agent Strands
- S3 vector store
- Retrieval Augmented Generation (RAG)

- ran this command to upload S3 doc to bucket:
  export BUCKET=agentic-ai-workshop-vn-lab2-travelagents3bucketrag-uk1ayek0ok4x

aws s3 cp seattletouroperators.txt s3://$BUCKET/

- knowledge base ID: XNWMVAGJCJ

- moving from managed workflow orchestration to autonomous agent reasoning.

- Strands Agents can use the MCP Client Tool available with Strands SDK to connect to external MCP Servers and dynamically load remote tools.

- Lab 2 Module 5:
  use Bedrock AgentCore Gateway to convert an existing Lambda Function into a fully managed MCP Server
- Cognito Client Credentials The following Cognito resources have been created for your gateway:
  Cognito User Pool (ID: us-west-2_X2WSLbQB1)
  User Pool Domain: my-domain-1ae4qmgl.auth.us-west-2.amazoncognito.com
  Resource Server with scope: genesis-gateway:invoke
  User Pool Client with client credentials flow
  Client ID: 4r7aalnghgsr34oek1oe7ad6es Client Secret: 1obs9dm9535pk0cfubm51499feu557elnu01d9alnqa7cie52025 Important: Please save these credentials. The client secret will only be shown once.

- === AgentCore Gateway: TravelAttractionsGateway ===
  GATEWAY_URL: https://travelattractionsgateway-aemwoncsoo.gateway.bedrock-agentcore.us-west-2.amazonaws.com/mcp
  CLIENT_ID: 4r7aalnghgsr34oek1oe7ad6es
  CLIENT_SECRET: 1obs9dm9535pk0cfubm51499feu557elnu01d9alnqa7cie52025
  DOMAIN_URL: https://my-domain-1ae4qmgl.auth.us-west-2.amazoncognito.com

ID_TOKEN=$(echo $TOKEN_RESPONSE | jq -r '.AuthenticationResult.IdToken')
~ $ echo "ID Token: $ID_TOKEN"
ID Token: eyJraWQiOiJOQXV2VUhxanZmUjRNYXpKS29MU3dIaUtJNUtuSHEyaWFPOVBCYzRaK2FzPSIsImFsZyI6IlJTMjU2In0.eyJzdWIiOiI5OGExNTM2MC03MDMxLTcwOWYtMTA4Ni1kZGJhMzFhMTcyNTEiLCJlbWFpbF92ZXJpZmllZCI6dHJ1ZSwiaXNzIjoiaHR0cHM6XC9cL2NvZ25pdG8taWRwLnVzLXdlc3QtMi5hbWF6b25hd3MuY29tXC91cy13ZXN0LTJfUzFxaU5IWVYyIiwiY29nbml0bzp1c2VybmFtZSI6InRlc3R1c2VyIiwib3JpZ2luX2p0aSI6IjViZDA5NjY4LTg1MGYtNDZhMC05YWJmLTVkZjA3OWU5MmM4NyIsImF1ZCI6IjNvMTMxdGNiMmwxcmxubGpnZW9yOHIxbjN1IiwiZXZlbnRfaWQiOiIzMTAzZWQxZS1kY2I3LTRjOTctYjZiZS05MTIxMjZmMWRkNzIiLCJ0b2tlbl91c2UiOiJpZCIsImF1dGhfdGltZSI6MTc3MzU4NjYyOCwiZXhwIjoxNzczNTkwMjI4LCJpYXQiOjE3NzM1ODY2MjgsImp0aSI6ImI5MGFhNDhiLTRjODgtNDc1Yy05Y2VlLTRmMzA3ZGRjYjQ4NCIsImVtYWlsIjoidGVzdEBleGFtcGxlLmNvbSJ9.HvXj0cVNxOGMAaGKtEmp5dTrxSaI9jGQbbCjBF0V1Z_bdG2gEWbMxHj2WHWzb518-AcI-N8PmSLeUEVYwsvlB5ing3Bg8CBYJMpy51jgUdpVTDNYWY3omCxFG8ntb-H2UUNTinLASo2cO1hHXhHYrqoeOHmhVrA2z2S8FWm4ySudRsnDPUnvQFmDT2feuVofs7SEPdiErVvXt18Mx52WAgPuPCg2n_Usb0cLB9ei1i9rU121Ks-fwuUIWxNpY_VlJBC8g_f3ccpmLeD6Kek1siu2unangeS1B1uyp5k-RPhYMl06eaJHyM4V2O784C_g6SrSxqIk94HQyUlhF_i43w

curl -X POST https://fx27yod449.execute-api.us-west-2.amazonaws.com/prod/chat \
 -H "Authorization: $ID_TOKEN" \
 -H "Content-Type: application/json" \
 -d '{"prompt": "Can you tell me travel options to Seattle?"}' | jq
