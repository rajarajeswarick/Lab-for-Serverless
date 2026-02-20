# Lab-for-Serverless
Lab for creating Serverless architecture

<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/9cde5d49-294b-4c57-b5a9-ee2c9445f2de" />
An Amazon API Gateway is a collection of resources and methods. For this tutorial, you create one resource (DynamoDBManager) and define one method (POST) on it. The method is backed by a Lambda function (LambdaFunctionOverHttps). That is, when you call the API through an HTTPS endpoint, Amazon API Gateway invokes the Lambda function.

The POST method on the DynamoDBManager resource supports the following DynamoDB operations:

Create, update, and delete an item.
Read an item.
Scan an item.
Other operations (echo, ping), not related to DynamoDB, that you can use for testing.
The request payload you send in the POST request identifies the DynamoDB operation and provides necessary data. For example:

The following is a sample request payload for a DynamoDB create item operation:
```json
{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "1",
            "name": "Bob"
        }
    }
}
```

The following is a sample request payload for a DynamoDB read item operation:

```json
{
    "operation": "read",
    "tableName": "lambda-apigateway",
    "payload": {
        "Key": {
            "id": "1"
        }
    }
}
```
**Setup**

Create Custom Policy
We need to create a custom policy for least privilege

Open policies page in the IAM console
Click "Create policy" on top right corner
In the policy editor, click JSON, and paste the following

<img width="2519" height="1264" alt="image" src="https://github.com/user-attachments/assets/c33f592c-a3fd-42e4-9d22-7e490f887feb" />

```json

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
```
Give name "lambda-custom-policy", and click "Create policy" on botom right

**Create Lambda IAM Role**

Create the execution role that gives your function permission to access AWS resources.

To create an execution role

Open the roles page in the IAM console.

Choose Create role.

Create a role with the following properties.

Trusted entity type – AWS service, then select Lambda from Use case
Permissions – In the Permissions policies page, in the search bar, type lambda-custom-policy. The newly created policy should show up. Select it, and click Next.
Create IAM role

Role name – lambda-apigateway-role.
Click "Create role"

**Create Lambda Function**
To create the function

Click "Create function" in AWS Lambda Console
Create function
<img width="1801" height="1162" alt="image" src="https://github.com/user-attachments/assets/9b461e79-79f8-4272-8989-ec7d0c1ca916" />

Replace the boilerplate coding with the following code snippet and click "Deploy"
Example Python Code

```python
from __future__ import print_function
import boto3
import json

print('Loading function')


def lambda_handler(event, context):
    '''Provide an event that contains the following keys:

      - operation: one of the operations in the operations dict below
      - tableName: required for operations that interact with DynamoDB
      - payload: a parameter to pass to the operation being performed
    '''
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
'''

<img width="1835" height="942" alt="image" src="https://github.com/user-attachments/assets/2defc44d-312a-419e-acce-3fe1fc0f310d" />


Test Lambda Function
