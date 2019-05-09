# AWS Remote System Manager Pattern
How do you execute remote commands on EC2 instances easily and securely. Sure, there are RMM tools by the dozen out there to install agents on servers and provide execution. I have however created a pattern that uses built-in AWS services to allow this functionality. 

I was originally asked by one of our clients to automate an IIS website setup for one of their webservers based on values in a DynamoDB table. I came up with a lambda function using SSM to execute powershell commands on that webserver. We have since used this similar pattern to automate few other functions on Windows servers. We added an API Gateway in front of the Lambda function to create an http end point that can be triggered in number of ways.

Below is a quick overview diagram of this pattern.
![Remote System Manager Pattern](https://github.com/sadiqinz/AWSRemoteSystemManager/blob/master/AWSSystemManagerPattern.png)


## Getting Started
You can use the CloudFormation script here to create following resources.

1. API Gateway
2. Lambda Functin
3. EC2 Instance

Once the script is executed it will output URL for API Gateway as well as IP Address of the EC2 Instnace. As part of the script, it installs IIS on the EC2 instance and also creates a PowerShell script which will be used for updating index.html page on that server. 

## Prerequisites
You will need to create an EC2 Key and provide that as an input parameter for the CF script. 
You will also need to create a S3 bucket and copy over the Lambda Function "ssmLambdaFunction.zip" to that bucket. Make sure that S3 bucket is created in the same region as the one you will be executing CF script in. You will not be able to create a Lambda Function from a bucket in a different region.

## Installing
As part of launching the script you will need to specify the following three parameters.

1. S3 Bucket Name (This is where the Lambda Function is stored)
2. AWS Lambda Function File Name (Name of the Lambda Function File Name)
3. Instance Type (EC2 Instance Type, allowed values are t2.micro or t2.medium)
4. EC2 Key Name (EC2 KeyName to be used for this EC2 instance)

Once the CF script is executed you will need to do one more thing before this will work.
There is an issue with API Gateway where permission to execute Lambda Function is not granted via the CF script. You will have to perform that step manually by editing the Lambda Function under "Integration Request" and then click on save. 

### Adding Lambda Execution Permission

![Lambda Execution Permission](https://github.com/sadiqinz/AWSRemoteSystemManager/blob/master/EditingLambdaFunction.gif)

## Testing
You will get API Gateway's URL and external IP Address of EC2 instance as an ouput of the script. You can test to confirm that EC2 instance has been setup correctly by browsing to the IP Address and you should get the following heading.
"
'Hello World, Whats up Yo!!!'
"

I am setting up an IIS server with a powershell script that accepts a string parameter. This parameter value is then used to replace the heading on the default html page. You can test this by using Curl, Postman or any other utility that can POST to the API address passing API key in the header value. The heading parameter value which is set as the heading of the default html page is passed as a JSON string in Body of the POST request.

Following is an example of Curl command I used to update the web page.
curl -d '{"WebPageHeadLine": "Updating it yo again2!!"}' -X POST https://9t2mdq6zyh.execute-api.us-east-1.amazonaws.com/prod/UpdateServer -H 'Content-Type: application/json' -H 'x-api-key: NQKOuasYoe82B2H9xzC2kGCm9bKDSqe5tLarQ5ja' 

## Next Steps
My next update on this will be to allow passing a base64 encoded PowerShell command as part of the web request. This can then be executed as a command on the instance itself. This means we can pretty much do anything on the instance. 

The other change I would like to make is the ability to pass instnace ID or the Name as part of the command which can then be executed on that particular instance.



