# Configs

# Deployment steps


1. Deploy "Lambda-bucket.yml" inside "cloudformation-for-lambda-bucket" folder into master account. This template will create following resources

**s3 bucket**   s3 bucket in master account where this package will be deployed  - e.g. lambda-bucket.

## Bucket policy    
   This template will create bucket policy for the created bucket to grant access for multiaccount to Lambda code and deploy it through Stack set.

## Parameters in Template:

|Parameter           |Description                                                                           |Allowed values |
|--------------------|--------------------------------------------------------------------------------------|---------------|
|BucketNameForLambda | Name of the Bucket to create for Lambda to deploy in it - e.g. lambda-bucket         | lower, 3 min and 63 max |
