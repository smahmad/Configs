

# Automate the life cycle of default CloudWatch log in multiple accounts

This section covers an introductory note about CloudWatch and deployment steps.

## Amazon CloudWatch Logs

Amazon CloudWatch Logs to monitor, store, and access your log files from Amazon Elastic Compute Cloud (Amazon EC2) instances, AWS CloudTrail, Route 53, and other sources.

CloudWatch Logs enables you to centralize the logs from all of your systems, applications, and AWS services that you use, in a single, highly scalable service. You can then easily view them, search them for specific error codes or patterns, filter them based on specific fields, or archive them securely for future analysis.

**Log events**

A log event is a record of some activity recorded by the application or resource being monitored. The log event record that CloudWatch Logs understands contains two properties: the timestamp of when the event occurred, and the raw event message. Event messages must be UTF-8 encoded.

**Log streams**

A log stream is a sequence of log events that share the same source. More specifically, a log stream is generally intended to represent the sequence of events coming from the application instance or resource being monitored. For example, a log stream may be associated with an Apache access log on a specific host. When you no longer need a log stream, you can delete it using the [aws logs delete-log-stream](https://docs.aws.amazon.com/cli/latest/reference/logs/delete-log-stream.html) command.

**Log groups**

Log groups define groups of log streams that share the same retention, monitoring, and access control settings. Each log stream has to belong to one log group. For example, if you have a separate log stream for the Apache access logs from each host, you could group those log streams into a single log group called `MyWebsite.com/Apache/access_log`.

There is no limit on the number of log streams that can belong to one log group.

**Metric filters**

You can use metric filters to extract metric observations from ingested events and transform them to data points in a CloudWatch metric. Metric filters are assigned to log groups, and all of the filters assigned to a log group are applied to their log streams.

**Retention settings**

Retention settings can be used to specify how long log events are kept in CloudWatch Logs. Expired log events get deleted automatically. Just like metric filters, retention settings are also assigned to log groups, and the retention assigned to a log group is applied to their log streams.

**Log Retention** – By default, logs are kept indefinitely and never expire. You can adjust the retention policy for each log group, keeping the indefinite retention, or choosing a retention period between 10 years and one day.



**AWS Services Used In Solution**

- AWS CloudWatch - Logs to be exported.

- AWS Lambda - to run the codes (the version of the python code developed is 3.7)

- AWS S3 - to keep Logs Backup.

- AWS CloudFormation Template - to automate the solution.

- AWS SNS

  

**Solution Design**





![Solution Diagram](https://user-images.githubusercontent.com/60149354/109172443-98648b00-77a4-11eb-943b-2d1766d799c3.png)

## Package Contents

a. Cloudformation-for-lambda-bucket\Lambda-bucket.yml

b. CloudformationTemplate\cloudwtch_logs_exporting_s3_bucket.yml

c. Lambda\cloudwtch_logs_exporting_s3_bucket\index.py

## Deployment steps


1. Deploy "Lambda-bucket.yml" inside "cloudformation-for-lambda-bucket" folder into master account. This template will create following resources
**s3 bucket**   s3 bucket in master account where this package will be deployed  - e.g. lambda-bucket.

## Bucket policy    
   This template will create bucket policy for the created bucket to grant access for multiaccount to Lambda code and deploy it through Stack set.

## Parameters in Template:

|Parameter           |Description                                                                           |Allowed values |
|--------------------|--------------------------------------------------------------------------------------|---------------|
|BucketNameForLambda | Name of the Bucket to create for Lambda to deploy in it - e.g. lambda-bucket         | lower, 3 min and 63 max |
