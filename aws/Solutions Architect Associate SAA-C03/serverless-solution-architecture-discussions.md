# Serverless Solutions Architecture Discussions

## Mobile Application: MyTodoList

- Requirements
  - expose as REST API with HTTPS
  - serverless architecture
  - users should be able to directly interact with their own S3 folder
  - users should authenticate through a managed serverless service
  - users can write and read to-dos, but they mostly read them
  - database should scale, have some high read throughput
- Solution
  - serverless REST API: HTTPS, API Gateway, Lambda, DynamoDB
  - using Cognito to generate temporary credentials with STS to access S3 bucket with restricted policy App uses can directly access AWS resourcses this way
    - pattern can be applied to DynamoDB, Lambda, ...
  - cache reads on DynamoDB using DAX
  - cashe the REST requests at the API Gateway Level
  - security for authentication and authorization with Cognito, STS

## Serverless Website: MyBlog.com

- Requirements
  - scale globally
  - blogs are rarely written, but often read
  - some of the website is purely static files, the rest is a dynamic REST API
  - caching must be implemented where possible
  - any new users that subscribe should recieve a welcome email
  - any photo upload to the blog should have a thumbnail generated
- Solution
  - static site distributed with CloudFront and S3
  - REST API was serverless, didn't need Cognito because public
  - leverage Global DynamoDB table to sever the data globally
    - could have used Aurora Global Database
  - enabled DynamoDB steams to trigger a Lambda function
  - lambda function had IAM role which could use SES
  - SES (Simple Email Service) was used to send emails in a serverless way
  - S3 can trigger SQS / SNS / Lambda to notify of events

## Microservices Architecture

- Requirements
  - use a mircroservice architecture
  - many services interact with each other directly using a REST API
  - each architecture for each microsevice may vary in form and shape
  - we want a leaner development cycle for each service
- free to design each microservice the way you want
- Synchronous Patterns: API Gateway, Load Balancers
- Asynchronous patterns: SQS, Kinesis, SNS, Lambda Triggers (S3)
- Challenges
  - repeated overhead for creating each new microservice
  - issues with optimizing server density/utilization
  - complexity of running multiple versions of multiple microservices simultaneously
  - proliferation of client-side code requirements to integrate with many seperate services
- Some challenges are solved by serverless patterns
  - API Gateway, Lambda scale automatically and you pay per usage
  - easy clone APi, reproduce environments
  - generated client SDK through swagger integration for the API Gateway

## Software update distribution

- Requirments
  - application on EC2 that distributes software updates once in a while
  - new software update is out we get a lot of requests and the content is distributed in mass over network (very costly)
  - we dont want to change our application, but want to optimize out cost and CPU
- Solution
  - CloudFront in front of EC2
    - no changes to underlying architecture
    - will cache software update files at the edge
    - software update files are not dynamic, they are static (never changing)
    - EC2 isntances are not serverless, but CloudFront is and will scale for us
    - ASG will not scale as much, save a lot in EC2
    - save in availability, network bandwidth cost, etc
    - easy way to make an exisiting application more scalable and cheaper
