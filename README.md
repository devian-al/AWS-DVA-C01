# Lambda
## Benefits
- Easy Pricing:
  - Pay per request and compute time
  - Free tier of 1,000,000 AWS Lambda requests and 400,000 GBs of compute time
- Integrated with the whole AWS suite of services
- Integrated with many programming languages
- Easy monitoring through AWS CloudWatch
- Easy to get more resources per functions (up to 10GB of RAM!) - Increasing RAM will also improve CPU and network!

## AWS Lambda Pricing: example
- You can find overall pricing information here: https://aws.amazon.com/lambda/pricing/
- Pay per calls:
    - First 1,000,000 requests are free
    - $0.20 per 1 million requests thereafter ($0.0000002 per request)
- Pay per duration: (in increment of 1 ms)
    - 400,000 GB-seconds of compute time per month for FREE 
    - == 400,000 seconds if function is 1GB RAM
    - == 3,200,000 seconds if function is 128 MB RAM
    - After that $1.00 for 600,000 GB-seconds
- It is usually very cheap to run AWS Lambda so it’s very popular

## Lambda – Synchronous Invocations
- Synchronous: CLI, SDK, API Gateway, Application Load Balancer
  - Results is returned right away
  - Error handling must happen client side (retries, exponential backoff, etc...)

## Lambda - Synchronous Invocations - Services
- User Invoked:
  - Elastic Load Balancing (Application Load Balancer) 
  - Amazon API Gateway
  - Amazon CloudFront (Lambda@Edge)
  - Amazon S3 Batch
- Service Invoked:
  - Amazon Cognito
  - AWS Step Functions
- Other Services:
  - Amazon Lex
  - Amazon Alexa
  - Amazon Kinesis Data Firehose

## Lambda Integration with ALB
- To expose a Lambda function as an HTTP(S) endpoint...
- You can use the Application Load Balancer (or an API Gateway) 
- The Lambda function must be registered in a target group

## ALB Multi-Header Values
- ALB can support multi header values (ALB setting)
- When you enable multi-value headers, HTTP headers and query string parameters that are sent with multiple values are shown as arrays within the AWS Lambda event and response objects.

## Lambda@Edge
- You have deployed a CDN using CloudFront
- What if you wanted to run a global AWS Lambda alongside?
- Or how to implement request filtering before reaching your application?
- For this, you can use Lambda@Edge:
deploy Lambda functions alongside your CloudFront CDN
  - Build more responsive applications
  - You don’t manage servers, Lambda is deployed globally 
  - Customize the CDN content
  - Pay only for what you use
- You can use Lambda to change CloudFront requests and responses:
  - After CloudFront receives a request from a viewer (viewer request)
  - Before CloudFront forwards the request to the origin (origin request)
  - After CloudFront receives the response from the origin (origin response)
  - Before CloudFront forwards the response to the viewer (viewer response)
- You can also generate responses to viewers without ever sending the request to the origin

### Use Cases
- Website Security and Privacy
- Dynamic Web Application at the Edge
- Search Engine Optimization (SEO)
- Intelligently Route Across Origins and Data Centers 
- Bot Mitigation at the Edge
- Real-time Image Transformation
- A/BTesting
- User Authentication and Authorization
- User Prioritization
- User Tracking and Analytics

## Lambda – Asynchronous Invocations
- S3, SNS, CloudWatch Events...
- The events are placed in an Event Queue
- Lambda attempts to retry on errors
  - 3 tries total
  - 1 minute wait after 1st , then 2 minutes wait
- Make sure the processing is idempotent (in case of retries)
- If the function is retried, you will see duplicate logs entries in CloudWatch Logs
- Can define a DLQ (dead-letter queue) – SNS or SQS – for failed processing (need correct IAM permissions)
- Asynchronous invocations allow you to speed up the processing if you don’t need to wait for the result (ex: you need 1000 files processed)

## S3 Events Notifications
- S3:ObjectCreated, S3:ObjectRemoved, S3:ObjectRestore, S3:Replication...
- Object name filtering possible (*.jpg)
- Use case: generate thumbnails of images uploaded to S3
- S3 event notifications typically deliver events in seconds but can sometimes take a minute or longer
- If two writes are made to a single non- versioned object at the same time, it is possible that only a single event notification will be sent
- If you want to ensure that an event notification is sent for every successful write, you can enable versioning on your bucket.

## Lambda – Event Source Mapping
NOT FOR DISTRIBUTION © Stephane Maarek www.datacumulus.com
- Kinesis Data Streams
- SQS & SQS FIFO queue
- DynamoDB Streams
- Common denominator: records need to be polled from the source
- Your Lambda function is invoked synchronously

## Streams & Lambda (Kinesis & DynamoDB)
- An event source mapping creates an iterator for each shard, processes items in order 
- Start with new items, from the beginning or from timestamp
- Processed items aren't removed from the stream (other consumers can read them) 
- Low traffic: use batch window to accumulate records before processing
- You can process multiple batches in parallel
  - up to 10 batches per shard
  - in-order processing is still guaranteed for each partition key

## Streams & Lambda – Error Handling
- By default, if your function returns an error, the entire batch is reprocessed until the function succeeds, or the items in the batch expire.
- To ensure in-order processing, processing for the affected shard is paused until the error is resolved
- You can configure the event source mapping to:
  - discard old events
  - restrict the number of retries
  - split the batch on error (to work around Lambda timeout issues)
- Discarded events can go to a Destination

## Lambda – Event Source Mapping SQS & SQS FIFO
- Event Source Mapping will poll SQS (Long Polling)
- Specify batch size (1-10 messages)
- Recommended: Set the queue visibility timeout to 6x the timeout of your Lambda function
- To use a DLQ
    - set-up on the SQS queue, not Lambda (DLQ for Lambda is only for async invocations)
    - Or use a Lambda destination for failures

## Queues & Lambda
- Lambda also supports in-order processing for FIFO (first-in, first-out) queues, scaling up to the number of active message groups.
- For standard queues, items aren't necessarily processed in order.
- Lambda scales up to process a standard queue as quickly as possible.
- When an error occurs, batches are returned to the queue as individual items and might be processed in a different grouping than the original batch.
- Occasionally, the event source mapping might receive the same item from the queue twice, even if no function error occurred.
- Lambda deletes items from the queue after they're processed successfully.
- You can configure the source queue to send items to a dead-letter queue if they can't be processed.

## Lambda Event Mapper Scaling
- Kinesis Data Streams & DynamoDB Streams:
  - One Lambda invocation per stream shard
  - If you use parallelization, up to 10 batches processed per shard simultaneously
- SQS Standard:
  - Lambda adds 60 more instances per minute to scale up
  - Up to 1000 batches of messages processed simultaneously
- SQS FIFO:
  - Messages with the same GroupID will be processed in order
  - The Lambda function scales to the number of active message groups

## Lambda – Destinations
- Nov 2019: Can configure to send result to a destination
- Asynchronous invocations - can define destinations for successful and failed event:
  - Amazon SQS
  - Amazon SNS
  - AWS Lambda
  - Amazon EventBridge bus
- Note: AWS recommends you use destinations instead of DLQ now (but both can be used at the same time)
- Event Source mapping: for discarded event batches
  - Amazon SQS
  - Amazon SNS
- Note: you can send events to a DLQ directly from SQS

## Lambda Execution Role (IAM Role)
- Grants the Lambda function permissions to AWS services / resources
- Sample managed policies for Lambda:
  - AWSLambdaBasicExecutionRole – Upload logs to CloudWatch.
  - AWSLambdaKinesisExecutionRole – Read from Kinesis
  - AWSLambdaDynamoDBExecutionRole – Read from DynamoDB Streams 
  - AWSLambdaSQSQueueExecutionRole – Read from SQS
  - AWSLambdaVPCAccessExecutionRole – Deploy Lambda function in VPC
  - AWSXRayDaemonWriteAccess – Upload trace data to X-Ray.
- When you use an event source mapping to invoke your function, Lambda uses the execution role to read event data.
- Best practice: create one Lambda Execution Role per function

## Lambda Resource Based Policies
- Use resource-based policies to give other accounts and AWS services permission to use your Lambda resources
- Similar to S3 bucket policies for S3 bucket
- An IAM principal can access Lambda:
  - if the IAM policy attached to the principal authorizes it (e.g. user access) 
  - OR if the resource-based policy authorizes (e.g. service access)
- When an AWS service like Amazon S3 calls your Lambda function, the resource-based policy gives it access.

## Lambda Environment Variables
- Environment variable = key / value pair in “String” form
- Adjust the function behavior without updating code
- The environment variables are available to your code
- Lambda Service adds its own system environment variables as well
- Helpful to store secrets (encrypted by KMS)
- Secrets can be encrypted by the Lambda service key, or your own CMK

## Lambda Logging & Monitoring
- CloudWatch Logs:
    - AWS Lambda execution logs are stored in AWS CloudWatch Logs
    - Make sure your AWS Lambda function has an execution role with an IAM policy that authorizes writes to CloudWatch Logs
- CloudWatch Metrics:
    - AWS Lambda metrics are displayed in AWS CloudWatch Metrics
    - Invocations, Durations, Concurrent Executions
    - Error count, Success Rates,Throttles
    - Async Delivery Failures
    - Iterator Age (Kinesis & DynamoDB Streams)

## Lambda Tracing with X-Ray
- Enable in Lambda configuration (Active Tracing)
- Runs the X-Ray daemon for you
- Use AWS X-Ray SDK in Code
- Ensure Lambda Function has a correct IAM Execution Role 
    - The managed policy is called AWSXRayDaemonWriteAccess
- Environment variables to communicate with X-Ray
    - _X_AMZN_TRACE_ID: contains the tracing header
    - AWS_XRAY_CONTEXT_MISSING: by default, LOG_ERROR
    - AWS_XRAY_DAEMON_ADDRESS: the X-Ray Daemon IP_ADDRESS:PORT

## Lambda by default
- By default, your Lambda function is launched outside your own VPC (in an AWS-owned VPC)
- Therefore it cannot access resources in your VPC (RDS, ElastiCache, internal ELB...)

## Lambda in VPC
- You must define the VPC ID, the Subnets and the Security Groups
- Lambda will create an ENI (Elastic Network Interface) in your subnets
- AWSLambdaVPCAccessExecutionRole

### Internet Access
- A Lambda function in your VPC does not have internet access
- Deploying a Lambda function in a public subnet does not give it internet access or a public IP
- Deploying a Lambda function in a private subnet gives it internet access if you have a NAT Gateway / Instance
- You can use VPC endpoints to privately access AWS services without a NAT

## Lambda Function Configuration
- RAM:
    - From 128MB to 10GB in 1MB increments
    - The more RAM you add, the more vCPU credits you get
    - At 1,792 MB, a function has the equivalent of one full vCPU
    - After 1,792 MB, you get more than one CPU, and need to use multi-threading in your code to benefit from it (up to 6 vCPU)
- If your application is CPU-bound (computation heavy), increase RAM 
- Timeout: default 3 seconds, maximum is 900 seconds (15 minutes)

## Lambda Execution Context
- The execution context is a temporary runtime environment that initializes any external dependencies of your lambda code
- Great for database connections, HTTP clients, SDK clients...
- The execution context is maintained for some time in anticipation of another Lambda function invocation
- The next function invocation can “re-use” the context to execution time and save time in initializing connections objects
- The execution context includes the /tmp directory
- Intialise outside the handler

**BAD**
```python
# The DB connection is established
# at every function invocation

import os
 def get_usr_handler(event, context):
    DB_URL = os.getenv("DB_URL")
    db_client = db.connect(DB_URL)
    user = db_client.get(user_id = event["user_id"])
    return user
```
**Good**
```python
# The DB connection is established once
# and re-used across invocations

import os
DB_URL = os.getenv("DB_URL")
db_client = db.connect(DB_URL)

 def get_usr_handler(event, context):
    user = db_client.get(user_id = event["user_id"])
    return user
```
## Lambda Functions /tmp space
- If your Lambda function needs to download a big file to work...
- If your Lambda function needs disk space to perform operations...
- You can use the /tmp directory
- Max size is 512MB
- The directory content remains when the execution context is frozen, providing transient cache that can be used for multiple invocations (helpful to checkpoint your work)
- For permanent persistence of object (non temporary), use S3

## Lambda Concurrency and Throttling 
- Concurrency limit: up to 1000 concurrent executions
- Can set a “reserved concurrency” at the function level (=limit)
- Each invocation over the concurrency limit will trigger a “Throttle”
- Throttle behavior:
  - If synchronous invocation => return ThrottleError - 429
  - If asynchronous invocation => retry automatically and then go to DLQ
- If you need a higher limit, open a support ticket

## Concurrency and Asynchronous Invocations
- If the function doesn't have enough concurrency available to process all events, additional requests are throttled.
- For throttling errors (429) and system errors (500-series), Lambda returns the event to the queue and attempts to run the function again for up to 6 hours.
- The retry interval increases exponentially from 1 second after the first attempt to a maximum of 5 minutes.

## Cold Starts & Provisioned Concurrency
- Cold Start:
  - New instance => code is loaded and code outside the handler run (init)
  - If the init is large (code, dependencies, SDK...) this process can take some time. 
  - First request served by new instances has higher latency than the rest
- Provisioned Concurrency:
  - Concurrency is allocated before the function is invoked (in advance)
  - So the cold start never happens and all invocations have low latency
  - Application Auto Scaling can manage concurrency (schedule or target utilization)
- Note:
  - Cold starts inVPC have been dramatically reduced in Oct & Nov 2019
  - https://aws.amazon.com/blogs/compute/announcing-improved-vpc-networking-for-aws-lambda-functions/

## Lambda Function Dependencies
- If your Lambda function depends on external libraries: for example AWS X-Ray SDK, Database Clients, etc...
- You need to install the packages alongside your code and zip it together 
  - For Node.js, use npm & “node_modules” directory
  - For Python, use pip --target options 
  - For Java, include the relevant .jar files
- Upload the zip straight to Lambda if less than 50MB, else to S3 first
- Native libraries work: they need to be compiled on Amazon Linux
- AWS SDK comes by default with every Lambda function

## Lambda and CloudFormation – inline
- Inline functions are very simple
- Use the Code.ZipFile proper ty
- You cannot include function dependencies with inline functions
### Through S3
- You must store the Lambda zip in S3
- You must refer the S3 zip location in the CloudFormation code
    - S3Bucket
    - S3Key: full path to zip
    - S3ObjectVersion: if versioned bucket
- If you update the code in S3, but don’t update S3Bucket, S3Key or S3ObjectVersion, CloudFormation won’t update your function

## Lambda Layers
- Custom Runtimes
- Externalize dependencies to re-use them

## Lambda Container Images
- Deploy Lambda function as container images of up to 10GB from ECR
- Pack complex dependencies, large dependencies in a container
- Base images are available for Python, Node.js, Java, .NET, Go, Ruby
- Can create your own image as long as it implements the Lambda Runtime API
- Test the containers locally using the Lambda Runtime Interface Emulator
- Unified workflow to build apps

## AWS Lambda Versions
- When you work on a Lambda function, we work on $LATEST
- When we’re ready to publish a Lambda function, we create a version
- Versions are immutable
- Versions have increasing version numbers
- Versions get their own ARN (Amazon Resource Name)
- Version = code + configuration (nothing can be changed - immutable)
- Each version of the lambda function can be accessed

## AWS Lambda Aliases
- Aliases are ”pointers” to Lambda function versions
- We can define a “dev”, ”test”, “prod” aliases and have them point at different lambda versions
- Aliases are mutable
- Aliases enable Blue / Green deployment by assigning weights to lambda functions
- Aliases enable stable configuration of our event triggers / destinations
- Aliases have their own ARNs
- Aliases cannot reference aliases

## Lambda & CodeDeploy
- CodeDeploy can help you automate traffic shift for Lambda aliases
- Feature is integrated within the SAM framework
- Linear: grow traffic every N minutes until 100%
  - Linear10PercentEvery3Minutes
  - Linear10PercentEvery10Minutes
- Canary: try X percent then 100% 
  - Canary10Percent5Minutes
  - Canary10Percent30Minutes
- AllAtOnce: immediate
- Can create Pre & Post Traffic hooks to check the health of the Lambda function

## AWS Lambda Limits to Know - per region
- Execution:
    - Memory allocation: 128 MB – 10GB (1 MB increments)
    - Maximum execution time: 900 seconds (15 minutes)
    - Environment variables (4 KB)
    - Disk capacity in the “function container” (in /tmp): 512 MB 
    - Concurrency executions: 1000 (can be increased)
- Deployment:
    - Lambda function deployment size (compressed .zip): 50 MB
    - Size of uncompressed deployment (code + dependencies): 250 MB
    - Can use the /tmp directory to load other files at startup
    - Size of environment variables: 4 KB

## AWS Lambda Best Practices
- **Perform heavy-duty work outside of your function handler **
  - Connect to databases outside of your function handler
  - Initialize the AWS SDK outside of your function handler
  - Pull in dependencies or datasets outside of your function handler
- **Use environment variables for:**
  - Database Connection Strings, S3 bucket, etc... don’t put these values in your code 
  - Passwords, sensitive values... they can be encrypted using KMS
- **Minimize your deployment package size to its runtime necessities.** 
  - Break down the function if need be
  - Remember the AWS Lambda limits
  - Use Layers where necessary
- **Avoid using recursive code, never have a Lambda function call itself**

# DynamoDB
## Traditional Architecture 
-  Traditional applications leverage RDBMS databases
-  These databases have the SQL query language
-  Strong requirements about how the data should be modeled
-  Ability to do query joins, aggregations, complex computations
-  Vertical scaling (getting a more powerful CPU / RAM / IO)
-  Horizontal scaling (increasing reading capability by adding EC2 / RDS Read Replicas) 
## NoSQL databases 
- NoSQL databases are non-relational databases and are distributed 
- NoSQL databases include MongoDB, DynamoDB, ... 
- NoSQL databases do not support query joins (or just limited support) 
- All the data that is needed for a query is present in one row 
- NoSQL databases don’t perform aggregations such as “SUM”, “AVG”, ... 
- NoSQL databases scale horizontally 
- There’s no “right or wrong” for NoSQL vs SQL, they just require to model the data differently and think about user queries differently 
## Amazon DynamoDB 
-  Fully managed, highly available with replication across multiple AZs
-  NoSQL database - not a relational database
-  Scales to massive workloads, distributed database
-  Millions of requests per seconds, trillions of row, 100s of TB of storage -  Fast and consistent in performance (low latency on retrieval) 
- Integrated with IAM for security, authorization and administration -  Enables event driven programming with DynamoDB Streams
-  Low cost and auto-scaling capabilities
-  Standard & Infrequent Access (IA) Table Class 
## DynamoDB - Basics 
-  DynamoDB is made of Tables
-  Each table has a Primary Key (must be decided at creation time) -  Each table can have an infinite number of items (= rows)
-  Each item has attributes (can be added over time – can be null) -  Maximum size of an item is 400KB 
-  Data types supported are:
   -  `Scalar Types` – String, Number, Binary, Boolean, Null -  Document Types – List, Map
   -  `Set Types` – String Set, Number Set, Binary Set 
## DynamoDB – Primary Keys 
-  **Option 1**: Partition Key (HASH)
   -  Partition key must be unique for each item 
   -  Partition key must be “diverse” so that the data is distributed -  Example:“User_ID” for a users table 

-  **Option 2**: Partition Key + Sort Key (HASH + RANGE)
   -  The combination must be unique for each item
   -  Data is grouped by partition key
   -  Example: users-games table, “User_ID” for Partition Key and “Game_ID” for Sort Key 

## DynamoDB – Partition Keys (Exercise) 
-	We’re building a movie database 
-	What is the best Partition Key to maximize data distribution? -  movie_id 
-  producer_name
-  leader_actor_name -  movie_language 
-	“movie_id” has the highest cardinality so it’s a good candidate 
-	“movie_language” doesn’t take many values and may be skewed towards English so it’s not a great choice for the Partition Key 
## DynamoDB – Read/Write Capacity Modes 
-  Control how you manage your table’s capacity (read/write throughput) 
-  **Provisioned Mode (default)**
   -  You specify the number of reads/writes per second 
   -  You need to plan capacity beforehand
   -  Pay for provisioned read & write capacity units 
-  **On-Demand Mode**
   -  Read/writes automatically scale up/down with your workloads 
   -  No capacity planning needed
   -  Pay for what you use, more expensive ($$$) 
-  You can switch between different modes once every 24 hours
## R/W Capacity Modes – Provisioned 
-	Table must have provisioned read and write capacity units 
-	`Read Capacity Units (RCU)` – throughput for reads 
-	`Write Capacity Units (WCU)` – throughput for writes 
-	Option to setup `auto-scaling` of throughput to meet demand 
-	Throughput can be exceeded temporarily using “Burst Capacity” 
-	If Burst Capacity has been consumed, you’ll get a `ProvisionedThroughputExceededException`
-	It’s then advised to do an `exponential backoff` retry

## DynamoDB – Write Capacity Units (WCU)
- One Write Capacity Unit (WCU) represents one write per second for an item up to 1 KB in size
- If the items are larger than 1 KB, more WCUs are consumed

| Scenario | Result
|--- |--- |
| we write 10 items per second, with item size 2 KB | We need <br> $10 * ({2 KB \over 1 KB}) = 20 WCUs$|
| we write 6 items per second, with item size 4.5 KB | We need <br> $6 * ({5 KB \over 1 KB}) = 20 WCUs$|
| we write 120 items per second, with item size 2 KB | We need <br> $({120 \over 60}) * ({2 KB \over 1 KB}) = 4 WCUs$|
## Strongly Consistent Read vs. Eventually Consistent Read
- Eventually Consistent Read (default)
- If we read just after a write, it’s possible we’ll get some stale data because of replication
- Strongly Consistent Read
- If we read just after a write, we will get the correct data
- Set `ConsistentRead` parameter to `True` in API calls (GetItem, BatchGetItem, Query, Scan)
- Consumes twice the RCU

DynamoDB – Read Capacity Units (RCU)
- One Read Capacity Unit (RCU) represents one Strongly Consistent Read per second, or two Eventually Consistent Reads per second, for an item up to 4 KB in size
- If the items are larger than 4 KB, more RCUs are consumed

| Scenario | Result
|--- |--- |
| 10 Strongly Consistent Reads per second, with item size 4 KB | We need <br>$10 * ({4 KB \over 4 KB}) = 10 RCUs$|
| 16 Eventually Consistent Reads per second, with item size 12 KB | We need <br> $({16 \over 2}) * ({12 KB \over 4 KB}) = 24 RCUs$|
| 10 Strongly Consistent Reads per second, with item size 6 KB | We need <br> $10 * ({8 KB \over 4 KB}) = 20 RCUs$ <br> round up 6KB to 8 KB|

## DynamoDB – Partitions Internal
- Data is stored in partitions
- Partition Keys go through a hashing algorithm to know to which partition they go to
To Compute the number of partions:
- No. of partitians <sub>by capacity</sub> = $({RCU_Total \over 3000}) + ({WCU_Total \over 1000})$

- No. of partitians <sub>by size</sub> = ${Total Size \over 10GB}$

- No. of partitions = ceil(max(# of particians<sub>bycapacity</sub>, # of partitions<sub>bysize</sub>))
- WCUs and RCUs are spread evenly across partitions

## DynamoDB – Throttling
- If we exceed provisioned RCUs or WCUs, we get `ProvisionedThroughputExceededException`
- Reasons:
    - `Hot Keys` – one partition key is being read too many times (e.g., popular item)
    - `Hot Partitions`
    - `Very large items`, remember RCU and WCU depends on size of items
- Solutions:
    - `Exponential backoff` when exception is encountered (already in SDK)
    - `Distribute partition` keys as much as possible
    - If RCU issue, we can use `DynamoDB Accelerator (DAX)`

## R/W Capacity Modes – On-Demand
- Read/writes automatically scale up/down with your workloads
- No capacity planning needed (WCU / RCU)
- Unlimited WCU & RCU, no throttle, more expensive
- You’re charged for reads/writes that you use in terms of RRU and WRU 
- `Read Request Units` (RRU) – throughput for reads (same as RCU)
- `Write Request Units` (WRU) – throughput for writes (same as WCU) 
- 2.5x more expensive than provisioned capacity (use with care)
- `Use cases`: unknown workloads, unpredictable application traffic, ...

## DynamoDB – Writing Data
**PutItem**
    - Creates a new item or fully replace an old item (same Primary Key) 
    - ConsumesWCUs 
**UpdateItem**
    - Edits an existing item’s attributes or adds a new item if it doesn’t exist
    - Can be used to implement Atomic Counters – a numeric attribute that’s unconditionally incremented
**Conditional Writes**
    - Accept a write/update/delete only if conditions are met, otherwise returns an error - Helps with concurrent access to items
    - No performance impact
## DynamoDB – Reading Data
- GetItem
- Read based on Primary key
- Primary Key can be HASH or HASH+RANGE
- Eventually Consistent Read (default)
- Option to use Strongly Consistent Reads (more RCU - might take longer)
- ProjectionExpression can be specified to retrieve only certain attributes

 
 ## DynamoDB – Reading Data (Query)
- Query returns items based on:
- KeyConditionExpression
    - Partition Key value (must be = operator) – required
    - SortKeyvalue(=,<,<=,>,>=,Between,Beginswith)–optional
- FilterExpression
    - Additional filtering after the Query operation (before data returned to you)
    - Use only with non-key attributes (does not allow HASH or RANGE attributes)
- Returns:
    - The number of items specified in Limit 
    - Or upto1MBofdata
- Ability to do pagination on the results
- Can query table, a Local Secondary Index, or a Global Secondary Index

 ## DynamoDB – Reading Data (Scan)
- Scan the entire table and then filter out data (inefficient)
- Returns up to 1 MB of data – use pagination to keep on reading
- Consumes a lot of RCU
- Limit impact using Limit or reduce the size of the result and pause
- For faster performance, use Parallel Scan
- Multiple workers scan multiple data segments at the same time
- Increases the throughput and RCU consumed
- Limit the impact of parallel scans just like you would for Scans
- Can use ProjectionExpression & FilterExpression (no changes to RCU) © Stephane Maarek

  
 ## DynamoDB – Deleting Data
- DeleteItem
    - Delete an individual item
    - Ability to perform a conditional delete
- DeleteTable
    - Delete a whole table and all its items
    - Much quicker deletion than calling DeleteItem on all items

## DynamoDB – Batch Operations
- Allows you to save in latency by reducing the number of API calls
- Operations are done in parallel for better efficiency
- Part of a batch can fail; in which case we need to try again for the failed items
- BatchWriteItem
    - Up to 25 PutItem and/or DeleteItem in one call
    - Up to 16 MB of data written,up to 400 KB of data per item 
    - Can’t update items (use UpdateItem)
- BatchGetItem
    - Return items from one or more tables
    - Up to 100 items,up to 16 MB of data
    - Items are retrieved in parallel to minimize latency


## DynamoDB – Local Secondary Index (LSI)
- Alternative Sort Key for your table (same Partition Key as that of base table)
- The Sort Key consists of one scalar attribute (String, Number, or Binary)
- Up to 5 Local Secondary Indexes per table
- Must be defined at table creation time
- `Attribute Projections` – can contain some or all the attributes of the base table (`KEYS_ONLY, INCLUDE, ALL`)

## DynamoDB – Global Secondary Index (GSI)
- `Alternative Primary Key (HASH or HASH+RANGE)` from the base table
- Speed up queries on non-key attributes
- The Index Key consists of scalar attributes (`String, Number, or Binary`)
- `Attribute Projections` – some or all the attributes of the base table (`KEYS_ONLY, INCLUDE, ALL`) - Must provision RCUs & WCUs for the index
NOT FOR DISTRIBUTION © Stephane Maarek www.datacumulus.com
- `Can be added/modified after table creation`

## DynamoDB - PartiQL
- Use a SQL-like syntax to manipulate DynamoDB tables
- Supports some (but not all) statements: 
    - INSERT
    - UPDATE 
    - SELECT 
    - DELETE
- It supports Batch operations

## DynamoDB – Optimistic Locking
- DynamoDB has a feature called “Conditional Writes”
- A strategy to ensure an item hasn’t changed before you update/delete it
- Each item has an attribute that acts as a version number

## DAX
- Fully-managed, highly available, seamless in-memory cache for DynamoDB
- Microseconds latency for cached reads & queries
- Doesn’t require application logic modification (compatible with existing DynamoDB APIs)
- Solves the “Hot Key” problem (too many reads)
- 5 minutes TTL for cache (default)
- Up to 10 nodes in the cluster
- Multi-AZ (3 nodes minimum recommended for production)
- Secure (Encryption at rest with KMS,VPC, IAM, CloudTrail, ...)

## DynamoDB Streams
- Ordered stream of item-level modifications (create/update/delete) in a table
- Stream records can be:
- Sent to Kinesis Data Streams
- Read by AWS Lambda
- Read by Kinesis Client Library applications
- Data Retention for up to 24 hours
- Use cases:
    - react to changes in real-time (welcome email to users)
    - Analytics
    - Insert into derivative tables
    - Insert into ElasticSearch
    - Implement cross-region replication
## DynamoDB Streams
- Ability to choose the information that will be written to the stream: 
    - `KEYS_ONLY` – only the key attributes of the modified item
    - `NEW_IMAGE` – the entire item, as it appears after it was modified
    - `OLD_IMAGE` – the entire item, as it appeared before it was modified
    - `NEW_AND_OLD_IMAGES` – both the new and the old images of the item
- DynamoDB Streams are made of shards, just like Kinesis Data Streams 
- You don’t provision shards, this is automated by AWS
- Records are not retroactively populated in a stream after enabling it

## DynamoDB Streams & AWS Lambda
- You need to define an Event Source Mapping to read from a DynamoDB Streams
- You need to ensure the Lambda function has the appropriate permissions
- Your Lambda function is invoked synchronously

## DynamoDB –TimeTo Live (TTL)
- Automatically delete items after an expiry timestamp
- Doesn’t consume any WCUs (i.e., no extra cost)
- The TTL attribute must be a “Number” data type with “Unix Epoch timestamp” value
- Expired items deleted within 48 hours of expiration
- Expired items, that haven’t been deleted, appears in reads/queries/scans (if you don’t want them, filter them out)
- Expired items are deleted from both LSIs and GSIs
- A delete operation for each expired item enters the DynamoDB Streams (can help recover expired items)
- Use cases: reduce stored data by keeping only current items, adhere to regulatory obligations, ...

## DynamoDB CLI – Good to Know
- `--projection-expression`: one or more attributes to retrieve 
- `--filter-expression`: filter items before returned to you
- General AWS CLI Pagination options (e.g., DynamoDB, S3, ...)
    - `--page-size`: specify that AWS CLI retrieves the full list of items but with a larger number of API calls instead of one API call (default: 1000 items)
    - `--max-items`: max. number of items to show in the CLI (returns NextToken)
    - `--starting-token`: specify the last NextToken to retrieve the next set of items

## DynamoDBTransactions
- Coordinated, all-or-nothing operations (add/update/delete) to multiple items across one or more tables
- Provides Atomicity, Consistency, Isolation, and Durability (ACID)
- `Read Modes` – Eventual Consistency, Strong Consistency,Transactional
- `Write Modes` – Standard,Transactional
- `Consumes 2x WCUs & RCUs`
    - DynamoDB performs 2 operations for every item (prepare & commit)
- Two operations: (up to 25 unique items or up to 4 MB of data)
    - `TransactGetItems` – one or more GetItem operations
    - `TransactWriteItems` – one or more PutItem, UpdateItem, and DeleteItem operations
- Use cases: financial transactions, managing orders, multiplayer games, ...

## DynamoDB as Session State Cache 
- It’s common to use DynamoDB to store session states
- vs. ElastiCache
    - ElastiCache is in-memory, but DynamoDB is serverless 
    - Both are key/value stores
- vs. EFS
    - EFS must be attached to EC2 instances as a network drive
- vs. EBS & Instance Store
    - EBS & Instance Store can only be used for local caching, not shared caching
- vs. S3
    - S3 is higher latency, and not meant for small objects

## DynamoDB Write Sharding
- Imagine we have a voting application with two candidates, candidate A and candidate B
- If `Partition Key` is “Candidate_ID”, this results into two partitions, which will generate issues (e.g., Hot Partition)
- A strategy that allows better distribution of items evenly across partitions
- Add a suffix to Partition Key value
- Two methods:
    - Sharding Using Random Suffix
    - Sharding Using Calculated Suffix

## DynamoDB Operations
- Table Cleanup
    - Option 1: Scan + DeleteItem
      - Very slow,consumesRCU&WCU,expensive
- Option 2: Drop Table + Recreate table 
    - Fast,efficient,cheap
- Copying a DynamoDB Table
    - Option 1: Using AWS Data Pipeline
    - Option 2: Backup and restore into a new table
        - Takes some time
    - Option 3: Scan + PutItem or BatchWriteItem
        - Write your own code

## DynamoDB – Security & Other Features
- **Security**
    - VPC Endpoints available to access DynamoDB without using the Internet
    - Access fully controlled by IAM
    - Encryption at rest using AWS KMS and in-transit using SSL/TLS
- **Backup and Restore feature available **
    - Point-in-time Recovery (PITR) like RDS 
    - No performance impact
- **GlobalTables**
    - Multi-region,multi-active,fullyreplicated,highperformance
- **DynamoDB Local**
    - Develop and test apps locally without accessing the DynamoDB web service (without Internet)
- AWS Database Migration Service (AWS DMS) can be used to migrate to DynamoDB (from MongoDB, Oracle, MySQL, S3, ...)

## DynamoDB – Fine-Grained Access Control
- Using `Web Identity Federation` or `Cognito Identity Pools`, each user gets AWS credentials
- You can assign an IAM Role to these users with a `Condition` to limit their API access to DynamoDB
- `LeadingKeys` – limit row-level access for users on the Primary Key
- `Attributes` – limit specific attributes the user can see

# API Gateway
- AWS Lambda + API Gateway: No infrastructure to manage 
- Support for the WebSocket Protocol
- Handle API versioning (v1, v2...)
- Handle different environments (dev, test, prod...)
- Handle security (Authentication and Authorization) 
- Create API keys, handle request throttling
- Swagger / Open API import to quickly define APIs 
- Transform and validate requests and responses
- Generate SDK and API specifications 
- Cache API responses

## API Gateway – Integrations High Level
- Lambda Function
  - Invoke Lambda function
  - Easy way to expose REST API backed by AWS Lambda
- HTTP
  - Expose HTTP endpoints in the backend
  - Example: internal HTTP API on premise, Application Load Balancer... 
  - Why? Add rate limiting, caching, user authentications, API keys, etc...
- AWS Service
  - Expose any AWS API through the API Gateway?
  - Example: start an AWS Step Function workflow, post a message to SQS
  - Why? Add authentication, deploy publicly, rate control...
API Gateway - Endpoint Types
- `Edge-Optimized (default)` For global clients
  - Requests are routed through the CloudFront Edge locations (improves latency) 
  - The API Gateway still lives in only one region
- `Regional`
  - For clients within the same region
  - Could manually combine with CloudFront (more control over the caching strategies and the distribution)
- `Private`
  - Can only be accessed from your VPC using an interface VPC endpoint (ENI)
  - Use a resource policy to define access
## API Gateway – Deployment Stages
- Making changes in the API Gateway does not mean they’re effective 
- You need to make a “deployment” for them to be in effect
- It’s a common source of confusion
- Changes are deployed to “Stages” (as many as you want)
- Use the naming you like for stages (dev, test, prod)
- Each stage has its own configuration parameters
- Stages can be rolled back as a history of deployments is kept

## API Gateway – Stage Variables
- Stage variables are like environment variables for API Gateway
- Use them to change often changing configuration values
- They can be used in:
  - Lambda function ARN
  - HTTP Endpoint
  - Parameter mapping templates
**Use cases:**
  - Configure HTTP endpoints your stages talk to (dev, test, prod...)
  - Pass configuration parameters to AWS Lambda through mapping templates
  - Stage variables are passed to the ”context” object in AWS Lambda

## API Gateway Stage Variables & Lambda Aliases 
- We create a stage variable to indicate the corresponding Lambda alias
- Our API gateway will automatically invoke the right Lambda function!

## API Gateway – Canary Deployment
- Possibility to enable canary deployments for any stage (usually prod)
- Choose the % of traffic the canary channel receives
- Metrics & Logs are separate (for better monitoring)
- Possibility to override stage variables for canary
- This is blue / green deployment with AWS Lambda & API Gateway

## API Gateway - IntegrationTypes - 
**Integration Type MOCK**
- API Gateway returns a response without sending the request to the backend 
**IntegrationType HTTP / AWS (Lambda & AWS Services)**
- you must configure both the integration request and integration response 
- Setup data mapping using mapping templates for the request & response
**Integration Type AWS_PROXY (Lambda Proxy)**
- incoming request from the client is the input to Lambda
- The function is responsible for the logic of request / response
- No mapping template, headers, query string parameters... are passed as arguments
**Integration Type HTTP_PROXY **
- No mapping template
- The HTTP request is passed to the backend
- The HTTP response from the backend is forwarded by API Gateway
## Mapping Templates (AWS & HTTP Integration)
- Mapping templates can be used to modify request / responses 
- Rename / Modify query string parameters
- Modify body content
- Add headers
- Uses Velocity Template Language (VTL): for loop, if etc... 
- Filter output results (remove unnecessary data)
**Example**
SOAP API are XML based, whereas REST API are JSON based
In this case, API Gateway should:
- Extract data from the request: either path, payload or header
- Build SOAP message based on request data (mapping template)
- Call SOAP service and receive XML response
- Transform XML response to desired format (like JSON), and respond to the user

## AWS API Gateway Swagger / Open API spec 
- Common way of defining REST APIs, using API definition as code
- Import existing Swagger / OpenAPI 3.0 spec to API Gateway 
  - Method
  - Method Request
  - Integration Request
  - Method Response
  - + AWS extensions for API gateway and setup every single option
- Can export current API as Swagger / OpenAPI spec
- Swagger can be written inYAML or JSON
- Using Swagger we can generate SDK for our applications

## Caching API responses
- Caching reduces the number of calls made to the backend
- Default TTL (time to live) is 300 seconds (min: 0s, max: 3600s)
- Caches are defined per stage
- Possible to override cache settings per method
- Cache encryption option
- Cache capacity between 0.5GB to 237GB
- Cache is expensive, makes sense in production, may not make sense in dev / test

## API Gateway Cache Invalidation
- Able to flush the entire cache (invalidate it) immediately
- Clients can invalidate the cache with header: `Cache- Control: max-age=0`(with proper IAM authorization)
- If you don't impose
an InvalidateCache policy (or choose the Require authorization check box in the console), any client can invalidate the API cache

## API Gateway – Usage Plans & API Keys
- If you want to make an API available as an offering ($) to your customers
- **Usage Plan:**
  - who can access one or more deployed API stages and methods
  - how much and how fast they can access them
  - uses API keys to identify API clients and meter access
  - configure throttling limits and quota limits that are enforced on individual client
- **API Keys:**
  - alphanumeric string values to distribute to your customers
  - Ex:WBjHxNtoAb4WPKBC7cGm64CBibIb24b4jt8jJHo9
  - Can use with usage plans to control access
  - Throttling limits are applied to the API keys
  - Quotas limits is the overall number of maximum requests

## API Gateway – Correct Order for API keys
- To configure a usage plan
  1. Create one or more APIs, configure the methods to require an API key, and deploy the APIs to stages.
  2. Generate or import API keys to distribute to application developers (your customers) who will be using your API.
  3. Create the usage plan with the desired throttle and quota limits.
  4. Associate API stages and API keys with the usage plan.
- Callers of the API must supply an assigned API key in the x-api-key header in requests to the API.

## API Gateway – Logging & Tracing
- CloudWatch Logs:
    - Enable CloudWatch logging at the Stage level (with Log Level)
    - Can override settings on a per API basis (ex: ERROR, DEBUG, INFO) 
    - Log contains information about request / response body
- X-Ray:
    -  Enable tracing to get extra information about requests in API Gateway
    - X-Ray API Gateway + AWS Lambda gives you the full picture

## API Gateway – CloudWatch Metrics
- Metrics are by stage, Possibility to enable detailed metrics
- `CacheHitCount & CacheMissCount`: efficiency of the cache
- `Count`:The total number API requests in a given period.
- `IntegrationLatency`:The time between when API Gateway relays a request to the backend and when it receives a response from the backend.
- `Latency`:The time between when API Gateway receives a request from a client and when it returns a response to the client.The latency includes the integration latency and other API Gateway overhead.
- `4XXError (client-side) & 5XXError (server-side)`

## API Gateway Throttling
- Account Limit
    - API Gateway throttles requests at10000 rps across all API 
    - Soft limit that can be increased upon request
- In case of throttling => 429 Too Many Requests (retriable error)
- Can set Stage limit & Method limits to improve performance
- Or you can define Usage Plans to throttle per customer
> Just like Lambda Concurrency, one API that is overloaded, if not limited, can cause the other APIs to be throttled

## API Gateway - Errors
- 4xx means Client errors 
    - 400: Bad Request
    - 403:Access Denied,WAF filtered 
    - 429: Quota exceeded,Throttle
- 5xx means Server errors
    - 502: Bad Gateway Exception, usually for an incompatible output returned from a Lambda proxy integration backend and occasionally for out-of-order invocations due to heavy loads.
    - 503: Service Unavailable Exception
    - 504: Integration Failure – ex Endpoint Request Timed-out Exception API Gateway requests time out after 29 second maximum

## AWS API Gateway - CORS
- CORS must be enabled when you receive API calls from another domain.
- The OPTIONS pre-flight request must contain the following headers: 
    - Access-Control-Allow-Methods
    - Access-Control-Allow-Headers 
    - Access-Control-Allow-Origin
- CORS can be enabled through the console

![API-CORS](https://user-images.githubusercontent.com/33105405/186798417-7087d05a-8bc1-4980-ab5b-84169acf519a.png)

## API Gateway – Resource Policies
- Resource policies (similar to Lambda Resource Policy)
- Allow for Cross Account Access (combined with IAM Security)
- Allow for a specific source IP address
- Allow for a VPC Endpoint
  
## API Gateway – Security 
### IAM Permissions
- Create an IAM policy authorization and attach to User / Role
- Authentication = IAM | Authorization = IAM Policy 
- Good to provide access within AWS (EC2, Lambda, IAM users...)
- Leverages “Sig v4” capability where IAM credential are in headers

### Cognito User Pools
- Cognito fully manages user lifecycle, token expires automatically
- API gateway verifies identity automatically from AWS Cognito
- No custom implementation required
- Authentication = Cognito User Pools | Authorization = API Gateway Methods

### Lambda Authorizer (formerly Custom Authorizers)
- Token-based authorizer (bearer token) – ex JWT (JSON Token) or Oauth
- A request parameter-based Lambda authorizer (headers, query string, stage var)
- Lambda must return an IAM policy for the user, result policy is cached
- Authentication = External | Authorization = Lambda function

### Summary
- IAM:
    - Great for users / roles already within your AWS account, + resource policy for cross account 
    - Handle authentication + authorization
    - Leverages Signature v4
- Custom Authorizer:
    - Great for 3rd party tokens
    - Very flexible in terms of what IAM policy is returned
    - Handle Authentication verification + Authorization in the Lambda function 
    - Pay per Lambda invocation, results are cached
- Cognito User Pool:
    - You manage your own user pool (can be backed by Facebook, Google login etc...) 
    - No need to write any custom code
    - Must implement authorization in the backend

## API Gateway – HTTP API vs REST API
- **HTTP APIs**
  - low-latency, cost-effective AWS Lambda proxy, HTTP proxy APIs and private integration (no data mapping)
  - support OIDC and OAuth 2.0 authorization, and built-in support for CORS
  - No usage plans and API keys 
- **REST APIs**
  - All features (except Native OpenID Connect / OAuth 2.0)
## API Gateway – WebSocket API – Overview
**WebSockets**
  - Two-way interactive communication between a user’s browser and a server
  - Server can push information to the client 
  - This enables stateful application use cases
- WebSocket APIs are often used in real- time applications such as chat applications, collaboration platforms, multiplayer games, and financial trading platforms
- Works with AWS Services (Lambda, DynamoDB) or HTTP endpoints

## API Gateway – WebSocket API – Routing
- Incoming JSON messages are routed to different backend
- If no routes => sent to `$default`
- You request a route selection expression to select the field on JSON to route from
- Sample expression: `$request.body.action`
- The result is evaluated against the route keys available in your API Gateway
- The route is then connected to the backend you’ve setup through API Gateway

![API-WebSocs](https://user-images.githubusercontent.com/33105405/186798661-5bf00ac2-3a09-47d2-95f5-964b67d9a35e.png)

## API Gateway - Architecture
- Create a single interface for all the microservices in your company
- Use API endpoints with various resources
- Apply a simple domain name and SSL certificates
- Can apply forwarding and transformation rules at the API Gateway level


# Amazon Cognito
## Overview
To give our users an identity so that they can interact with our application.
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
| | Migrate User Lambda Trigger | Migrate a user from an existing user directory to user pools |
| Messages | Custom message lambda trigger | Advanced customisation and localization of messages |
| Token Creation | Pre Token Generation Lambda Trigger | Add or Remove attribtutes in ID tokens |


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

# AWS Simple Token Service
## Overview
- Allows to grant limited and temporary access to AWS resources (up to 1 hour). 
- `AssumeRole`: Assume roles within your account or cross account
- `AssumeRoleWithSAML`: return credentials for users logged with SAML
- `AssumeRoleWithWebIdentity`
  - return creds for users logged with an IdP (Facebook Login, Google Login, OIDC compatible...) 
  - AWS recommends against using this, and using Cognito Identity Pools instead
- `GetSessionToken`: for MFA, from a user or AWS account root user
- `GetFederationToken`: obtain temporary creds for a federated user
- `GetCallerIdentity`: return details about the IAM user or role used in the API call
- `DecodeAuthorizationMessage`: decode error message when an AWS API is denied
### STS to Assume a Role
- Define an IAM Role within your account or cross-account
- Define which principals can access this IAM Role
- Use AWS STS to retrieve credentials and impersonate the IAM Role you have access to (`AssumeRole API`)
- Temporary credentials can be valid between 15 minutes to 1 hour
### STS with MFA
- Use GetSessionToken from STS
- Appropriate IAM policy using IAM Conditions
- aws:MultiFactorAuthPresent:true
- Reminder, GetSessionToken returns:
  - Access ID
  - Secret Key
  - SessionToken 
  - Expiration date

### IAM Best Practices – General
- Never use Root Credentials, enable MFA for Root Account
- Grant Least Privilege
  - Each Group / User / Role should only have the minimum level of permission it needs
  - Never grant a policy with “*” access to a service
  - Monitor API calls made by a user in CloudTrail (especially Denied ones)
- Never ever ever store IAM key credentials on any machine but a personal computer or on-premise server
- On premise server best practice is to call STS to obtain temporary security credentials

### IAM Best Practices – IAM Roles
- EC2 machines should have their own roles
- Lambda functions should have their own roles
- ECS Tasks should have their own roles (`ECS_ENABLE_TASK_IAM_ROLE=true`)
- CodeBuild should have its own service role
- Create a least-privileged role for any service that requires it
- Create a role per application / lambda function (do not reuse roles)

### IAM Best Practices – Cross Account Access
- Define an IAM Role for another account to access
- Define which accounts can access this IAM Rile
- Use AWS STS to retrieve credentials and impersonate the IAM Rolw you have access to (Assume Role API)
- Tempeory credentials valid between `15 mins to 1 hr`

### Authorization Model
Evaluation of Policies, simplified
1. If there’s an explicit DENY, end decision and DENY
2. If there’s an ALLOW, end decision with ALLOW
3. Else DENY

### IAM Policies & S3 Bucket Policies
- IAM Policies are attached to users, roles, groups
- S3 Bucket Policies are attached to buckets
- When evaluating if an IAM Principal can perform an operation X on a bucket, the `union` of its assigned IAM Policies and S3 Bucket Policies will be evaluated.

| Scenario | Outcome |
|--- |--- |
| - IAM Role attached to EC2 instance, authorizes RW to “my_bucket” <br> - No S3 Bucket Policy attached | EC2 instance `can` read and write to `my_bucket` |
| - IAM Role attached to EC2 instance, authorizes RW to “my_bucket” <br> - S3 Bucket Policy attached, explicit deny to the IAM Role | EC2 instance `cannot` read and write to `my_bucket` |
| - IAM Role attached to EC2 instance, no S3 bucket permissions <br> - S3 Bucket Policy attached, explicit RW allow to the IAM Role | EC2 instance `can` read and write to `my_bucket` |
| - IAM Role attached to EC2 instance, explicit deny S3 bucket permissions <br> - S3 Bucket Policy attached, explicit RW allow to the IAM Role | EC2 instance `cannot` read and write to `my_bucket` |

Dynamic Policies with IAM
- How do you assign each user a /home/<user> folder in an S3 bucket?
**Option 1**
  - Create an IAM policy allowing georges to have access to /home/georges - Create an IAM policy allowing sarah to have access to /home/sarah
  - Create an IAM policy allowing matt to have access to /home/matt
  - ... One policy per user!
  - This doesn’t scale
**Option 2**
- Create one dynamic policy with IAM
- Leverage the special policy variable ${aws:username}

Dynamic Policy Sample

```json
{
    "Effect": "Allow", 
    "Action": ["s3:*"], 
    "Resource": ["arn:aws:s3:::{{bucket}}/{{aws:username}}/*"] 
}
```
### Inline vs Managed Policies
- AWS Managed Policy 
  - Maintained by AWS
  - Good for power users and administrators
  - Updated in case of new services / new APIs
- Customer Managed Policy
  - Best Practice, re-usable, can be applied to many principals 
  - Version Controlled + rollback, central change management
- Inline
  - Strict one-to-one relationship between policy and principal
  - Policy is deleted if you delete the IAM principal
### Granting a User Permissions to Pass a Role to an AWS Service
- To configure many AWS services, you must `pass` an IAM role to the service (this happens only once during setup)
- The service will later assume the role and perform actions
- Example of passing a role: - To an EC2 instance
  - To a Lambda function
  - To an ECS task
  - To CodePipeline to allow it to invoke other services
- For this, you need the IAM permission `iam:PassRole`
- It often comes with iam:GetRole to view the role being passed

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": ["ec2:*"],
            "Resource": "*",
        }
        
        {
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": "arn:aws:iam::12345678912:role/S3Access",
        }
    ]
}
```
#### Can a role be passed to any service?
- No: Roles can only be passed to what their trust allows
- A trust policy for the role that allows the service to assume the role
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111122223333:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {}
    }
  ]
}
```
### What is Microsoft Active Directory (AD)?
- Found on any Windows Serverwith AD Domain Service
- Database of objects: User Accounts, Computers, Printers, File Shares, Security Groups
- Centralized security management, create account, assign permissions
- Objects are organized in trees
- A group of trees is a forest

### AWS Directory Services

| Service | Description |
| --- | --- |
| AWS Managed Microsoft AD | - Create your own AD in AWS, manage users locally, supports MFA <br>- Establish “trust” connections with your on- premise AD |
| AD Connector | - Directory Gateway (proxy) to redirect to on- premise AD, supports MFA <br> - Users are managed on the on-premise AD | 
| Simple AD | - AD-compatible managed directoru on AWS <br> - Cannot be joined with on-premise AD|
# AWS Security and Encryption
### Encruption in Flight (SSL)
- Data is encrypted before sending and decrypted after receiving
- SSL certificates help with encryption (HTTPS)
- Encryption in flight ensures no MITM (man in the middle attack) can happen
### Server side encryption at rest
- Data is encrypted after being received by the server
- Data is decrypted before being sent
- It is stored in an encrypted form thanks to a key (usually a data key)
- The encryption / decryption keys must be managed somewhere and the server must have access to it
### Client side encryption
- Data is encrypted by the client and never decrypted by the server 
- Data will be decrypted by a receiving client
- The server should not be able to decrypt the data
- Could leverage Envelope Encryption
## AWS KMS (Key Management Service)
- Anytime you hear “encryption” for an AWS service, it’s most likely KMS 
- Easy way to control access to your data, AWS manages keys for us
- Fully integrated with IAM for authorization
- Seamlessly integrated into:
  - Amazon EBS: encrypt volumes
  - Amazon S3: Server side encryption of objects - Amazon Redshift: encryption of data
  - Amazon RDS: encryption of data
  - Amazon SSM: Parameter store
  - Etc...
- But you can also use the CLI / SDK
## KMS – Customer Master Key (CMK)Types
***Symmetric (AES-256 keys)***
  - First offering of KMS, single encryption key that is used to Encrypt and Decrypt
  - AWS services that are integrated with KMS use Symmetric CMKs
  - Necessary for envelope encryption
  - You never get access to the Key unencrypted (must call KMS API to use)
***Asymmetric (RSA & ECC key pairs)***
  - Public (Encrypt) and Private Key (Decrypt) pair
  - Used for Encrypt/Decrypt, or Sign/Verify operations
  - The public key is downloadable, but you can’t access the Private Key unencrypted
  - Use case: encryption outside of AWS by users who can’t call the KMS API
### KMS (Key Management Service)
- Able to fully manage the keys & policies: 
  - Create
  - Rotation policies
  - Disable
  - Enable
- Able to audit key usage (using CloudTrail)
- Three types of Customer Master Keys (CMK):
  - AWS Managed Service Default CMK: free
  - User Keys created in KMS: $1 / month
  - User Keys imported (must be 256-bit symmetric key): $1 / month
- pay for API call to KMS ($0.03 / 10000 calls)
- Anytime you need to share sensitive information... use KMS 
  - Database passwords
  - Credentials to external service 
  - Private Key of SSL certificates
- The value in KMS is that the CMK used to encrypt data can never be retrieved by the user, and the CMK can be rotated for extra security
- Never ever store your secrets in plaintext, especially in your code!
- Encrypted secrets can be stored in the code / environment variables - KMS can only help in encrypting up to 4KB of data per call
- `If data > 4 KB, use envelope encryption`
- To give access to KMS to someone:
  - Make sure the Key Policy allows the user
  - Make sure the IAM Policy allows the API calls

### KMS Key Policies
- Control access to KMS keys, “similar” to S3 bucket policies
- Difference: you cannot control access without them
- Default KMS Key Policy:
  - Created if you don’t provide a specific KMS Key Policy
  - Complete access to the key to the root user = entire AWS account 
  - Gives access to the IAM policies to the KMS key
- Custom KMS Key Policy:
  - Define users, roles that can access the KMS key
  - Define who can administer the key
  - Useful for cross-account access of your KMS key

### Copying Snapshots across accounts
1. Create a Snapshot, encr ypted with your own CMK
2. Attach a KMS Key Policy to authorize cross-account access
3. Share the encr ypted snapshot
4. (in target) Create a copy of the Snapshot, encrypt it with a KMS Key in your account
5. Create a volume from the snapshot

### Envelope Encryption
- KMS Encrypt API call has a limit of 4 KB
- If you want to encrypt >4 KB, we need to use Envelope Encryption
- The main API that will help us is the GenerateDataKey API
  > **For the exam**: anything over 4 KB of data that needs to be encrypted must use the Envelope Encryption == `GenerateDataKey API`

### Encryption SDK
- The AWS Encryption SDK implemented Envelope Encryption for us 
- The Encryption SDK also exists as a CLI tool we can install
- Implementations for Java, Python, C, JavaScript
- Feature - Data Key Caching:
  - re-use data keys instead of creating new ones for each encryption
  - Helps with reducing the number of calls to KMS with a security trade-off
  - Use LocalCryptoMaterialsCache (max age, max bytes, max number of messages)

### KMS Symmetric – API Summary
- `Encrypt`: encrypt up to 4 KB of data through KMS
- `GenerateDataKey`: generates a unique symmetric data key (DEK) 
  - returns a plaintext copy of the data key
  - AND a copy that is encrypted under the CMK that you specify
- `GenerateDataKeyWithoutPlaintext`:
  - Generate a DEK to use at some point (not immediately)
  - DEK that is encrypted under the CMK that you specify (must use Decrypt later)
- `Decrypt`: decrypt up to 4 KB of data (including Data Encryption Keys) 
- `GenerateRandom`: Returns a random byte string

### KMS Request Quotas
- When you exceed a request quota, you get a ThrottlingException:
- To respond, use exponential backoff (backoff and retry)
- For cryptographic operations, they share a quota
- This includes requests made by AWS on your behalf (ex: SSE-KMS)
- For GenerateDataKey, consider using DEK caching from the Encryption SDK 
- You can request a Request Quotas increase through API or AWS support

| API Operations | Request quotas (per second) |
|--- |--- |
| Decrypt <br> Encrypt <br>GenerateDataKey (symmetric) <br> GenerateDataKeyWithoutPlaintext (symmetric) <br> GenerateRandom <br> ReEncrypt<br> Sign (asymmetric)<br> Verify (asymmetric) | These shared quotas vary with the AWS Region and the type of CMK used in the request. Each quota is calculated separately. <br> Symmetric CMK quota: <br> - 5,500 (shared) <br> - 10,000 (shared) in the following regions: us-east2, ap-southeast-1, ap-southeast-2, ap-norteast-1, eu-central-1, eu-west-2 <br> - 30,000 (shared) in us-east-1, us-west-2,eu-west-1 <br> Asymmetric CMK quota: <br> - 500 (shared) for RSA CMKs <br> - 300 (shared) for Elliptic curve (ECC) CMK |

## S3 Encryption for Objects
- There are 4 methods of encrypting objects in S3
  - SSE-S3: encrypts S3 objects using keys handled & managed by AWS
  - SSE-KMS: leverage AWS Key Management Service to manage encryption keys 
  - SSE-C: when you want to manage your own encryption keys
  - Client Side Encryption
> It’s important to understand which ones are adapted to which situation for the exam

### SSE-KMS
- encryption using keys handled & managed by KMS
- KMS Advantages: user control + audit trail
- Object is encrypted server side
- Must set header: `"x-amz-server-side-encryption": ”aws:kms"`
- SSE-KMS leverages the GenerateDataKey & Decrypt KMS API calls 
- These KMS API calls will show up in CloudTrail, helpful for logging
- To perform SSE-KMS, you need:
  - A KMS Key Policy that authorizes the user / role
  - An IAM policy that authorizes access to KMS
  - Otherwise you will get an access denied error
- S3 calls to KMS for SSE-KMS count against your KMS limits 
  - If throttling, try exponential backoff
  - If throttling, you can request an increase in KMS limits
  - The service throttling is KMS, not Amazon S3

### S3 Bucket Policies – Force SSL
- To force SSL, create an S3 bucket policy with a DENY on the condition `aws:SecureTransport = false`
  > **Note**: Using an allow on `aws:SecureTransport = true` would allow anonymous GetObject if using SSL
- Read more here: https://aws.amazon.com/premiumsupport/knowledge-center/s3-bucket-policy-for-config-rule/

```json
{
  "Id": "ExamplePolicy",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowSSLRequestsOnly",
      "Action": "s3:*",
      "Effect": "Deny",
      "Resource": [
        "arn:aws:s3:::DOC-EXAMPLE-BUCKET",
        "arn:aws:s3:::DOC-EXAMPLE-BUCKET/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      },
      "Principal": "*"
    }
  ]
}
```

### S3 Bucket Policy – Force Encryption of SSE-KMS
1. Deny incorrect encryption header: make sure it includes aws:kms (== SSE-KMS)
2. Deny no encryption header to ensure objects are not uploaded un-encrypted
> Note: could swap 2) for S3 default encryption of SSE-KMS

### S3 Bucket Key for SSE-KMS encryption
- New setting to decrease...
  - Number of API calls made to KMS from S3 by 99%
  - Costs of overall KMS encryption with Amazon S3 by 99%
- This leverages data keys
  - A “S3 bucket key” is generated
  - That key is used to encrypt KMS objects with new data keys
- You will see less KMS CloudTrail events in CloudTrail

## SSM Parameter Store
- Secure storage for configuration and secrets 
- Optional Seamless Encryption using KMS
- Serverless, scalable, durable, easy SDK
- Version tracking of configurations / secrets
- Configuration management using path & IAM 
- Notifications with CloudWatch Events
- Integration with CloudFormation
- SSM Parameter Store Hirearchy

```ruby
/my-dept/
    my-app/
        dev/
            db-url
            db-password
        prod/
            db-url
            db-password
    next-app/
/other-dept/
/aws/reference/secretsmanager/secret_ID_inSecrets_Manager
/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2

```
### Standard and Advanced parameter tiers

|  	| Standard 	| Advanced 	|
|---	|---	|---	|
| Total number of parameters allowed (per AWS account and region) 	| 10,000 	| 100,000 	|
| (per AWS account and 	| 4 KB 	| 8 KB 	|
| Region) 	| No 	| Yes 	|
| Cost 	| No additional charge 	| charges apply 	|
| Storage Pricing 	| Free 	| $0.05 per advanced parameter per month 	|
| API Interaction Pricing <br> Higher throughput = up to 1000 transactions / sec) 	| Standard Throughput: free <br> Higher Throughput: $0.05 per 10,000 API interactions 	| Standard Throughput: $0.05 per 10,000 API interactions<br><br>Higher Throughput: $0.05 per 10,000 API interactions 	|

### Parameters Policies (for advanced parameters)
- Allow to assign a TTL to a parameter (expiration date) to force updating or deleting sensitive data such as passwords
- Can assign multiple policies at a time

### AWS Secrets Manager
- Newer service, meant for storing secrets
- Capability to force rotation of secrets every X days
- Automate generation of secrets on rotation (uses Lambda)
- Integration with Amazon RDS (MySQL, PostgreSQL, Aurora) - Secrets are encrypted using KMS
- Mostly meant for RDS integration

### SSM Parameter Store vs Secrets Manager

| Secerets Manager | SSM Parameter Store |
|--- |--- |
| $$$ | $ |
| - Automatic rotation of secrets with AWS Lambda <br> - Lambda function is provided for RDS, Redshift, DocumentDB <br> - KMS encryption is mandatory <br> - Can integration with CloudFormation | - Simple API <br>- No secret rotation (can enable rotation using Lambda triggered by CW Events) <br>- KMS encryption is optional <br>- Can integration with CloudFormation <br>- Can pull a Secrets Manager secret using the SSM Parameter Store API |

## CloudWatch Logs - Encryption
- You can encrypt CloudWatch logs with KMS keys
- Encryption is enabled at the log group level, by associating a CMK with a log group, either when you create the log group or after it exists.
- You cannot associate a CMK with a log group using the CloudWatch console.
- You must use the CloudWatch Logs API:
- `associate-kms-key` : if the log group already exists
- `create-log-group` : if the log group doesn’t exist yet

## CodeBuild Security
- To access resources in your VPC, make sure you specify a VPC configuration for your CodeBuild
- Secrets in CodeBuild:
- Don’t store them as plaintext in environment variables
- Instead...
  - Environment variables can reference parameter store parameters 
  - Environment variables can reference secrets manager secrets
