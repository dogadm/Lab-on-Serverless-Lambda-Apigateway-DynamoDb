
# LAB FOR SERVERLESS

## High level/Lab Overview

![Serverless Lab Diagram](/Images/Serverless%20Lab%20Design.jpeg)

Amazon API Gateway consists of various resources and methods. In this tutorial, you will be creating a resource called DynamoDBManager and defining a method, namely POST, for it. 
This method is supported by a Lambda function named LambdaFunctionOverHttps. 
Essentially, when you make an API call through an HTTPS endpoint, Amazon API Gateway triggers the associated Lambda function.

The POST method available for the DynamoDBManager resource enables the execution of the following DynamoDB operations:

*Creating, updating, and deleting an item.
*Reading an item.
*Scanning an item.
*Additional operations (such as echo and ping) that are unrelated to DynamoDB, primarily utilized for testing purposes.

To perform these operations, you need to provide a request payload in the POST request, which specifies the DynamoDB operation and includes the required data. 
For instance, when performing a create item operation, the request payload should resemble the following sample:

```bash
{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "5",
            "name": "Nkechi"
        }
    }
}
```

Similarly, for a read item operation, the request payload should be structured as shown in this sample:

```bash
{
    "operation": "read",
    "tableName": "lambda-apigateway",
    "payload": {
        "Key": {
            "id": "5"
        }
    }
}
```

These request payloads allow you to specify the desired DynamoDB operation and provide the necessary data for its execution.

## SETUP

#### Create Lambda IAM Role

To grant your function access to AWS resources, you need to create an execution role.

Follow these steps to create the execution role:

* Access the IAM console and navigate to the roles page.
* Click on "Create role."
* Create a role with the specified properties:
    * Trusted entity: Lambda
    * Role name: lambda-apigateway-role
    * Permissions: Create a custom policy granting access to DynamoDB and CloudWatch Logs. This policy should include the necessary permissions for the function to write data to DynamoDB and upload logs.

```bash

{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "Statement1",
			"Effect": "Allow",
			"Action": [
				"dynamodb:GetItem",
				"dynamodb:Query",
				"dynamodb:Scan",
				"dynamodb:PutItem",
				"dynamodb:UpdateItem",
				"dynamodb:DeleteItem"
			],
			"Resource": [
				"*"
			]
		},
		{
			"Sid": "Statement2",
			"Effect": "Allow",
			"Action": [
				"logs:CreateLogGroup",
				"logs:CreateLogStream",
				"logs:PutLogEvents"
			],
			"Resource": [
				"*"
			]
		}
	]
}

``` 

#### Create Lambda Function

##### To create a function in the AWS Lambda Console, follow these steps: 

![ createfuntion page from aws console](/Images/createfuntion%20page%20from%20aws%20console.jpeg)

- Click on the "Create function" button.
- Choose "Author from scratch" as the starting point. Give the function the name "LambdaFunctionOverHttps" and select Python 3.7 as the runtime.
- In the Permissions section, select "Use an existing role" and choose the previously created "lambda-apigateway-role" from the dropdown menu.
- Click on the "Create function" button to proceed.

![ createfuntion page 2 from aws console](/Images/createfuntion%20page%202%20from%20aws%20console.jpeg)

![ LambafunctionOverHttps page 3 from aws console](/Images/LambafunctionOverHttps%20page%203.jpeg)


After creating the function, you will be taken to the Lambda basic information page. To replace the default boilerplate code, simply copy and paste the desired code snippet into the editor. Once you've made the changes, click on the "Save" and "Deploy" button to save the modified function.

##### Example Code(Python)

```bash

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

```




#### Test Lambda Function

To test our newly created function before creating DynamoDB and the API, we will perform a sample echo operation. This operation will output whatever input we provide.

Follow these steps to configure and execute a test event:

- Click on the arrow next to "Select a test event" and choose "Configure test events".

![ LambafunctionOverHttps page 4 from aws console](/Images/LambafunctionOverHttps%20page%204.jpeg)

- In the test events configuration window, paste the provided JSON into the event. The "operation" field specifies the action the lambda function will perform, which in this case is simply returning the payload from the input event as output. Click "Create" to save the test event.

``` bash
{
    "operation": "echo",
    "payload": {
        "somekey1": "somevalue1",
        "somekey2": "somevalue2"
    }
}

```
![ LambafunctionOverHttps page 5 from aws console](/Images/LambafunctionOverHttps%20page%205.jpeg)

Click on "Test" to execute the test event. The output will be displayed in the console.

![ LambafunctionOverHttps page 6 from aws console](/Images/LambafunctionOverHttps%20page%206.jpeg)

![ LambafunctionOverHttps page 7 from aws console](/Images/LambafunctionOverHttps%20page%207.jpeg)

Now, we are ready to proceed with creating the DynamoDB table and an API using our lambda function as the backend.

#### Create DynamoDb Table


To set up the DynamoDB table required for the Lambda function, follow these steps:

- Access the DynamoDB console.
- Select "Create table."
- Configure the table with the following settings:
    - Table name: lambda-apigateway
    - Primary key: id (string)
- Click on "Create" to create the table.

![ dynamodb page 1 from aws console](/Images/dynamodb%20page%201.jpeg)

#### Create API 

To create the API and integrate it with Lambda, follow these steps:

- Access the API Gateway console.
- Click on "API"

![ api page 1 from aws console](/Images/api%20page%201.jpeg)

- Scroll down and choose "Build" for a REST API.

![ api page 2 from aws console](/Images/api%20page%202.jpeg)

- Provide the API name as "DynamoDBOperations" and keep the default settings. Click on "Create API."

![ api page 3 from aws console](/Images/api%20page%203.jpeg)

Now, let's add a resource to the API:

- Click on "Actions" and select "Create Resource."

![ api page 4 from aws console](/Images/api%20page%204.jpeg)

- Enter "DynamoDBManager" as the Resource Name, and the Resource Path will be populated automatically. Click on "Create Resource."

![ api page 5 from aws console](/Images/api%20page%205.jpeg)

Next, we will create a POST Method for the API:

- With the "/dynamodbmanager" resource selected, click on "Actions" again and choose "Create Method."

![ api page 6 from aws console](/Images/api%20page%206.jpeg)

- Select "POST" from the dropdown menu and click the checkmark to confirm.

![ api page 7.1 from aws console](/Images/api%20page%207.jpg)

For the integration with Lambda:

- The integration options will appear automatically, with "Lambda Function" selected. Choose the "LambdaFunctionOverHttps" function that was created earlier. As you start typing the name, it should show up in the options. Select it and click on "Save."
- A popup window will appear to add a resource policy for the Lambda function to be invoked by this API. Click on "Ok."

![ api page 8 from aws console](/Images/api%20page%208.jpeg)

Congratulations! The API-Lambda integration is now complete.

#### Deploy the API

To deploy the created API to a stage called "prod," follow these steps:

- Click on "Actions" and choose "Deploy API."

![ deployapi page 1 from aws console](/Images/deployapi%20page%201.jpeg)

- When prompted for the deployment stage, select "[New Stage]." Set the "Stage name" as "Prod." Click on "Deploy."

![ deployapi page 2 from aws console](/Images/deployapi%20page%202.jpeg)

Now, we are ready to run our solution! To invoke the API endpoint, we need the endpoint URL. Here's how to obtain it:

- In the "Stages" screen, expand the "Prod" stage.
- Select the "POST" method.
- Copy the "Invoke URL" displayed on the screen.

![ deployapi page 3 from aws console](/Images/deployapi%20page%203.jpeg)

You now have the API endpoint URL ready to be used.

#### Running our solution

- The Lambda function supports the create operation, which allows you to create an item in your DynamoDB table. To request this operation, use the following JSON:

```bash
{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "5678DCBA",
            "number": 9
        }
    }
}
```

- To execute the API from your local machine, you can choose between using Postman or a Curl command, based on your convenience and familiarity.

For Postman, follow these steps:

    - Select "POST" in Postman and paste the API invoke URL.
    - Under the "Body" section, select "raw" and paste the JSON mentioned above.
    - Click "Send" to execute the API. The response should show an "HTTPStatusCode" of 200.

![ postman page 1 from aws console](/Images/postman%20page%201.jpeg)

    - To execute the API from the terminal using Curl, run the following command:

```bash
$ curl -X POST -d "{\"operation\":\"create\",\"tableName\":\"lambda-apigateway\",\"payload\":{\"Item\":{\"id\":\"1\",\"name\":\"Nkechi\"}}}" https://$API.execute-api.$REGION.amazonaws.com/prod/DynamoDBManager
```

- To validate that the item is inserted into the DynamoDB table, navigate to the DynamoDB console, select the "lambda-apigateway" table, go to the "Explore table Items" tab, and you should see the newly inserted item displayed.

![ dynamodb page 1 from aws console](/Images/dynamodb%20page%201.jpeg)

To retrieve all the inserted items from the table using the "list" operation, pass the following JSON to the API:

```bash
{
    "operation": "list",
    "tableName": "lambda-apigateway",
    "payload": {}
}
```

![ postman page 2 from aws console](/Images/postman%20page%202.jpeg)

Executing this will return all the items from the DynamoDB table.

Congratulations! You have successfully created a serverless API using API Gateway, Lambda, and DynamoDB!

## Cleanup 

Let's clean up the resources we have created for this lab.

#### Cleaning up DynamoDB

To remove the table, Lambda function, and API that were created, follow these steps:

- To delete the DynamoDB table, access the DynamoDB console, select the "lambda-apigateway" table, and click on "Delete table."

![ cleanup dynamodb page 1 from aws console](/Images/cleanup%20dynamodb%20page%201.jpeg)

- To delete the Lambda function, go to the Lambda console, select the "LambdaFunctionOverHttps" function, click on "Actions," and then choose "Delete."

![ cleanup lambda page 1 from aws console](/Images/cleanup%20lambda%20page%201.jpeg)

- To delete the API that was created, navigate to the API Gateway console, go to the "APIs" section, select the "DynamoDBOperations" API, click on "Actions," and then choose "Delete."

![ cleanup api page 1 from aws console](/Images/cleanup%20api%20page%201.jpeg)

By following these steps, you can delete the DynamoDB table, Lambda function, and API from their respective consoles.

---

## 🔗 Links
[![portfolio](https://img.shields.io/badge/my_portfolio-000?style=for-the-badge&logo=ko-fi&logoColor=white)](https://dolapoadeola.framer.website/)
[![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/adeoladolapo/)

---







