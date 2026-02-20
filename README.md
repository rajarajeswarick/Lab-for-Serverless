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

```json
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
Let's test our newly created function. We haven't created DynamoDB and the API yet, so we'll do a sample echo operation. The function should output whatever input we pass.

Click the "Test" tab right beside "Code" tab

Give "Event name" as echotest

Paste the following JSON into the event. The field "operation" dictates what the lambda function will perform. In this case, it'd simply return the payload from input event as output. Click "Save" to save

{
    "operation": "echo",
    "payload": {
        "somekey1": "somevalue1",
        "somekey2": "somevalue2"
    }
}
Click "Test", and it will execute the test event. You should see the output in the console
Execute test event

We're all set to create DynamoDB table and an API using our lambda as backend!

Create DynamoDB Table
Create the DynamoDB table that the Lambda function uses.

To create a DynamoDB table

Open the DynamoDB console.
Choose "tables" from left pane, then click "Create table" on top right.
Create a table with the following settings.
Table name – lambda-apigateway
Partition key – id (string)
Choose "Create table".
create DynamoDB table

Create API
To create the API

Go to API Gateway console
Click Create API
Scroll down and select "Build" for REST API
Give the API name as "DynamoDBOperations", keep everything as is, click "Create API"
Create REST API

Each API is collection of resources and methods that are integrated with backend HTTP endpoints, Lambda functions, or other AWS services. Typically, API resources are organized in a resource tree according to the application logic. At this time you only have the root resource, but let's add a resource next. Click "Create Resource"

Input "DynamoDBManager" in the Resource Name. Click "Create Resource"

Create resource

Let's create a POST Method for our API. With the "/dynamodbmanager" resource selected, click "Create Method".
Create resource method

Select "POST" from drop down.

Integration type should be pre-selected as "Lambda function". Select "LambdaFunctionOverHttps" function that we created earlier. As you start typing the name, your function name will show up.Select the function, scroll down and click "Create method".

Create lambda integration

Our API-Lambda integration is done!

Deploy the API
In this step, you deploy the API that you created to a stage called prod.

Click "Deploy API" on top right

Now it is going to ask you about a stage. Select "[New Stage]" for "Stage". Give "Prod" as "Stage name". Click "Deploy"

Deploy API to Prod Stage

We're all set to run our solution! To invoke our API endpoint, we need the endpoint url. In the "Stages" screen, expand the stage "Prod", keep expanding till you see "POST", select "POST" method, and copy the "Invoke URL" from screen
Copy Invoke Url

Running our solution
The Lambda function supports using the create operation to create an item in your DynamoDB table. To request this operation, use the following JSON:
{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "1234ABCD",
            "number": 5
        }
    }
}
To execute our API from local machine, we are going to use Postman and Curl command. You can choose either method based on your convenience and familiarity.

To run this from Postman, select "POST" , paste the API invoke url. Then under "Body" select "raw" and paste the above JSON. Click "Send". API should execute and return "HTTPStatusCode" 200.
Execute from Postman

To run this from terminal using Curl, run the below
$ curl -X POST -d "{\"operation\":\"create\",\"tableName\":\"lambda-apigateway\",\"payload\":{\"Item\":{\"id\":\"1\",\"name\":\"Bob\"}}}" https://$API.execute-api.$REGION.amazonaws.com/prod/DynamoDBManager
To validate that the item is indeed inserted into DynamoDB table, go to Dynamo console, select "lambda-apigateway" table, select "Explore table items" button from top right, and the newly inserted item should be displayed.

Dynamo Item

Dynamo Itemshow

To get all the inserted items from the table, we can use the "list" operation of Lambda using the same API. Pass the following JSON to the API, and it will return all the items from the Dynamo table
{
    "operation": "list",
    "tableName": "lambda-apigateway",
    "payload": {
    }
}
List Dynamo Items

We have successfully created a serverless API using API Gateway, Lambda, and DynamoDB!

Cleanup
Let's clean up the resources we have created for this lab.

Cleaning up DynamoDB
To delete the table, from DynamoDB console, select the table "lambda-apigateway", then on top right , click "Actions", then "Delete table"

To delete the Lambda, from the Lambda console, select lambda "LambdaFunctionOverHttps", click "Actions", then click Delete

To delete the API we created, in API gateway console, under APIs, select "DynamoDBOperations" API, click "Delete"


