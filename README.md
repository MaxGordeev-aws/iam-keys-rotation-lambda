# Automatic IAM User Access Key Rotation

# AWS Services Used

    - SNS Topic
    - Secrets Manager
    - EventBridge (Formerly known as CloudWatch Events)
    - AWS Lambda
    - Identity Access Management

# Workflow

It’s important for us to get a clear picture of what we’re trying to achieve and how all of these AWS services are connected to one another. Here’s how it works:

    - An IAM User is created with Access Key generated. 

    - 3 Event Bridge rules are used to trigger a lambda function based on number of days elapsed.
        - Every 90 days – Create new access keys.
        - Every 104 days – Deactivate the old access key.
        - Every 128 days – Delete the old access key.

    - Lambda function receives 1) IAM Username 2) The action to perform and performs the 3 actions above accordingly on IAM User.

    - Secrets Manager stores the new access key and holds records of previous access keys.

# Steps

    1. Create the IAM user and generate AWS Access Key.
       - For all IAM user and roles created, skip attaching or creating IAM policies. This will be revisited once we have created all necessary resources.
    
    2. Create IAM Role for our Lambda function.
       - When creating this role, set the trusted entity type to be AWS service and the usecase to be “Lambda”.
    
    3. Create Secrets Manager Secret with the secret name matching the name of the IAM username you intended.
       - Select “Other type of secrets” option as secret type. 
       - For secret key names, use “AccessKey” for IAM Access Key and “SecretKey” for IAM secret access key.
       - Keep key rotation disabled.
    
    4. Create Lambda Function that will process IAM key rotation requests.
       - Use the code above for the lambda function. 
       - Be sure to modify parts of the code that have been surrounded by '<>'. 

    5. Create Event bridge rule to trigger creating access key.

        - Select “default” event bus
        - Define the pattern to use “Schedule”
            - Set Fixed Rate to every 90 days
        - Set the target as “Lambda function”
            - Set the Function to the lambda function name in step 4
            - Set “Configure input” setting to “Constant (JSON text)”
                - Set value to: { "action": "create", "username": "<IAM_USER>"}
    
    6. Create Event bridge rule to trigger deactivating access key.

        - Select “default” event busDefine the pattern to use “Schedule”
        - Set Fixed Rate to every 104 daysSet the target as “Lambda function”
        - Set the Function to the lambda function name in step 4
            - Set “Configure input” setting to “Constant (JSON text)”
                - Set value to: { "action": "deactivate", "username": "<IAM_USER>"}

    7. Create Event bridge rule to trigger deleting deactivated access key.

        - Select “default” event busDefine the pattern to use “Schedule”
        - Set Fixed Rate to every 118 daysSet the target as “Lambda function”
        - Set the Function to the lambda function name in step 4
        - Set the Function to the lambda function name in step 4
            - Set “Configure input” setting to “Constant (JSON text)”
                - Set value to: { "action": "delete", "username": "<IAM_USER>"}

    8. Create IAM Policy to enabled our lambda function to 
        - Access Secrets Manager Secret
        - Access IAM service to manage user access key. 

     ```
      {
    "Version":"2012-10-17",
    "Statement": [
      {
        "Effect": "Allow"
        "Action": [
          "secretsmanager:GetSecretValue",
          "secretsmanager:PutSecretValue"
        ],
        "Resource": "<secrets manager secret arn>"
      },
      {
        "Effect": "Allow"
        "Action": [
          "iam:UpdateAccessKey",
          "iam:CreateAccessKey",
          "iam:DeleteAccessKey",
        ],
        "Resource": "<secrets manager secret arn>"
        "Principal": {
          "AWS": "<iam role arn>"
        }         
      },
      {
        "Effect": "Allow"
        "Action": "iam:ListAccessKeys",
        "Resource": "*"
      },
      {
        "Effect": "Allow"
        "Action": "sns:Publish",
        "Resource": "<sns topic arn>"
      }
    ]
}
     ```   


