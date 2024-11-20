S3 Event Trigger with Lambda and SNS Workflow

Project Overview:

This project automates the handling of S3 bucket events using AWS Lambda and SNS. When a file is uploaded to an S3 bucket, an S3 event triggers a Lambda function. The Lambda function processes the event, logs the file details, and sends a notification through SNS to subscribed users.

Architecture Workflow

S3 Bucket:
Monitors object creation events.
Sends an event notification to the Lambda function.

Lambda Function:
Extracts event details (bucket name and object key).
Logs the file details to AWS CloudWatch.
Publishes a notification to an SNS topic.

SNS Topic:
Receives the message from Lambda.
Forwards the message to all subscribed endpoints (e.g., email).
Steps to Implement

Step 1: Setup IAM Role
Create an IAM Role:
Use the AWS CLI to create a role with permissions for Lambda, S3, and SNS.
Attach the following policies to the role:
AWSLambda_FullAccess
AmazonSNSFullAccess

Save the Role ARN:
Note the ARN for use in Lambda creation.

Step 2: Create S3 Bucket
Use the AWS CLI or Management Console to create an S3 bucket.
Ensure the bucket name is unique globally.
Enable S3 event notifications.

Step 3: Create Lambda Function
Write the Lambda function in Python (refer to the provided script).

Zip the Lambda code:
zip -r s3-lambda-function.zip ./s3-lambda-function

Deploy the Lambda function:
aws lambda create-function \
  --function-name s3-lambda-function \
  --runtime python3.8 \
  --role <role-arn> \
  --handler s3-lambda-function.lambda_handler \
  --zip-file fileb://./s3-lambda-function.zip

Step 4: Add S3 Bucket Permissions
Allow the S3 bucket to invoke the Lambda function:
aws lambda add-permission \
  --function-name s3-lambda-function \
  --statement-id s3-lambda-trigger \
  --action lambda:InvokeFunction \
  --principal s3.amazonaws.com \
  --source-arn arn:aws:s3:::<bucket-name>

Configure the S3 bucket to trigger the Lambda function on object creation:
aws s3api put-bucket-notification-configuration \
  --bucket <bucket-name> \
  --notification-configuration '{
    "LambdaFunctionConfigurations": [{
        "LambdaFunctionArn": "arn:aws:lambda:<region>:<account-id>:function:s3-lambda-function",
        "Events": ["s3:ObjectCreated:*"]
    }]
  }'

Step 5: Create an SNS Topic
Use the AWS CLI to create an SNS topic:
aws sns create-topic --name s3-lambda-sns
Save the Topic ARN for use in the Lambda function.

Step 6: Subscribe to SNS Topic
Add a subscription to the topic:
aws sns subscribe \
  --topic-arn <topic-arn> \
  --protocol email \
  --notification-endpoint <your-email>
Confirm the subscription by clicking the link sent to your email.

Step 7: Test the Workflow
Upload a file to the S3 bucket:
aws s3 cp example_file.txt s3://<bucket-name>/example_file.txt

Verify:
The Lambda function logs in CloudWatch.
The SNS notification is sent to your email.

Conclusion
This project showcases how AWS services like S3, Lambda, and SNS can be integrated to build an event-driven architecture for real-time notifications and processing. It is a practical example for learning and applying serverless concepts with AWS.
