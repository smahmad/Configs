Using AWS Budget's alerts in the member accounts to move the AWS account to an OU with restrictive SCP 

2022-04-05   
  

## Prerequisites ##

a. Add resource based policy

b. Add Tags in the account

c. SES must be activated in production instead of sandbox

## a. Add resource based policy ##

1- Navigate into eventbridge from aws console.

2- Select default event bus
	
3- In permissions 
	
```
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "mgmt-bus-stmt",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "events:PutEvents",
    "Resource": "arn:aws:events:<region>:<valid account id>:event-bus/default",
    "Condition": {
      "StringEquals": {
        "aws:PrincipalOrgID": <"o-srsgnvvcqo">
      }
    }
  }]
}
```
* Note: replace region, valid account id and principleOrgID in above policy 

## b. Add tags in user accounts ##

1- Navigate into AWS Organizations from aws console.

2- Select AWS accounts.

3- Click "organization unit" then Select "user account".

4- In Tags Click "Manage tags".

5- Add following tags:

   | Key           |Value                                        |
| ------------------- | ---------------------------------- |
| BillingContact | < nshafiq@enquizit.com >                        |
| AccountExclusion | True/False                        |
| BudgetedAmount | 100                        |
| AlertThresholds | 80:100                        |

7- Click "Save changes".


## Package Contents ##
 
a. Cross-account-role.yml

b. bucket_with_policy.yml 

c. budget_control.yml 

d. budget_control_member_acct.yml

e. Readme.md   


## a. Cross-account-role.yml ##

Deploy the following stack in Management account, this will create an cross account in management account that will be used by the lambda funtion in member accounts when call aws organization API  .

1. Select "Create stack" and choose the option "With new resources (standard)".

2. Choose "Template is Ready" and "Upload Template".

3. Click "Choose file" and upload the file "bucket_with_policy.yml". Click "Next".

4. Click "Next" and specify the following parmeters.

   | Parameter           | Description                        | Default Values                                        |
   | ------------------- | ---------------------------------- | ----------------------------------------------------- |
   | pAssumeRoleName | Bucket name                        | -                                                     |
   | pExternalAccountArnList| ARNs for the root account to deply | arn:aws:iam::<Acctid>:root,arn:aws:iam::<Acctid>:root |

5. Click "Next".

6. Click "Create Stack".

Output: A Assume Role in management account is created for lambda function  



## b. bucket_with_policy.yml ##


Deploy the following stack in you master account, this will create an S3 bucket & bucket policy in the master account that will be used by the lambda in when run cli command is being used to make cloudformation package .

1. Select "Create stack" and choose the option "With new resources (standard)".

2. Choose "Template is Ready" and "Upload Template".

3. Click "Choose file" and upload the file "bucket_with_policy.yml". Click "Next".

4. Click "Next" and specify the following parmeters.

   | Parameter           | Description                        | Default Values                                        |
   | ------------------- | ---------------------------------- | ----------------------------------------------------- |
   | BucketNameForLambda | Bucket name                        | -                                                     |
   | ARNlist             | ARNs for the root account to deply | arn:aws:iam::<Acctid>:root,arn:aws:iam::<Acctid>:root |

5. Click "Next".

6. Click "Create Stack".

Output: A S3 bucket in master account is created for lambda function  
  

## c. budget_control.yml ##
  

Go to the terminal i.e VSC(visual studio code)

1- Open the terminal.

2- Navigate into the cloudformation folder, where the yml file is present.

3- Run the following cloudformation packaged command with Bucket name:

```
aws cloudformation package --template-file budget_control.yml  --s3-bucket <Bucket Name>  --output-template-file budget_control_packaged_template.yml
```

```<Bucket Name>``` Bucket name created in above step (b)

Output: A budget_control_packaged_template.yml is being created in the same folder.

The following stack will create an lambda function and event bridge rule.

1. Select "Create stack" and choose the option "With new resources (standard)".

2. Choose "Template is Ready" and "Upload Template".

3. Click "Choose file" and upload the file "budget_control_packaged_template.yml". Click "Next".

4. Click "Next" and specify the following parameters.

   | Parameter              | Description                                  | Default Values                   |
   | ---------------------- | -------------------------------------------- | -------------------------------- |
   | pQuarantineOUName          | Name of the Quarantine OU         | Quarantine-OU  |
   | pLambdaTiggerSchedular | Cron Job to schedule the lambda trigger time | cron(0 6 1 * ? *)                |
   | pSenderEmail           | SES Verified Email Address                   | Verified Email Address           |
   | pOrganizationRoot       | Root Organization Id                   | < r-n6tb >           |

5. Click "Next".

6. Click "Create Stack".

Output: A Event bridge rule and lambda function is created  
  
	

## d. budget_control_member_acct.yml ##


Go to the terminal i.e VSC(visual studio code)

1- Open the terminal.

2- Navigate into the cloudformation folder, where the yml file is present.

3- Run the following cloudformation packaged command with Bucket name:

```
aws cloudformation package --template-file budget_control_member_acct.yml  --s3-bucket <Bucket Name>  --output-template-file budget_control_member_acct_packaged_template.yml
```

```<Bucket Name>```     Bucket name created in above step (b)

Output: A budget_control_member_acct_packaged_template.yml is being created in the same folder.

1. Select "Create stack set" and choose the Permissions for your stackset. Select "Self-service permissions" if you want to deploy in individual accounts.

2. Choose "Template is Ready" and "Upload Template".

3. Click "Choose file" and upload the file "budget_control_member_acct_packaged_template.yml". Click "Next".

4. Click "Next and specify the following parameters".

   | Parameter                      | Description                                                  | Default Values   |
   | ------------------------------ | ------------------------------------------------------------ | ---------------- |
   | ManagementAccount              | Management AWS Account ID for configuring the event bus ARN in the member accounts | 123456789012     |
   | pOrgAccountAdminEmail          | Email of the Organization Account Admin                      | -                |
   | pOrgAccountMasterEmail         | Email of the Organization Account Admin                      | -                |
   | pBudgetName                    | Budget Name                                                  | EQ-BUDGET        |
   | pAssumeRoleArn                    | Assume Role Arn created in step .a Name                                                  | < arn:aws:iam::188106553370:roleOrganozation-access-assume-role >        |

5. Click Next and Configure Stack set options such as Tags .

6. Click Next and set deployment options. you can either select deploy stack in accounts or Deploy stacks in Organization units
   Click  Next, review the details and then click "Submit".

Output: A Budget in child accounts, lambda, sns topic will be created
