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

![Cog-Identity-with-CUP](https://user-images.githubusercontent.com/33105405/186547671-85f31c95-9e65-4ac0-898d-2e8272ebf451.png)

![Cog-Identity-Pool](https://user-images.githubusercontent.com/33105405/186547683-58662dd6-b341-417c-8055-9ddf43362f71.png)


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
