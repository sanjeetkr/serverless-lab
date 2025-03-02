# serverless-lab
An Amazon gateway is an interface between clients and backend services, allowing users to expose AWS Lambda functions, HTTP endpoints, and other AWS services via RESTful and WebSocket APIs. Here, we create one API named DynamoDBManager and one POST method on it backed by LambdaFunctionOverHttps a Lambda function. Lambda function is inserting the data in lambda-apigateway a DynamoDB database.

POST method DynamoDBManager supports below DynamoDB operations
Create an item
Read an item
List all items
Built-in logging and monitoring via CloudWatch
The JSON request which you send in the POST request identifies the operation and provides respective data.

✅ Tip: Used Postman a thrid party application to send HTTP request to access DynamoDBManager API

SETUP
Create Lambda IAM Role
Configured an custom IAM role to give executional permissions to access AWS resources. To do so follow

Open IAM -> Policies in AWS console
Create a policy to give permissions – Custom policy with permission to DynamoDB and CloudWatch Logs. This custom policy has the permissions that the function needs to write data to DynamoDB and upload logs.
Below is a JSON for IAM policy:
{
"Version": "2012-10-17",
"Statement": [
{
  "Sid": "Stmt1428341300017",
  "Action": [
    "dynamodb:DeleteItem",
    "dynamodb:GetItem",
    "dynamodb:PutItem",
    "dynamodb:Query",
    "dynamodb:Scan",
    "dynamodb:UpdateItem"
  ],
  "Effect": "Allow",
  "Resource": "*"
},
{
  "Sid": "",
  "Resource": "*",
  "Action": [
    "logs:CreateLogGroup",
    "logs:CreateLogStream",
    "logs:PutLogEvents"
  ],
  "Effect": "Allow"
}
]
}
Open IAM -> Roles to create a new role
Trust Entity - Lambda
Role name - lambda-apigateway-role
Attach above policy
Create Lambda Function
GO to Lambda on AWS Console and click "Create API" image

Select "Author from Scratch"

Give Function name as "LambdaFunctionOverHttps"

Select Runtime language as "Python 3.13"

Architecture as x86_64

Click "Create function" image

Click on the Lambda function and replace the boilerplate coding with below code

from __future__ import print_function

import boto3
import json

  print('Loading function')
 def lambda_handler(event, context):
 #Provide an event that contains the following keys:

 #  - operation: one of the operations in the operations dict below
 #  - tableName: required for operations that interact with DynamoDB
 #  - payload: a parameter to pass to the operation being performed
 
 #print("Received event: " + json.dumps(event, indent=2))

 operation = event['operation']

 if 'tableName' in event:
     dynamo = boto3.resource('dynamodb').Table(event['tableName'])

 operations = {
     'create': lambda x: dynamo.put_item(**x),
     'read': lambda x: dynamo.get_item(**x),
     'update': lambda x: dynamo.update_item(**x),
     'delete': lambda x: dynamo.delete_item(**x),
     'list': lambda x: dynamo.scan(**x),
     'echo': lambda x: x,
     'ping': lambda x: 'pong'
 }

 if operation in operations:
     return operations[operation](event.get('payload'))
 else:
     raise ValueError('Unrecognized operation "{}"'.format(operation))
Test Lambda Function
Let's test our Lambda function before delploying and as we haven't created our API and DynamoDB so this will be just echo test. Hence, function should output whatever input we pass

Click on the "Test" tab on the Lambda function
Paste the below JSON
{
    "operation": "echo",
    "payload": {
        "somekey1": "somevalue1",
        "somekey2": "somevalue2"
    }
}
Click "Save" image

Click "Test" in the "Code" tab image

Once you are happy with the testing then Click "Deploy" image

We are all set to create DynamoDB table and API using our Lambda as backend!
Create API
Create DynamoDBOperations API following below steps

Go to API Gateway on AWS Console

Click Create API image

Scroll down and Select "Build" for Rest API image

Give API name as "DynamoDBOperations" and click "Create API" image image

API is a collection of resources and methods are integrated with backend HTTP endpoints, Lambda functions, or other AWS services. Click "Create Resource"
image

Mention resource name "DynamoDBManager" and click "Create resource"
image

Let's create a POST method for our API. Select "/DynamoDBManager" and click "Create Method"
image

Select

Method type: POST
Integration type: Lambda
Lambda function: LambdaFunctionOverHttps (which we created earlier)
Click "Create method" image
Create Amazon DynamoDB Table
Create the DynamoDB table that the Lambda function uses.

Follow below steps to create DynamoDB table
Go to DynamoDB service on AWS console
Select "Create table" image
Provide Table name: lambda-apigateway Primary Key/ Partition Key: id (string)
Select "Create table" image image image
Access and test API using Postman application
Postman is primarily used for testing and developing APIs, allowing developers to easily create, send, and analyze API requests, making it a key tool for managing the entire API lifecycle, from design and testing to documentation and collaboration. Follow below steps to access our API:

Create a Balnk Collection which is nothing but like creating a folder
In collection create new request by clicking ... and click "Add Request"
Select method "POST" from drop down
Add the API URL from the "Invoke URL" attribute of the API POST method in the PROD stage. image image image
Test new Load Testing feature of Postman
Postman enables you to simulate user traffic and observe how your API behaved under load. It also helps to identify any issues or bottlenecks that affect performance.

Configuration
TImeout: 5 sec to 3 sec
Limit instances: 10
Memeory: 128MB to 1024MB
Reserved Concurrency: 10
🚀Observed significant performance boost after adjusting Lambda configurations. This enabled average response time improved drastically from 315ms to 94ms! 🚀
