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

![SAM-CodeDeploy](https://user-images.githubusercontent.com/33105405/186556692-515b5670-f0c0-45a2-b4e2-9d0a88d4b099.png)

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

# AWS Cloud Development Kit (CDK)
- Define your cloud infrastructure using a familiar language:
- JavaScript/TypeScript,Python,Java,and.NET
- Contains high level components called `constructs`
- The code is `compiled` into a CloudFormation template (JSON/YAML)
- You can therefore deploy infrastructure and application runtime code together
- Great for Lambda functions and Docker containers in ECS / EKS

## CDK VS SAM

| SAM | CDK |
|--- |--- |
| Serveless focused | All AWS services |
| Write declaratively in JSON or YAML | Write in programming language such as JavaScript/TypeScript, Pyton, etc
| Great for quickly getting started with Lambda | - |
|Leverages CloudFormation | Leverages Cloudformation |

# AWS Step Functions
- AWS Step Functions is a web service that provides `serverless orchestration` for modern applications. It enables you to coordinate the components of distributed applications and microservices using visual workflows.
- Step Functions is based on the `concepts of tasks and state machines`.
  - A task performs work by using an activity or an AWS Lambda function, or by passing parameters to the API actions of other services.
  - A finite state machine can express an algorithm as a number of states, their relationships, and their input and output.
- You define state machines using the JSON-based Amazon States Language.
- A state is referred to by its name, which can be any string, but which must be unique within the scope of the entire state machine. An instance of a state exists until the end of its execution.
  - There are 8 types of states:
    - `Task state` – Do some work in your state machine. AWS Step Functions can invoke Lambda functions directly from a task state.
    - `Choice state` – Make a choice between branches of execution
    - `Fail state` – Stops execution and marks it as failure
    - `Succeed state` – Stops execution and marks it as a success
    - `Pass state` – Simply pass its input to its output or inject some fixed data
    - `Wait state` – Provide a delay for a certain amount of time or until a specified time/date
    - `Parallel state` – Begin parallel branches of execution
    - `Map state` – Adds a for-each loop condition
  - **Common features between states**
    - Each state must have a Type field indicating what type of state it is.
    - Each state can have an optional Comment field to hold a human-readable comment about, or description of, the state.
    - Each state (except a Succeed or Fail state) requires a Next field or, alternatively, can become a terminal state by specifying an End field.
- `Activities enable you to place a task in your state machine where the work is performed by an activity worker that can be hosted on Amazon EC2, Amazon ECS, or mobile devices.`
- Activity tasks let you assign a specific step in your workflow to code running in an activity worker. Service tasks let you connect a step in your workflow to a supported AWS service.
- With Transitions, after executing a state, AWS Step Functions uses the value of the Next field to determine the next state to advance to. States can have multiple incoming transitions from other states.
- **State machine data is represented by JSON text**. 
- It takes the following forms:
  - The initial input into a state machine
  - Data passed between states
  - The output from a state machine
- Individual states receive JSON as input and usually pass JSON as output to the next state.
- **Common Use Cases**
  - Step Functions can help ensure that `long-running, multiple ETL jobs execute in order and complete successfully`, instead of manually orchestrating those jobs or maintaining a separate application.
  - By using Step Functions to handle a few tasks in your codebase, you can approach the transformation of monolithic applications into microservices as a series of small steps.
  - You can use Step Functions to easily `automate recurring tasks such as patch management, infrastructure selection, and data synchronization, and Step Functions will automatically scale, respond to timeouts, and retry failed tasks.`
  - Use Step Functions to `combine multiple AWS Lambda functions into responsive serverless applications` and microservices, without having to write code for workflow logic, parallel processes, error handling, timeouts or retries.
  - You can also orchestrate data and services that run on Amazon EC2 instances, containers, or on-premises servers.

## Error Handling in Step Functions
- Any state can encounter runtime errors for various reasons:
  - State machine definition issues (for example, no matching rule in a Choice state)  
  - Task failures (for example, an exception in a Lambda function)
  - Transient issues (for example, network partition events)
- Use Retry (to retry failed state) and Catch (transition to failure path) in the State Machine to handle the errors instead of inside the Application Code
- Predefined error codes:
  - `States.ALL` : matches any error name
  - `States.Timeout`:Task ran longer thanTimeoutSeconds or no heartbeat received - States.TaskFailed: execution failure
  - `States.Permissions`: insufficient privileges to execute code
- The state may report is own errors

## Step Functions – Retry (Task or Parallel State)
- Evaluated from top to bottom
- `ErrorEquals`: match a specific kind of error
- `IntervalSeconds`: initial delay before retrying
- `BackoffRate`: multiple the delay after each retry
- `MaxAttempts`: default to 3, set to 0 for never retried
- When max attempts are reached, the Catch kicks in

```json
"Retry": [ 
{
      "ErrorEquals": [ "ErrorA", "ErrorB" ],
      "IntervalSeconds": 1,
      "BackoffRate": 2.0,
      "MaxAttempts": 2
}
{
   "ErrorEquals": [ "States.Timeout" ],
   "IntervalSeconds": 3,
   "MaxAttempts": 2,
   "BackoffRate": 1.5
} 
{
   "ErrorEquals": [ "States.All" ],
   "IntervalSeconds": 5,
   "MaxAttempts": 5,
   "BackoffRate": 2.0
} 

]
```
## Step Functions – Catch (Task or Parallel State)
Evaluated from top to bottom
- `ErrorEquals`: match a specific kind of error
- `Next`: State to send to
- `ResultPath` - A path that determines what input is sent to the state specified in the Next field.
  
```json
"Catch": [ 
{
   "ErrorEquals": [ "java.lang.Exception" ],
   "ResultPath": "$.error-info",
   "Next": "RecoveryState"
}, 
{
   "ErrorEquals": [ "States.ALL" ],
   "Next": "EndState"
} 
]
```
## Step Function - Standard vs Express

|  	| Standard Workflows 	| Express Workflows 	|
|---	|---	|---	|
| Maximum duration 	| 1 year. 	| 5 minutes. 	|
| Supported execution start rate 	| Over 2,000 per second 	| Over 100,000 per second 	|
| Supported state transition rate 	| Over 4,000 per second per account 	| Nearly Unlimited 	|
| Pricing 	| Priced per state transition. A state transition is counted each time a step in your execution is completed (more expensive) 	| Priced by the number of executions you run, their duration, and memory consumption (cheaper) 	|
| Execution history 	| Executions can be listed and described with Step Functions APIs, and visually debugged through<br>the console. They can also be inspected in CloudWatch Logs by enabling logging on your state machine. 	| Executions can be inspected in CloudWatch Logs by enabling logging on your state machine. 	|
| Execution semantics 	| Exactly-once workflow execution. 	| At-least-once workflow execution. 	|

# App Sync
- AppSync is a managed service that uses GraphQL
- GraphQL makes it easy for applications to get exactly the data they need.
- This includes combining data from one or more sources 
  - NoSQL data stores, Relational databases, HTTP APIs...
  - Integrates with DynamoDB, Aurora, Elasticsearch & others  
  - Custom sources with AWS Lambda
- Retrieve data in real-time with WebSocket or MQTT on WebSocket 
- For mobile apps: local data access & data synchronization
- It all starts with uploading one GraphQL schema

![GraphQL-AppSync](https://user-images.githubusercontent.com/33105405/186572140-0b566582-e879-4a99-813c-cbd7496fd377.png)

## AppSync – Security
- There are four ways you can authorize applications to interact with your AWS AppSync GraphQL API:
  - API_KEY
  - AWS_IAM: IAM users / roles / cross-account access
  - OPENID_CONNECT: OpenID Connect provider / JSON Web Token
  - AMAZON_COGNITO_USER_POOLS
- For custom domain & HTTPS, use CloudFront in front of AppSync

# AWS Amplify
- Set of tools to get started with creating mobile and web applications
- “Elastic Beanstalk for mobile and web applications”
- Must-have features such as data storage, authentication, storage, and machine-learning, all powered by AWS services
- Front-end libraries with ready-to-use components for React.js,Vue, Javascript, iOS, Android, Flutter, etc...
- Incorporates AWS best practices to for reliability, security, scalability
- Build and deploy with the Amplify CLI or Amplify Studio

## AWS Amplify – Important Features
### Authentication
  - `amplify add auth`
  - Leverages Amazon Cognito
  - User registration, authentication, account recovery & other operations
  - Support MFA, Social Sign-in, etc...
  - Pre-built UI components
  - Fine-grained authorization
### Datastore
  - `amplify add api`
  - Leverages Amazon AppSync and Amazon DynamoDB
  - Work with local data and have automatic synchronization to the cloud without complex code
  - Powered by GraphQL
  - Offline and real-time capabilities
  - Visual data modeling w/ Amplify Studio
### AWS Amplify Hosting
  - `amplify add hosting`
  - Build and Host Modern Web Apps 
  - CICD (build, test, deploy)
  - Pull Request Previews
  - Custom Domains
  - Monitoring
  - Redirect and Custom Headers - Password protection
