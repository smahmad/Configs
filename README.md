# AWS Orgnization Unit Service Control policy implementation
2020-04-07
Enquizit.inc

## Summary
This package allows for deploying, managing and attaching your Service Control Policies from a single location. Simply update the CloudFormation template (CFT) to add/remove/update SCPs. Note: Each SCP added after the first SCP in the CFT after the 1st must have the DependsOn attribute with the name of the SCP preceeding it (this prevents errors due to overlapping API calls).

## Package Contents

1. cft/eq-organization-scp.yml
2. Configuration/buidspec-validate-cft.yml
3. deployment-targets.txt
4. parameters-stackset.txt

NOTE: If the user want to deploy the this scp template as a self managed stackset then there is no need for the deployment-targets.txt and if the user want to deploy it as service managed then deployment-targets.txt must contain target ID (it can be root or OU ID). 



## Deployment steps without pipeline 
1. Navigate to CloudFormation  
    a. Click "Create Stack" -> "With new resource (standard)"  
    b. Under Prerequisite select "Template is ready"  
    c. Under Specify Template select "Upload a template file" and choose `eq-organization-scp.yml`

2. Enter and Stack name and specify the following parameters:
    | Parameter             | Description                           |  Values                |
    |-----------------------|---------------------------------------|-------------------------------| 
    | ResourcePrefix | Prefix for resources created. | Comma Delimited List Of OUs |
    | DDRWSTVOUs | SCP to Disallow the deletion of resources with specific tag values | Comma Delimited List Of OUs |
    | RTNaOUs | SCP to "Disallow Creation Of Resources Outside Of North America | Comma Delimited List Of OUs |
    | DCUOUs | SCP to "Disallow Create New User Action. | Comma Delimited List Of OUs |
    | DSSOUs | SCP to "Disallow The Use Of Specified Services | Comma Delimited List Of OUs |
    | DCNROUs | SCP to "Disallow The Creation Of Network Resources | Comma Delimited List Of OUs |
    | EEVEOUs | SCP to "Enforce The Encryption Of EBS Volume | Comma Delimited List Of OUs |

3. Run the cloudformation template and it will create all the manadatory polices  
4. To update the policy, update the cloudformation template file `eq-organization-scp.yml` and execute changeset to update existing policies


**NOTE:** The naming convention for the parameter takes the first letter of every word used in the description of the SCP and (OUs) is appended the end (i.e) (Disallow the deletion of resources with specific tag values) (DDRWSTVOUs)

7. To delete the policy, delete the cloudformation template and it will delete all the polices
