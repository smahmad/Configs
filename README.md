# AWS Single Sign-on to Integrate with Azure Active Directory

AWS Single Sign-On (AWS SSO) is where you create, or connect, your workforce identities in AWS once and manage access centrally across your AWS organization. We can create user identities directly in AWS SSO, or you can bring them from your Microsoft Active Directory or a standards-based identity provider, such as Okta Universal Directory or Azure AD. 

Following are the instructions to integrate AWS Single Sign-on with Azure Active Directory (Azure AD).

## **Setting up Azuru AD as identity source**

Following are the steps to configure up the Azure active directory as Identity source in AWS Single Sign-on.

1) Go to AWS SSO console, select settings. choose Change Identity Source actions form Actions list for under Identity Source section 

2) Select External identity provider option for Choose identity source Step for the source for users and groups that we want to change fro Azure AD. Click on Next

3) For Configure external identity provider, we have two important sections, Service provider metadata and Identity provider metadata. First click on Download metadata file button to download to AWS SSO SAML metadata that we will be used to create the federation between the AWS SSO service and Azure AD enterprise application for AWS SSO.

4) From here goto Azure Active directory and Create and Add new Enterprise Application of type Non-gallery application. Give a name to your application, NBME AWS SSO for example.

5) Select Single sign-on for your created application and choose SAML method.

6) Once you slect SAML as Single Sign-On method you will see a configuration page. Here we will upload the fedration metadata by clicking on Upload metadata file button. select the metadta file that we downloaded in step 3 to upload and Add. On adding file, the needed basic SAML configuration information will be filled, Entity Id and Assertion Consumer service URL. Save this SAML configuration.

7) On configuration Page, on stage 3-SAML Signing certificate, click on Download for Federation Metadata XML file.

8) Go back to AWS SSO service console, click on Choose file to upload Idp SAML metadata file that we downloaded in Step 7. Click on Next.

9) On Confirm change step, Make sure that the conditions of chnaging the Identity source are read. Then type ACCEPT and click on Change Identity Source. with this step  configuration of Azuru AD as identity source is completed and you see the following information is filled for identity source:

  - Identity source: External identity provider

  - Authentication method: SAML 2.0

  - User portal URL: https://nbme-dev.awsapps.com/start
  
  - Provisioning: Manual


## **Enable Automatic Provisioning**

In most circumstances we would like to do provisioning automatically, so where we make a chnage to a user and group in Azure AD. We would want to automatically propagate through AWS SSO. Following are the steps to enable Automatic provisioning from Manual.


1) On Change Identity Source section, by clicking on Enable in fornt of Enable Automatic Provisioning, we will have SCIM endpoint and Access token to copy and to use their values in the Azure AD console'

  - **NOTE:** Acess token will be shown once, but it can be gnerated agin, so make sure to take note of this.

2) Goto Azure Active directory. Select Provisioning for your created application and set Provisioing Mode from Manual to Automatic. Here we have to provide the value of SCIM endpoint to Tenant URL and Access token to Service Token and click on Save to configure provisioning to automatic.

3) Clcik on test connection to confirm and verifiy the endpoint access and credentials are are authorized to enable provision

## Amazon Inspector

Amazon Inspector tests the network accessibility of your Amazon EC2 instances and the security state of your applications that run on those instances. Amazon Inspector assesses applications for exposure, vulnerabilities, and deviations from best practices. After performing an assessment, Amazon Inspector produces a detailed list of security findings that is organized by level of severity.



**Following are the key components:** 



**Amazon Inspector agent**

A software agent that you can install on the EC2 instances that are included in the assessment target. The agent collects a wide set of configuration data. 
https://docs.aws.amazon.com/inspector/latest/userguide/inspector_supported_os_regions.html

**Assessment target**

A collection of AWS resources that work together as a unit to help you accomplish your business goals. Amazon Inspector evaluates the security state of the resources that constitute the assessment target.

**Note**

To create an Amazon Inspector assessment target, you must first tag your EC2 instances with key-value pairs of your choice. Next, you can create a view of these tagged EC2 instances that have common keys or common values.

**Assessment template**

A configuration that is used during your assessment run. The template includes the following:

    - Rules packages that Amazon Inspector uses to evaluate your assessment target

    - Amazon SNS topics that you want Amazon Inspector to send notifications to about assessment run states and findings

    - Tags (key-value pairs) that you can assign to findings that are generated by the assessment run

    - The duration of the assessment run

**Assessment run**

  - During an assessment run, Amazon Inspector monitors, collects, and analyzes configuration data (telemetry) from resources within the specified target. Next, Amazon Inspector     analyzes the data and compares it against a set of security rules packages that are specified in the assessment template used during the assessment run. 

  - A completed assessment run produces a list of findings, which are potential security issues of various levels of severity. 

**Finding**

A potential security issue that Amazon Inspector discovers during an assessment run of the specified target. Findings are displayed in the Amazon Inspector console or retrieved through the API. They contain both a detailed description of the security issue and a recommendation on how to fix it.

**Rule**

In the context of Amazon Inspector, a security check performed during an assessment run. When a rule detects a potential security issue, Amazon Inspector generates a finding that describes the issue.

**Rules package**

A rules package corresponds to a security goal that you might have. You can specify your security goal by selecting the appropriate rules package when you create an Amazon Inspector assessment template.

## Deployment



1. In the master account, an AWS CloudFormation stack set is used to create the resource group, assessment target, configure the assessment template and run that assessment template (through lambda function which include the Crhelper Module ).

2. AWS Security Hub Multiaccount Scripts (These scripts automate the process of enabling and disabling AWS Security Hub simultaneously across a group of AWS accounts that are in your control. (Note, that you can have one master account and up to a 1000 member accounts) 

   


| ***SrNo.\*** | ***File\***              | ***Description\***                                           | ***Location\*** |
| ------------ | ------------------------ | ------------------------------------------------------------ | --------------- |
| 1            | inspector_deployment.yml | Create resource group, assessment target, configure the assessment template and run that assessment template. | AWS-Inspector   |

   

## **Deployment steps**


   - ***s3 bucket:***   s3 bucket in master account where this package will be deployed  - e.g. inspector-lambda-code.
   
   - ***Bucket policy:***   This template will create bucket policy for the created bucket to grant access for multiaccount to Lambda code and deploy it through Stack set.
   
**Parameters in Template:**

|Parameter           |Description                                                                           |Allowed values |
|--------------------|--------------------------------------------------------------------------------------|---------------|
|BucketNameForLambda |Name of the Bucket to create for Lambda to deploy in it - e.g. lambda-bucket         |[Valid S3 Bucket Name](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html)|
|ARNlist |Comma delimited list of ARNs for the root accounts to deploy Lambda - e.g. arn:aws:iam::999999999999:root,arn:aws:iam::999999999999:root        |[Valid AWS Account Id](https://docs.aws.amazon.com/general/latest/gr/acct-identifiers.html)|

#### Package and Upload the artifacts

The aws cloudformation package does follow actions:
   - ZIPs up the local files ( lamda ).
   - Uploads them to a designated (lambda-bucket) S3 bucket.
   - Generates a new template where the local paths are replaced with the S3 URIs.

**Run the package command:**

Navigate into the path "AWS-Inspector" and use cloudformation packaged command given below:

```
aws cloudformation package --template-file </path_to_template/template.yml> --s3-bucket bucket-name --output-template-file <packaged-template.yml>
```

e.g.

```
aws cloudformation package --template-file inspector_deployment.yml --s3-bucket inspector-lambda-code --output-template-file inspector_deployment_packaged.yml
```

#### Deploying the “packaged” template

1. To deploy StackSet, select the inspector_deployment_packaged.yml in master account to deploy StackSets in the Target Accounts. It will create  resources i.e   resource group, assessment target, configure the assessment template and run that assessment template.

2. Give meaningful name to StackSet

3. Specify stack parameters as you would a single CloudFormation template. These parameters will be applied in each Target Account.

   **Parameters in Template**

   | Parameter               | Description                                   | Allowed values/Default |
   | ----------------------- | --------------------------------------------- | ---------------------- |
   | pAssessmentTemplateName | The name of the Assessment Template           | Eq-AssessmentTemplate  |
   | pAssessmentTargetName   | The name of the Assessment Target             | Eq-AssessmentTarget    |
   | pResourceGroupTagsKey   | The Value name of the tag of the resource     | Patch Group            |
   | pResourceGroupTagsValue | The key name of the tag of the resource       | Dev                    |
   | pDurationInSeconds      | The duration of the assessment run in seconds | 180                    |

## Amazon Security Hub


GuardDuty and Security Hub is enabled in all AWS Accounts within the AWS Organizations and for future accounts added under Organization. So following  is just for reference, if security hub is not auto enable for all accounts with in organization or you want to enable security hub not only with in organisation, also for across account.

**AWS Security Hub Multiaccount Scripts**

https://github.com/awslabs/aws-securityhub-multiaccount-scripts

**Security hub reports filtering**

When you display a list of findings from the Findings page, the Integrations page, or the Insights page, the list is always filtered based on the record state and workflow status. 

- The record state indicates whether the finding is active or archived.
- The workflow status indicates the status of the investigation into the finding.
- The severity of the finding.
  - The finding must have either Label or Normalized populated.
  - If only one of these attributes is populated, then Security Hub automatically populates the other one. 
  - If neither attribute is populated, then the finding is invalid. Label is the preferred attribute.
    - Label
      The severity value of the finding. The allowed values are the following.

      INFORMATIONAL - No issue was found.

      LOW - The issue does not require action on its own.

      MEDIUM - The issue must be addressed but not urgently.

      HIGH - The issue must be addressed as a priority.

      CRITICAL - The issue must be remediated immediately to avoid it escalating.

    - If you provide Normalized and do not provide Label, then Label is set automatically as follows.

      0 - INFORMATIONAL

      1–39 - LOW

      40–69 - MEDIUM

      70–89 - HIGH

      90–100 - CRITICAL
      
    - Normalized
      
      Deprecated. The normalized severity of a finding. This attribute is being deprecated. Instead of providing Normalized, provide Label.
      
  - To add a filter to the finding list
    - Open the AWS Security Hub console at https://console.aws.amazon.com/securityhub/.

      To display a finding list, do one of the following:

       1.In the Security Hub navigation pane, choose Findings.

       2.In the Security Hub navigation pane, choose Insights. Choose an insight. Then on the results list, choose an insight result.

       3.In the Security Hub navigation pane, choose Integrations. Choose See findings for an integration.

      *Choose the Add filters box.*

       4.In the menu, under Filters, choose a filter.

       5.Choose the filter match type.

      *For a string filter, you can choose from the following comparison options:*

       6.is – Find a value that exactly matches the filter value.

       7.starts with – Find a value that starts with the filter value.

       8.is not – Find a value that does not match the filter value.

       9.does not start with – Find a value that does not start with the filter value.

      10.For a numeric filter, you can choose whether to provide a single number (Simple) or a range of numbers (Range).

      11.For a date or time filter, you can choose whether to provide a length of time from the current date time (Rolling window) or a date range (Fixed range).

      *Adding multiple filters has the following interactions:*

      13.is and starts with filters are joined by OR. A value matches if it contains any of the filter values. For example, if you specify Severity label is CRITICAL and                 Severity label is HIGH, the results include both critical and high severity findings.

      14.is not and does not start with filters are joined by AND. A value matches only if it does not contain any of those filter values. For example, if you specify Severity           label is not LOW and Severity label is not MEDIUM, the results do not include either low or medium severity findings.

      15.If you have an is filter on a field, you cannot have a is not or a does not start with filter on the same field.

      16.Specify the filter value.

      *Note that for string filters, the filter value is case sensitive.*

      For example, for findings from Security Hub, Product name is Security Hub. If you use the EQUALS operator to see findings from Security Hub, you must enter Security Hub as       the filter value. If you enter security hub, no findings are displayed.

       17.Similarly, if you use the PREFIX operator, and enter Sec, Security Hub findings are displayed. If you enter sec, no Security Hub findings are displayed.

       18.Choose Apply.
       
    - Grouping findings
      - you can group the findings based on the values of a selected attribute.
      - For example, if you group the findings by AWS account ID, you see a list of account identifiers, with the number of matching findings for each account.
      
      *To group the findings in a findings list*
      
       1.On the finding list, choose the Add filters box.
       
       2.In the menu, under Grouping, choose Group by.
       
       3.In the list, choose the attribute to use for the grouping.
       
       4.Choose Apply.
 

## Amazon Inspector service limitations

Currently, your assessment targets can consist only of EC2 instances.



The following are Amazon Inspector limits per AWS account per region:

| Resource                         | Default Limit | Comments                                                     |
| :------------------------------- | :------------ | :----------------------------------------------------------- |
| Instances in running assessments | 500           | The maximum number of EC2 instances that can be included across all running assessments per account per region. |
| Assessment runs                  | 50000         | The maximum number of assessment runs that you can create per account per region. You can have multiple assessment runs happening at the same time as long as the assessment targets used for these runs do not contain overlapping EC2 instances. |
| Assessment Templates             | 500           | The maximum number of assessment templates that you can have at any given time per account per region. |
| Assessment Targets               | 50            | The maximum number of assessment targets that you can have at any given time per account per region. |
