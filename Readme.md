>[!info]
>This mini project's objective is to streamline data insertion into DynamoDB through an S3 bucket using AWS Lambda, enhancing automation efficiency within a serverless architecture.

## Create an S3 Bucket
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/249545be-45c8-4e07-a1e3-ea4abce5fe57)
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/47c4af6f-0430-4c40-8bb1-4d69503476fb)

## Create a DynamoDB table
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/300fe42a-76f6-49fd-8653-8fff056745ea)
Keep other settings of the table as default and create the table.

![[Pasted image 20240322172032.png]]

## Create the lambda function 
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/bf30c0b0-542e-41f8-835d-14c16fc21904)
Rest of the settings are default now we can create the lambda function.

## Timeout setting
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/8516e172-8db3-40c2-8b2b-410b9caffc06)
In order to land to this page we need to select the lambda function → click on Configuration → click on General configuration → set the time limit.

## Permissions

![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/0449a0ce-aa9b-4172-8cd4-5bd3c635b57c)
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/85b1b0b2-9dc6-4e60-a3da-fa73cbee5ab4)
Policies have been attached to access the S3 bucket and the dynamoDB. In order to land to this page we need to click on the lambda function → select the permissions tab → select the add permissions to the S3 bucket and the dynamodb tables.

## Sample json
```txt
{
	"CustomerID": "01",
	"Product": "Coffee",
	"Address": "135, Rose avenue",
	"Quantity": "4"

}
```
Create in sample json file in notepad and upload that into the S3 bucket.
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/c8022c90-96b3-41ff-b68d-185448f207b4)
In the bottom we can notice that there is Key that had been created. Copy that, which shall be used in the boto3 code.

## Boto3 Code

### Uploading a JSON to S3 and converting that to a dictionary

Look into the request syntax mentioned in the documentation: 
https://boto3.amazonaws.com/v1/documentation/api/1.9.42/reference/services/s3.html#S3.Client.get_object
Look into the same link for the response syntax:
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/43c81974-a0aa-4d34-8428-93bcd200cad7)
This response structure is a dictionary

```python
import json
import boto3 
client = boto3.client('s3')

def lambda_handler(event, context):
    response = client.get_object(
	    Bucket='boto3-bucket-01',
	    Key='Dynamodb_sample.txt',
)

    json_data = response['Body']
    print(json_data)
    print(type(json_data))
```
For the first iteration deploy this code.
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/8c3330e7-4a96-4774-933b-a57fe603db1b)
>[!important]
>Whenever the code is changed we need to deploy the code first and then test the code.

![[Pasted image 20240322195411.png]]

![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/114ac987-ffb9-45b6-8017-c19c93885c14)
The code is functioning but the output is `botocore.response.StreamingBody` we need to understand what this means.
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/2d9ca1fb-7376-45d7-90a9-57b431d2bed1)
this class comes with some response, we need to use the method `read()` to understand the type of the output it gives. Now edit the code accordingly.
```python
```python
import json
import boto3 
client = boto3.client('s3')

def lambda_handler(event, context):
    response = client.get_object(
	    Bucket='boto3-bucket-01',
	    Key='Dynamodb_sample.txt',
)
# convert from streaming data
    json_data = response['Body'].read()
    print(json_data)
    print(type(json_data))
```

![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/a204e372-5bc3-4197-95aa-905ed492fa5e)
If we look into the results we can see it shows the output is in bytes. Which means all the contents that logs are displayed are in UTF-8.
>[!note]
>”The `<class 'bytes'>` indicates the data type of the byte string, which is bytes in Python. Bytes are a data type used to represent sequences of bytes (i.e., binary data) in Python. They are immutable and can contain any raw data, including text data encoded in various formats like UTF-8.”

Finally we are trying to convert the byte into a string and code is changed as follows:
```python
import json
import boto3 
client = boto3.client('s3')

def lambda_handler(event, context):
    response = client.get_object(
        Bucket='boto3-bucket-01',
        Key='Dynamodb_sample.txt',
)
# convert from streaming data to byte
    json_data = response['Body'].read()
# convert data from byte to string
    data_string = json_data.decode('UTF-8')
    print(data_string)
    print(type(data_string))
```
And the result is:
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/ac30025d-9822-435d-aa0b-092119422b64)

![[Pasted image 20240322195526.png]]

>[!note]
>**JSON keys can only be strings.** **The dictionary's keys can be any hashable object**. The keys in JSON are ordered sequentially and can be repeated. The keys in the dictionary cannot be repeated and must be distinct.

```python
import json
import boto3 
client = boto3.client('s3')

def lambda_handler(event, context):
    response = client.get_object(
        Bucket='boto3-bucket-01',
        Key='Dynamodb_sample.json',
)
# convert from streaming data to byte
    json_data = response['Body'].read()
# convert data from byte to string
    data_string = json_data.decode('UTF-8')
    # print(data_string)
    # print(type(data_string))
# convert from json string to dictionary
    data_dict = json.loads(data_string)
    print(data_dict)
    print(type(data_dict))
```
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/e93b8959-0c2a-4049-bfd7-bc595bc83da1)

### Inserting data to DynamoDB table

>[!note]
>Start off by adding the service resource `dynamodb=boto3.resource('dynamodb')`
>

https://boto3.amazonaws.com/v1/documentation/api/latest/guide/dynamodb.html#amazon-dynamodb

```python
import json
import boto3 
client = boto3.client('s3')
dynamodb=boto3.resource('dynamodb')

def lambda_handler(event, context):
    response = client.get_object(
        Bucket='boto3-bucket-01',
        Key='Dynamodb_sample.json',
)
# convert from streaming data to byte
    json_data = response['Body'].read()
# convert data from byte to string
    data_string = json_data.decode('UTF-8')
    # print(data_string)
    # print(type(data_string))
# convert from json string to dictionary
    data_dict = json.loads(data_string)
    print(data_dict)
    print(type(data_dict))
    
    table = dynamodb.Table('RetailSales770')
    table.put_item(
        Item=data_dict
    )
```

Result:
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/d22ec3c8-1aa0-48c1-b26c-3061a6a17c07)

### Triggers
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/eb91f64f-16f2-4cd2-bac4-3eaa0c642c99)
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/0d745d2c-2073-4ca8-97ef-c4f1f4f6fdf5)
Now if you modify the same file and upload it there will be new item added to it.
