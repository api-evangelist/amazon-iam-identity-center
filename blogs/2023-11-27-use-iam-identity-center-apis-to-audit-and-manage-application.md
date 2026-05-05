---
title: "Use IAM Identity Center APIs to audit and manage application assignments"
url: "https://aws.amazon.com/blogs/security/use-iam-identity-center-apis-to-audit-and-manage-application-assignments/"
date: "Mon, 27 Nov 2023 18:17:19 +0000"
author: "Laura Reith"
feed_url: "https://aws.amazon.com/blogs/security/tag/aws-iam-identity-center/feed/"
---
<p>You can now use <a href="https://aws.amazon.com/iam/identity-center/" rel="noopener" target="_blank">AWS IAM Identity Center</a> application assignment APIs to programmatically manage and audit user and group access to <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/awsapps.html" rel="noopener" target="_blank">AWS managed applications</a>. Previously, you had to use the IAM Identity Center console to&nbsp;manually assign users and groups to an application. Now, you can automate this task so that you scale more effectively as your organization grows.</p> 
<p>In this post, we will show you how to use IAM Identity Center APIs to programmatically manage and audit user and group access to applications. The procedures that we share apply to both organization instances and account instances of IAM Identity Center.</p> 
<h2>Automate management of user and group assignment to&nbsp;applications</h2> 
<p>IAM Identity Center is where you create, or connect, your workforce users one time and centrally manage their access to multiple AWS accounts and applications. You configure <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/awsapps.html" rel="noopener" target="_blank">AWS managed applications</a> to work with IAM Identity Center directly from within the relevant application console, and then manage which users or groups need permissions to the application.</p> 
<p>You can already use the <a href="https://aws.amazon.com/blogs/security/use-new-account-assignment-apis-for-aws-sso-to-automate-multi-account-access/" rel="noopener" target="_blank">account assignment APIs to automate multi-account access and audit access assigned to your users</a> using IAM Identity Center <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/permissionsetsconcept.html" rel="noopener" target="_blank">permission sets</a>. Today, we expanded this capability with the new <a href="https://aws.amazon.com/about-aws/whats-new/2023/11/aws-iam-identity-center-apis-automate-access-applications/" rel="noopener" target="_blank">application assignment APIs</a>. You can use these new APIs to programmatically control application assignments and develop automated workflows for auditing them.</p> 
<p>AWS managed applications access user and group information directly from IAM Identity Center. One example of an AWS managed application is <a href="https://aws.amazon.com/redshift/" rel="noopener" target="_blank">Amazon Redshift</a>. When you configure Amazon Redshift as an AWS managed application with IAM Identity Center, and a user from your organization accesses the database, their group memberships defined in IAM Identity Center can map to Amazon Redshift database roles that grant them specific permissions. This makes it simpler for you to manage users because you don’t have to set database-object permissions for each individual. For more information, see <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/redshift-iam-access-control-idp-connect.html#redshift-iam-access-control-idp-connect-benefits" rel="noopener" target="_blank">The benefits of Redshift integration with AWS IAM Identity Center</a>.</p> 
<p>After you configure the integration between IAM Identity Center and Amazon Redshift, you can automate the assignment or removal of users and groups by using the <span style="font-family: courier;">DeleteApplicationAssignment</span> and <span style="font-family: courier;">CreateApplicationAssignment</span> APIs, as shown in Figure 1.</p> 
<div class="wp-caption aligncenter" id="attachment_32473" style="width: 790px;">
 <img alt="Figure 1: Use the CreateApplicationAssignment API to assign users and groups to Amazon Redshift" class="size-full wp-image-32473" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/11/27/img1-14.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-32473">Figure 1: Use the CreateApplicationAssignment API to assign users and groups to Amazon Redshift</p>
</div> 
<p>In this section, you will learn how to use <a href="https://docs.aws.amazon.com/singlesignon/latest/APIReference/welcome.html" rel="noopener" target="_blank">Identity Center APIs</a> to assign a group to your Amazon Redshift application. You will also learn how to delete the group assignment.</p> 
<h3>Prerequisites</h3> 
<p>To follow along with this walkthrough, make sure that you’ve completed the following prerequisites:</p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/get-started-enable-identity-center.html" rel="noopener" target="_blank">Enable IAM Identity Center</a>, and use the <a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/welcome.html" rel="noopener" target="_blank">Identity Store</a> to manage your identity data.&nbsp;If you use an <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/manage-your-identity-source-idp.html" rel="noopener" target="_blank">external identity provider</a>, then you should handle the user creation and deletion processes in those systems.</li> 
 <li>Configure Amazon Redshift to use IAM Identity Center as its identity source. When you configure Amazon Redshift to use IAM Identity Center as its identity source, the application requires explicit assignment by default. This means that you must explicitly assign users to the application in the Identity Center console or APIs.</li> 
 <li>Install and configure <a href="https://aws.amazon.com/cli/" rel="noopener" target="_blank">AWS Command Line Interface (AWS CLI)</a> version 2. For this example, you will use AWS CLI v2 to call the IAM Identity Center application assignment APIs. For more information, see&nbsp;<a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html" rel="noopener" target="_blank">Installing the AWS CLI</a> and <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html" rel="noopener" target="_blank">Configuring the AWS CLI</a>.</li> 
</ul> 
<h3>Step 1: Get your Identity Center instance information</h3> 
<p>The first step is to run the following command to get the Amazon Resource Name (ARN) and Identity Store ID for the instance that you’re working with:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">aws sso-admin list-instances</code></pre> 
</div> 
<p>The output should look similar to the following:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
  "Instances": [
      {
          "InstanceArn": "arn:aws:sso:::instance/ssoins-****************",
          "IdentityStoreId": "d-**********",
          "OwnerAccountId": "************",
          "Name": "MyInstanceName",
          "CreatedDate": "2023-10-08T16:45:19.839000-04:00",
          "State": {
              "Name": "ACTIVE"
          },
          "Status": "ACTIVE"
      }
  ],
  "NextToken": &lt;&lt;TOKEN&gt;&gt;
}</code></pre> 
</div> 
<p>Take note of the&nbsp;<span style="font-family: courier;">IdentityStoreId</span> and the&nbsp;<span style="font-family: courier;">InstanceArn</span> — you will use both in the following steps.</p> 
<h3 id="step_2">Step 2: Create user and group in your Identity Store</h3> 
<p>The next step is to create a user and group in your Identity Store.</p> 
<blockquote>
 <p><strong>Note:</strong> If you already have a group in your Identity Center instance, get its GroupId and then proceed to <a href="#Step_3">Step 3</a>. To get your GroupId, run the following command: </p> 
 <div class="hide-language"> 
  <pre class="unlimited-height-code"><code class="lang-text">aws identitystore get-group-id --identity-store-id “d-********” –alternate-identifier “GroupName” ,</code></pre> 
 </div> 
</blockquote> 
<p>Create a new user by using the&nbsp;<span style="font-family: courier;">IdentityStoreId</span> that you noted in the previous step.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">aws identitystore create-user --identity-store-id "d-**********" --user-name "MyUser" --emails Value="MyUser@example.com",Type="Work",Primary=true —display-name "My User" —name FamilyName="User",GivenName="My" </code></pre> 
</div> 
<p>The output should look similar to the following:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
    "UserId": "********-****-****-****-************",
    "IdentityStoreId": "d--********** "
}</code></pre> 
</div> 
<p>Create a group in your Identity Store:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">aws identitystore create-group --identity-store-id d-********** --display-name engineering</code></pre> 
</div> 
<p>In the output, make note of the <span style="font-family: courier;">GroupId</span> — you will need it later when you create the application assignment in <a href="#step_4">Step 4</a>:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
    "GroupId": "********-****-****-****-************",
    "IdentityStoreId": "d-**********"
}</code></pre> 
</div> 
<p>Run the following command to add the user to the group:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">aws identitystore create-group-membership --identity-store-id d-********** --group-id ********-****-****-****-************ --member-id UserId=********-****-****-****-************</code></pre> 
</div> 
<p>The result will look similar to the following:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
    "MembershipId": "********-****-****-****-************",
    "IdentityStoreId": "d-**********"
}</code></pre> 
</div> 
<h3 id="Step_3">Step 3: Get your Amazon Redshift application ARN instance</h3> 
<p>The next step is to determine the application ARN. To get the ARN, run the following command.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">aws sso-admin list-applications --instance-arn "arn:aws:sso:::instance/ssoins-****************"</code></pre> 
</div> 
<p>If you have more than one application in your environment, use the filter flag to specify the application account or the application provider. To learn more about the filter option, see the <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/sso-admin/paginator/ListApplications.html" rel="noopener" target="_blank">ListApplications API documentation</a>.</p> 
<p>In this case, we have only one application: Amazon Redshift.&nbsp;The response should look similar to the following. Take note of the&nbsp;<span style="font-family: courier;">ApplicationArn</span> — you will need it in the next step.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{

    "ApplicationArn": "arn:aws:sso:::instance/ssoins-****************/apl-***************",
    "ApplicationProviderArn": "arn:aws:sso::aws:applicationProvider/Redshift",
    "Name": "Amazon Redshift",
    "InstanceArn": "arn:aws:sso:::instance/ssoins-****************",
    "Status": "DISABLED",
    "PortalOptions": {
        "Visible": true,
        "Visibility": "ENABLED",
        "SignInOptions": {
            "Origin": "IDENTITY_CENTER"
        }
    },
    "AssignmentConfig": {
        "AssignmentRequired": true
    },
    "Description": "Amazon Redshift",
    "CreatedDate": "2023-10-09T10:48:44.496000-07:00"
}</code></pre> 
</div> 
<h3 id="step_4">Step 4: Add your group to the Amazon Redshift application</h3> 
<p>Now you can add your new group to the Amazon Redshift application managed by IAM Identity Center. The&nbsp;<span style="font-family: courier;">principal-id</span> is the <span style="font-family: courier;">GroupId</span> that you created in <a href="#step_2">Step 2</a>.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">aws sso-admin create-application-assignment --application-arn "arn:aws:sso:::instance/ssoins-****************/apl-***************" --principal-id "********-****-****-****-************" --principal-type "GROUP"</code></pre> 
</div> 
<p>The group now has access to Amazon Redshift, but with the <a href="https://docs.aws.amazon.com/redshift/latest/dg/r_Privileges.html" rel="noopener" target="_blank">default permissions</a> in Amazon Redshift. To grant access to databases, you can create roles that control the permissions available on a set of tables or views.</p> 
<p>To create these roles in Amazon Redshift, you need to connect to your cluster and run SQL commands. To connect to your cluster, use one of the following options:</p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/redshift/latest/mgmt/query-editor-v2-using.html" rel="noopener" target="_blank">Connect to Amazon Redshift through the query editor version 2</a></li> 
 <li><a href="https://docs.aws.amazon.com/redshift/latest/mgmt/connecting-to-cluster.html" rel="noopener" target="_blank">Connect to Amazon Redshift through Java Database Connectivity (JDBC), Open Database Connectivity (ODBC), or Python-based tools</a></li> 
</ul> 
<p>Figure 2 shows a connection to Amazon Redshift through the query editor v2.</p> 
<div class="wp-caption aligncenter" id="attachment_32252" style="width: 790px;">
 <img alt="Figure 2: Query editor v2" class="size-full wp-image-32252" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/11/14/img2-2-scaled.jpg" width="780" />
 <p class="wp-caption-text" id="caption-attachment-32252">Figure 2: Query editor v2</p>
</div> 
<p>By default, all users have CREATE and USAGE permissions on the PUBLIC schema of a database. To disallow users from creating objects in the PUBLIC schema of a database, use the <a href="https://docs.aws.amazon.com/redshift/latest/dg/r_REVOKE.html" rel="noopener" target="_blank">REVOKE</a> command to remove that permission. For more information, see&nbsp;<a href="https://docs.aws.amazon.com/redshift/latest/dg/r_Privileges.html" rel="noopener" target="_blank">Default database user permissions</a>.</p> 
<p>As the Amazon Redshift database administrator, you can create roles where the role name contains the identity provider namespace&nbsp;prefix and the group or user name.&nbsp;To do this, use the following syntax:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">CREATE ROLE <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;identitycenternamespace:rolename&gt;</span>;</code></pre> 
</div> 
<p>The <span style="font-family: courier;">rolename</span> needs to match the group name in IAM Identity Center.&nbsp;Amazon Redshift automatically maps the IAM Identity Center group or user to the role created previously.&nbsp;To expand the permissions of a user, use the <a href="https://docs.aws.amazon.com/redshift/latest/dg/r_GRANT.html" rel="noopener" target="_blank">GRANT </a>command.</p> 
<p>The <span style="font-family: courier;">identityprovidernamespace</span> is assigned when you create <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/redshift-iam-access-control-native-idp.html#redshift-iam-access-control-native-idp-setup" rel="noopener" target="_blank">the integration between Amazon Redshift and IAM Identity Center</a>. It represents your organization’s name and is added as a prefix to your IAM Identity Center managed users and roles in the Redshift database.</p> 
<p>Your syntax should look like the following:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">CREATE ROLE <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWSIdentityCenter:MyGroup&gt;</span>;</code></pre> 
</div> 
<h3>Step 5: Remove application assignment</h3> 
<p>If you decide that the new group no longer needs access to the Amazon Redshift application but should remain within the IAM Identity Center instance, run the following command:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">aws sso-admin delete-application-assignment --application-arn "arn:aws:sso:::instance/ssoins-****************/apl-***************" --principal-id "********-****-****-****-************" --principal-type "GROUP"</code></pre> 
</div> 
<blockquote>
 <p><strong>Note:</strong> Removing an application assignment for a group doesn’t remove the group from your Identity Center instance.</p>
</blockquote> 
<p>When you remove or add user assignments, we recommend that you review the application’s documentation because you might need to take additional steps to completely onboard or offboard a given user or group. For example, when you remove a user or group assignment, you must also remove the corresponding roles in Amazon Redshift. You can do this by using the <a href="https://docs.aws.amazon.com/redshift/latest/dg/r_DROP_ROLE.html" rel="noopener" target="_blank">DROP ROLE</a> command. For more information, see <a href="https://docs.aws.amazon.com/redshift/latest/dg/r_Database_objects.html" rel="noopener" target="_blank">Managing database security</a>.</p> 
<h2>Audit user and group access to applications</h2> 
<p>Let’s consider how you can use the new APIs to help you audit application assignments. In the preceding example, you used the AWS CLI to create and delete assignments to Amazon Redshift. Now, we will show you how to use the new <span style="font-family: courier;">ListApplicationAssignments</span> API to list the groups that are currently assigned to your Amazon Redshift application.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">aws sso-admin list-application-assignments --application-arn arn:aws:sso::****************:application/ssoins-****************/apl-****************</code></pre> 
</div> 
<p>The output should look similar to the following — in this case, you have a single group assigned to the application.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
    "ApplicationAssignments": [
        {
        "ApplicationArn": "arn:aws:sso::****************:application/ssoins-****************/apl-****************",
        "PrincipalId": "********-****-****-****-************",
        "PrincipalType": "GROUP"
        }
    ]
}</code></pre> 
</div> 
<p>To see the group membership, use the&nbsp;<span style="font-family: courier;">PrincipalId</span> information to query Identity Store and get information on the users assigned to the group with a combination of the <a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/API_ListGroupMemberships.html" rel="noopener" style="font-family: courier;" target="_blank">ListGroupMemberships</a> and&nbsp;<a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/API_DescribeGroupMembership.html" rel="noopener" style="font-family: courier;" target="_blank">DescribeGroupMembership</a> APIs.</p> 
<p>If you have several applications that IAM Identity Center manages, you can also create a script to automatically audit those applications. You can run this script periodically in an <a href="https://aws.amazon.com/lambda/" rel="noopener" target="_blank">AWS Lambda</a> function in your environment to maintain oversight of the members that are added to each application.</p> 
<p>To get the script for this use case, see the&nbsp;<a href="https://github.com/aws-samples/multiple-iam-identity-center-instance-management" rel="noopener" target="_blank">multiple-instance-management-iam-identity-center</a> GitHub repository. The repository includes instructions&nbsp;to deploy the script using Lambda within the <a href="https://aws.amazon.com/organizations/" rel="noopener" target="_blank">AWS Organizations</a> delegated administrator account. After deployment, you can invoke the Lambda function to get .csv files of every IAM Identity Center instance in your organization, the applications assigned to each instance, and the users that have access to those applications.</p> 
<h2>Conclusion</h2> 
<p>In this post, you&nbsp;learned how to use the IAM Identity Center application assignment APIs to assign users to Amazon Redshift and remove them from the application when they are no longer part of the organization. You also learned to list which applications are deployed in each account, and which users are assigned to each of those applications.</p> 
<p>To learn more about IAM Identity Center, see the <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html" rel="noopener" target="_blank">AWS IAM Identity Center user guide</a>. To test the application assignment APIs, see the <a href="https://docs.aws.amazon.com/singlesignon/latest/APIReference/welcome.html" rel="noopener" target="_blank">SSO-admin API reference guide</a>.</p> 
<p>If you have feedback about this post, submit comments in the <strong>Comments</strong> section below. If you have questions about this post, start a new thread on <a href="https://repost.aws/tags/TAJNFEvp8UQUaLplKZtOsAaw/aws-iam-identity-center" rel="noopener" target="_blank">AWS IAM Identity Center re:Post</a> or <a href="https://console.aws.amazon.com/support/home" rel="noopener" target="_blank">contact AWS Support</a>.</p> 
<p><strong>Want more AWS Security news? Follow us on <a href="https://twitter.com/AWSsecurityinfo" rel="noopener noreferrer" target="_blank" title="Twitter">Twitter</a>.</strong></p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Author" class="aligncenter size-full wp-image-15426" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2020/08/23/Laura-Reith-Author.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Laura Reith</h3> 
  <p>Laura is an Identity Solutions Architect at AWS, where she thrives on helping customers overcome security and identity challenges. In her free time, she enjoys wreck diving and traveling around the world.</p> 
  <p></p>
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Steve Pascoe" class="aligncenter size-full wp-image-32250" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/11/14/pascoes.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Steve Pascoe</h3> 
  <p>Steve is a Senior Technical Product Manager with the AWS Identity team. He delights in empowering customers with creative and unique solutions to everyday problems. Outside of that, he likes to build things with his family through Lego, woodworking, and recently, 3D printing.</p> 
  <p></p>
 </div> 
 <div class="blog-author-box" style="padding: 15px;"> 
  <p class="sowjir-1.jpeg"><img alt="sowjir-1.jpeg" class="alignleft size-full wp-image-5363" height="160" src="https://d2908q01vomqb2.cloudfront.net/5b384ce32d8cdef02bc3a139d4cac0a22bb029e8/2023/10/17/sowjir-1.jpeg" width="120" /></p> 
  <h3 class="lb-h4">Sowjanya Rajavaram</h3> 
  <p>Sowjanya is a Sr Solution Architect who specializes in Identity and Security in AWS. Her entire career has been focused on helping customers of all sizes solve their Identity and Access Management problems. She enjoys traveling and experiencing new cultures and food.</p> 
 </div> 
</footer>
