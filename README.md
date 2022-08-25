# Amazon Cognito
## Intro
We want to give our users an identity so that they can interact with our application.
- Cognito User Pools (CUP)
  - Sign in functionality for app users
  - Integrate with API Gateway & Application Load Balancer
- Cognito Identity Pools (Federated Identity):
  - Provide AWS credentials to users so they can access AWS resources directly 
  - Integrate with Cognito User Pools as an identity provider
- Cognito Sync:
  - Synchronize data from device to Cognito. 
  - Is deprecated and replaced by AppSync
 - Cognito vs IAM: `hundreds of users`, `mobile users`, `authenticate with SAML`
## Cognito User Pools (CUP) – User Features
- Create a serverless database of user for your web & mobile apps - Simple login: Username (or email) / password combination
- Password reset
- Email & Phone Number Verification
- Multi-factor authentication (MFA)
- Federated Identities: users from Facebook, Google, SAML...
- Feature: block users if their credentials are compromised elsewhere
- Login sends back a JSON Web Token (JWT)

![cup](https://user-images.githubusercontent.com/33105405/186533254-db245014-a78d-4dd6-b81d-ba984dd18dc7.png)

![aws-cognito-cup](https://user-images.githubusercontent.com/33105405/186533237-a7879df5-5904-4a90-8265-389e3f02d74a.png)

### Cognito User Pool Lambda Triggers

| User Pool Flow | Operation | Description |
|--- |--- |--- |
| Authenication Events | Pre Authentication Lambda Trigger | Custom validation to accept or deny the sign-in request |
| | Post Authentication Lambda Trigger | Event logging for custom analytics |
| | Pre Token Generation Lambda Trigger | Augment or suppress token claims |
| Sign-Up | Pre Sign-ip Lambda Trigger | Custom Validation to accept or deny the sign-up request |
| | Post Conformation Lambda Trigger | Custom welcome messages or event logging for custom analytics |
| | Migrate USer Lambda TRigger | Migrate a user from an existing user directory to user pools |
| Messages | Custom message lambda trigger | Advanced customisation and localization of messages |
| Token Creation | Pre Token Generation Lambda TRigger | Add or Remove attribtutes in ID tokens |


## Cognito User Pools – Hosted Authentication UI
- Cognito has a hosted authentication UI that you can add to your app to handle sign- up and sign-in workflows
- Using the hosted UI, you have a foundation for integration with social logins, OIDC or SAML
- Can customize with a custom logo and custom CSS

## Cognito Identity Pools (Federated Identities)
- Get identities for `users` so they obtain temporary AWS credentials
- Your identity pool (e.g identity source) can include:
  - Public Providers (Login with Amazon, Facebook, Google, Apple)
  - Users in an Amazon Cognito user pool
  - OpenID Connect Providers & SAML Identity Providers
  - Developer Authenticated Identities (custom login server)
  - Cognito Identity Pools allow for `unauthenticated (guest) access`
- Users can then access AWS services directly or through API Gateway
  - The IAM policies applied to the credentials are defined in Cognito
  - They can be customized based on the user_id for fine grained control

![Cog-Identity-Pool](https://user-images.githubusercontent.com/33105405/186548097-17b7d5fe-0164-431b-b9a4-e32aafc5113e.png)

![Cog-Identity-with-CUP](https://user-images.githubusercontent.com/33105405/186548108-5143ac9b-34f6-49ee-9154-6179917e10da.png)

## Cognito Identity Pools – IAM Roles
- Default IAM roles for authenticated and guest users
- Define rules to choose the role for each user based on the user’s ID 
- You can partition your users’ access using ***policy variables***
- IAM credentials are obtained by Cognito Identity Pools through STS 
- The roles must have a `trust` policy of Cognito Identity Pools

## Cognito User Pools vs Identity Pools
- Cognito User Pools:
  - Database of users for your web and mobile application
  - Allows to federate logins through Public Social, OIDC, SAML...
  - Can customize the hosted UI for authentication (including the logo)] 
  - Has triggers with AWS Lambda during the authentication flow
- Cognito Identity Pools:
  - Obtain AWS credentials for your users
  - Users can login through Public Social, OIDC, SAML & Cognito User Pools 
  - Users can be unauthenticated (guests)
  - Users are mapped to IAM roles & policies, can leverage policy variables
- CUP + CIP = manage user / password + access AWS services

## Cognito Sync
- Deprecated – use AWS AppSync now
- Store preferences, configuration, state of app
- Cross device synchronization (any platform – iOS, Android, etc...)
- Offline capability (synchronization when back online)
- Store data in datasets (up to 1MB), up to 20 datasets to synchronize 
- **Push Sync:** silently notify across all devices when identity data changes 
- **Cognito Stream:** stream data from Cognito into Kinesis
- **Cognito Events:** execute Lambda functions in response to events


# Serverless Application Model
## Intro
- Framework for developing and deploying serverless applications
- All the configuration is YAML code
- Generate complex CloudFormation from simple SAMYAML file
- Supports anything from CloudFormation: Outputs, Mappings, Parameters, Resources...
- Only two commands to deploy to AWS
- SAM can use CodeDeploy to deploy Lambda functions
- SAM can help you to run Lambda, API Gateway, DynamoDB locally

### SAM Recipe

Transform Header indicates it’s SAM template:
- Transform: 'AWS::Serverless-2016-10-31'
- Write Code
  
    `AWS::Serverless::Function`

    `AWS::Serverless::Api`

    `AWS::Serverless::SimpleTable`
- Package & Deploy:
  - aws cloudformation package / sam package
  - aws cloudformation deploy / sam deploy

![SAM-Deployment](https://user-images.githubusercontent.com/33105405/186555178-db5ab9ca-14ed-4629-bb2a-630555660b0e.png)

###  SAM - Cli Debugging
SAM – CLI Debugging
- Locally build, test, and debug your serverless applications that are defined using AWS SAM templates
- Provides a lambda-like execution environment locally
- SAM CLI + AWS Toolkits => step-through and debug your code
- Supported IDEs:AWS Cloud9,Visual Studio Code, JetBrains, PyCharm, IntelliJ, ...
- AWS Toolkits: IDE plugins which allows you to build, test, debug, deploy, and invoke Lambda functions built using AWS SAM

### SAM - Policy Templates

- List of templates to apply permissions to your Lambda Functions
- Full list available here: https://docs.aws.amazon.com/serverless- application- model/latest/developerguide/serverless- policy-templates.html#serverless-policy- template-table
- Important examples:
  - `S3ReadPolicy`: Gives read only permissions to objects in S3
  - `SQSPollerPolicy`: Allows to poll an SQS queue 
  - `DynamoDBCrudPolicy`: CRUD = create read update delete

### SAM and Code Deploy

- SAM framework natively uses CodeDeploy to update Lambda functions
- Traffic Shifting feature
- Pre and Post traffic hooks features to validate deployment (before the traffic shift starts and after it ends)
- Easy & automated rollback using CloudWatch Alarms

![SAM-CodeDeploy](https://user-images.githubusercontent.com/33105405/186555109-af923e49-40e5-4590-b49e-ec04efc37639.png)

## SAM Summary

SAM is built on CloudFormation
- SAM requires the Transform and Resources sections
- Commands to know:
  - sam build: fetch dependencies and create local deployment artifacts
  - sam package: package and upload to Amazon S3, generate CF template 
  - sam deploy: deploy to CloudFormation
- SAM Policy templates for easy IAM policy definition
- SAM is integrated with CodeDeploy to do deploy to Lambda aliases

# Serverless Application RRepository (SAR)
Managed repository for serverless applications
- The applications are packaged using SAM
- Build and publish applications that can be re-used by organizations
  - Can share publicly
  - Can share with specific AWS accounts
- This prevents duplicate work, and just go straight to publishing
- Application settings and behaviour can be customized using Environment variables

