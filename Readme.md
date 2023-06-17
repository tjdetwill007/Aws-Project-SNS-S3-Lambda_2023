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
    - In create policy page click on `json` and then pasete the following json permissions:
        '''sh
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
        '''
    -  In `Console sign-in details` click on `Download CSV file` which has the login credentials and then return to user list.
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
