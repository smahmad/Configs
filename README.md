# AWS Single Sign-on to Integrate with Azure Active Directory

AWS Single Sign-On (AWS SSO) is where you create, or connect, your workforce identities in AWS once and manage access centrally across your AWS organization. We can create user identities directly in AWS SSO, or you can bring them from your Microsoft Active Directory or a standards-based identity provider, such as Okta Universal Directory or Azure AD. 

Following are the instructions to integrate AWS Single Sign-on with Azure Active Directory (Azure AD).

## **Setting up Azuru AD as identity source**

Following are the steps to configure up the Azure active directory as Identity source in AWS Single Sign-on.

1) Go to AWS SSO console, select settings. choose **Change Identity Source** actions form Actions list for under **Identity Source** section 

2) Select **External identity provider** option for Choose identity source Step for the source for users and groups that we want to change in Azure AD. Click on **Next**

3) For Configure external identity provider, we have two important sections, **Service provider metadata** and **Identity provider metadata**. First click on **Download metadata file** button to download to AWS SSO SAML metadata that we will be used to create the federation between the AWS SSO service and Azure AD enterprise application for AWS SSO.

4) From here goto Azure Active directory and Create and Add new Enterprise Application of type Non-gallery application. Give a name to your application, NBME AWS SSO for example.

5) Select **Single sign-on** for your created application and choose **SAML** method.

6) Once you slect SAML as Single Sign-On method you will see a configuration page. Here we will upload the fedration metadata by clicking on **Upload metadata file** button. select the metadata file that we downloaded in **step 3** to upload and **Add**. On adding file, the needed basic SAML configuration information will be filled, Entity Id and Assertion Consumer service URL. **Save** this SAML configuration.

7) On configuration Page, on stage **3-SAML Signing certificate** , click on **Download** for Federation Metadata XML file.

8) Go back to AWS SSO service console, click on **Choose file** to upload Idp SAML metadata file that we downloaded in **Step 7**. Click on **Next**.

9) On Confirm change step, Make sure that the conditions of chnaging the Identity source are read. Then type **ACCEPT** and click on **Change Identity Source**. with this step configuration of Azuru AD as identity source is completed and you see the following information is filled for identity source:

  - Identity source: External identity provider

  - Authentication method: SAML 2.0

  - User portal URL: https://nbme-dev.awsapps.com/start
  
  - Provisioning: Manual


## **Enable Automatic Provisioning**

In most circumstances we would like to do provisioning automatically, so where we make a chnage to a user and group in Azure AD. We would want to automatically propagate through AWS SSO. Following are the steps to enable Automatic provisioning from Manual.


1) On Change Identity Source section, by clicking on **Enable** in fornt of Enable Automatic Provisioning, we will have **SCIM endpoint** and **Access token** to copy and to use their values in the Azure AD console'

  - **NOTE:** Acess token will be shown once, but it can be gnerated agin, so make sure to take note of this.

2) Goto Azure Active directory. Select **Provisioning** for your created application and set **Provisioing Mode** from Manual to **Automatic**. Here we have to provide the value of **SCIM endpoint to Tenant URL** and **Access token to Service Token** and click on **Save** to configure provisioning to automatic.

3) Click on **Test connection** to confirm and verifiy the endpoint access and credentials are are authorized to enable provision. after successfull connection test click on Save to update user provisioning settings.

4) Select **Users and Groups** for our application to add users and groups and do assignment also. This step is important that can be done before or after settting provisioning to automatic.

- **NOTE:** Dependig on your application and use case you may wnat to change the user attribute mappings on provisioning page.

5) Go back to provisioning and click on **Start provisioning** which will start the cycle. if there is any issue in this provisiong, we can check provisioning details to chcek steady state achived and provisioning intervale, and technical information and also we can check out provisoning logs here. Initial cycle wil take almost 30 seconds to complete.

## **Testing**

1) Go back to AWS SSO console and into users and check that the users and groups that we assigned to the enterprise application in Azure AD has been assigned and brought over via SCIM automaticall from Azure AD successfully.

2) Go to AWS Accounts and select any account and click on Assigne Users, The users and groups should be available where we can assign to account, assigne existing permission sets to users and groups or create new permission sets here to assign

- **NOTE:** we can assign grouops to accounts which is abiuosly the best proactice over assigning individual users.

3) Open up the User portal URL, it will got to login.microsoft.com which is our external identity provider that is configured in AWS SSO. Use username and the password of the user assigned to enterprise application, it will go to AWS SSO sign in page, where you can see the accounts which are assigned to the user with pwermissions.
