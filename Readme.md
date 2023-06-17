## Deploying an automated infrastructure on AWS for sending notification for every object uploaded on S3.

#### This is a demo project where i delpoyed an automated infrastructre using AWS cloudFormation, which will send notification on email for every object gets uploaded on S3 bucket with the file name, i will be using aws lambda function to excute this function automatically.

#### Resources used in this project:
-   AWS CLI
-   AWS Cloudformation
-   AWS S3
-   AWS SNS
-   AWS Lambda
-   AWS IAM

### Project Architecture:
![Project Diagram](https://github.com/tjdetwill007/Aws-Project-SNS-S3-Lambda_2023/blob/master/project_infra.png)

### Let's start:

### What I did:

-   Log in to your AWS management console.
-   Now we have to generate an access key to get the programatic access in our local computer terminal:
    -  Either you can use your root account to generate the access key `(Not recommend by aws)` or you can create a new iam user and generate the access key for that account.
    -  Go on search tab and search for `iam`.
    -  On left hand side navigation panel click on `users`.
    -  Click on `Add users`, and give the name to user as you desire `projectuser`.
    -  Tick on `Provide user access to the AWS Management Console - optional` if you want to give management console access to the user.
    -  Select `I want to create an IAM user`, `Custom password` and give the password you desire then click `Next`.
    -  Select `Attach policies directly`, and then click on `create policy` it will open a page in new tab.
    - In create policy page click on `json` and then paste the following json permissions and click `next`:
        ```sh
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Sid": "Stmt1686846496236",
                    "Action": [
                        "iam:AttachRolePolicy",
                        "iam:CreatePolicy",
                        "iam:CreateRole",
                        "iam:ListPolicies",
                        "iam:DetachRolePolicy",
                        "iam:DeleteRolePolicy",
                        "iam:DeleteRole"
                    ],
                    "Effect": "Allow",
                    "Resource": "*"
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "s3:*",
                        "s3-object-lambda:*"
                    ],
                    "Resource": "*"
                },
                {
                    "Action": [
                        "sns:*"
                    ],
                    "Effect": "Allow",
                    "Resource": "*"
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "cloudformation:DescribeStacks",
                        "cloudformation:ListStackResources",
                        "cloudformation:CreateUploadBucket",
                        "cloudwatch:ListMetrics",
                        "cloudwatch:GetMetricData",
                        "ec2:DescribeSecurityGroups",
                        "ec2:DescribeSubnets",
                        "ec2:DescribeVpcs",
                        "kms:ListAliases",
                        "iam:GetPolicy",
                        "iam:GetPolicyVersion",
                        "iam:GetRole",
                        "iam:GetRolePolicy",
                        "iam:ListAttachedRolePolicies",
                        "iam:ListRolePolicies",
                        "iam:ListRoles",
                        "lambda:*",
                        "logs:DescribeLogGroups",
                        "states:DescribeStateMachine",
                        "states:ListStateMachines",
                        "tag:GetResources",
                        "xray:GetTraceSummaries",
                        "xray:BatchGetTraces",
                        "cloudformation:*"
                    ],
                    "Resource": "*"
                },
                {
                    "Effect": "Allow",
                    "Action": "iam:PassRole",
                    "Resource": "*",
                    "Condition": {
                        "StringEquals": {
                            "iam:PassedToService": "lambda.amazonaws.com"
                        }
                    }
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "logs:DescribeLogStreams",
                        "logs:GetLogEvents",
                        "logs:FilterLogEvents"
                    ],
                    "Resource": "arn:aws:logs:*:*:log-group:/aws/lambda/*"
                }
            ]
        }
        ```
    -   Give a name to the policy `projectuserpolicy` and then click on `create policy`.
    -   Now go back to the previous tab and in the search tab write the policy name which we created now `projectuserpolicy`, and select the policy, then `Next`. 
    - Click on `create user`.
    -  In `Console sign-in details` click on `Download CSV file` which has the login credentials and then `return to user list`.
    -  In the user list select the user that we created, and then click on `Security Credentials`.
    -  Scroll down and then click on `Create Access Key` thereafter select `Command Line Interface (CLI)`, check the I understand.... and procced further on `Next`. In last Page Click `Create access Key`.
    - Download csv file of your key because you won't be able to retrive it afterwards once you lost it.

   #### Downloading Aws CLI v2 and configure in our local system.
   ##### Note: I am using Linux, if you are using windows please follow aws documentation to install it.

-   Open your terminal and paste following commands:
    ```
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

    unzip awscliv2.zip

    sudo ./aws/install 
    ```
-   Type `aws --version` to check if the aws was successfully installed.
-   Type `aws configure` hit enter.
    -   Paste the access key, and the Secret access key from the csv file that we downloaded.
    -   Give the region code in which you want to deploy your resources. `us-east-1`
    -   Keep default output format to `json`.

#### Creating YAML configurattion file to deploy our resources using CloudFormation.

-   Open any text editor and edit the following YAML code as per your need:

    ```sh
    ---
    AWSTemplateFormatVersion: 2010-09-09
    Description: Create infrastructure for automated notification.



    ##RESOURCES##

    Resources:
    #Creating a role for lambda function
    Lambdarole1:
        Type: AWS::IAM::Role
        Properties: 
        AssumeRolePolicyDocument:
            Version: '2012-10-17'
            Statement:
            - Effect: Allow
            Principal:
                Service: lambda.amazonaws.com
            Action: sts:AssumeRole
        ManagedPolicyArns: 
        - arn:aws:iam::aws:policy/service-role/AWSLambdaRole
        - arn:aws:iam::aws:policy/AmazonSNSFullAccess
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
        - arn:aws:iam::aws:policy/AmazonS3FullAccess
        RoleName: Lambdarole1
    #Creating a SNS topic
    LambdaTopic:
        Type: AWS::SNS::Topic
        Properties: 
        DisplayName: LambdaTopic
        TopicName: lambdatopic
    #Subscribing to the SNS topic with my EMAIL ID.
    MyEmailSubscription:
        Type: AWS::SNS::Subscription
        Properties:
        TopicArn: !Ref LambdaTopic
        Protocol: email
        Endpoint: tjdetwill007@gmail.com
    #Creating the Lambda Function with the python runtime.
    LambdaFunction:
        Type: AWS::Lambda::Function
        Properties: 
        FunctionName: MyLambdaFunction
        Handler: index.lambda_handler
        Runtime: python3.9
        Code:
            ZipFile: |
            import boto3
            import os
            import urllib.parse
            def lambda_handler(event, context):
                print(event)
                bucket_name=event['Records'][0]['s3']['bucket']['name']
                key=event['Records'][0]['s3']['object']['key']
                objectKey=urllib.parse.unquote_plus(key)
                s3 = boto3.client('s3')
                client_sns=boto3.client('sns')
                snsArn = os.environ.get("sns_Arn")



                try:
                    msg="A new File uploaded named {textFileName} in your bucket "+bucket_name
                    file_body = s3.get_object(Bucket=bucket_name, Key=objectKey)
                    file_content=file_body["Body"].read().decode('utf-8')
                    msg=msg.format(textFileName=objectKey)
                    client_sns.publish(
                    TopicArn = snsArn,
                    Message = msg,
                    Subject='Notification for new a new upload on bucket'
                )

                except Exception as e:
                    print(e)
                    print('Error getting object {} from bucket {}. Make sure they exist and your bucket is in the same region as this function.'.format(objectKey, bucket_name))
                    raise e

        Environment:
            Variables:
            sns_Arn: !Ref LambdaTopic
            
        Role: !GetAtt Lambdarole1.Arn
    #Adding permissions for s3 bucket in lambda permission
    S3InvokeLambdaPermission:
        Type: AWS::Lambda::Permission
        Properties:
        FunctionName: !GetAtt LambdaFunction.Arn
        Action: lambda:InvokeFunction      
        Principal: s3.amazonaws.com
        SourceArn: arn:aws:s3:::s3automate123321
    #Creating a bucket with the notification configuration that will trigger the lambda function
    mybucket:
        Type: AWS::S3::Bucket
        Properties:
        BucketName: s3automate123321
        NotificationConfiguration:
            LambdaConfigurations:
            - Event: 's3:ObjectCreated:Put'
                Function: !GetAtt LambdaFunction.Arn  
    #If you add any other resources please be assure about the dependency of each resources with other ressources to  avoid Validation ERROR.     
    #Please edit this yaml file resource name and attributes as per your environment. 
    ```
