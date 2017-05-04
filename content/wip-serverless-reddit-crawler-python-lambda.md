Title: Serverless Reddit Crawling With Python And AWS Lambdas
Date: 2017-05-13 10:20
Category: Articles
Status: draft

Amazon recently announced Python 3.6 support for AWS Lambda, so this is a good
time to explore what we can build with this. I decided to use Lambdas to create a data pipeline that 
can be used for crawling top posts from a given Reddit subreddit, and store any linked
images into S3 as a demonstration. Because how better to explore new tech than by
downloading hundreds of cat pictures from Reddit? Better yet if we don't have to host
a single server to achieve this.

Pic here: 
1. AWS Lambda
2. ???
3. Profit + cat pictures

Lambdas are serverless, event-driven elements of the AWS platform, which means you do
not have to worry about where to host the code, and how to scale it out. I think of them 
as cloud-hosted functions that will be called by AWS infrastructure to respond to some 
event, e.g. an HTTP call or a timed event.

This article will demonstrate how to use Lambdas to:

* Handle an HTTP request
* Process an SNS event
* Run on a schedule
* Chain Lambdas to process SQS messages

The pipeline will accept an HTTP request, triggering the crawl of the specified subreddit. 
This will load the top 100 posts from said subreddit (e.g. XXXXXXX), and then persist all pictures
linked by each of the posts in S3.

The finished pipeline works as follows:
![Reddit Crawler Serverless Pipeline]({filename}/images/reddit-scrape-aws-pipeline.png)

1. User POSTs a request to an HTTP endpoint requesting a certain subreddit to be crawled, e.g. /r/aww
2. This request is handled by the first Lambda, titled 'HTTP API Endpoint' in the blueprit. The Lambda will create a new SNS notification, and return a '201 Created' to the user.
3. The SNS event triggers the 'Subreddit Article Loader' Lambda, which will look up the top 100 posts in the given subreddit. It will then enqueue an SQS message for each post. 
4. A Scheduled Lambda ('Article Download Balancer') will periodically check for new work in the SQS queue. When new messages arrive, it will invoke Download Worker Lambdas, handing out one piece of work (i.e a reddit post to process) to ech worker. 
5. Each Worker Lambda will download the pictures linked by the reddit post they were invoked to process, and persist
the files to S3. 

Create the _HTTP API Endpoint_ Lambda and the SNS topic
-----------------------------------------------------

h3: what is SNS and why are we using it? 
- event driven, triggers lambdas
- good for decoupling lambdas
- lambdas can retry 

lambda: 
- pip install boto3 <-- do we need to do this, or do the EC2s come with some libraries installed?
- gets the url from the request
- logs
- adds to SNS
- returns 201 Created

Deploy and test the Lambda
--------------------------

1. Package the lambda + libs in a zip
1. Deploy lambda using UI
2. Hit the lambda with curl (but mention Insomnia), get the error
2. Set up the rights
3. Hit the lambda, get 201 back
4. Show the logs
5. Show the event in SNS


Create the _Subreddit Article Loader_ Lambda
--------------------------------------------
1. Link to getting a key for reddit dev access
2. pip install praw
3. get the link out of SNS event & log
4. PRAW get top links, log, filter out anything that's not imgur or reddit images
5. send each link to sqs

- Why SQS not SNS???



..last(ish):
Create CloudFormation to deploy the whole stack