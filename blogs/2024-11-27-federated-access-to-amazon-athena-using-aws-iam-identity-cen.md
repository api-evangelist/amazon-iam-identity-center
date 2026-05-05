---
title: "Federated access to Amazon Athena using AWS IAM Identity Center"
url: "https://aws.amazon.com/blogs/security/federated-access-to-amazon-athena-using-aws-iam-identity-center/"
date: "Wed, 27 Nov 2024 21:14:16 +0000"
author: "Ajay Rawat"
feed_url: "https://aws.amazon.com/blogs/security/tag/aws-iam-identity-center/feed/"
---
<p>Managing <a href="https://aws.amazon.com/athena/" rel="noopener" target="_blank">Amazon Athena</a> through identity federation allows you to manage authentication and authorization procedures centrally. Athena is a serverless, interactive analytics service that provides a simplified and flexible way to analyze petabytes of data.</p> 
<p>In this blog post, we show you how you can use the <a href="https://docs.aws.amazon.com/athena/latest/ug/jdbc-v3-driver.html" rel="noopener" target="_blank">Athena JDBC driver</a> (which includes a browser Security Assertion Markup Language (SAML) plugin) to connect to Athena from third-party SQL client tools, which helps you quickly implement identity federation capabilities and <a href="https://aws.amazon.com/iam/features/mfa/" rel="noopener" target="_blank">multi-factor authentication</a> (MFA). This enables automation and enforcement of data access policies across your organization.</p> 
<p>You can use <a href="https://aws.amazon.com/iam/identity-center/" rel="noopener" target="_blank">AWS IAM Identity Center</a> to federate access to users to AWS accounts. IAM Identity Center integrates with <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html" rel="noopener" target="_blank">AWS Organizations</a> to manage access to the AWS accounts under your organization. In this post, you will learn how to configure the Athena driver to use the <a href="https://docs.aws.amazon.com/athena/latest/ug/jdbc-v3-driver-aws-configuration-profile-credentials.html" rel="noopener" target="_blank">AWS configuration profile credentials</a>. This will allow you to resolve credentials from IAM Identity Center and use the MFA capability of your federation identity provider (IdP).In this post, you will learn how you can integrate the Athena browser-based SAML plugin to add single sign-on (SSO) and MFA capability with your federation identity provider (IdP).</p> 
<h2>Prerequisites</h2> 
<p>To implement this solution, you must have the follow prerequisites:</p> 
<ul> 
 <li>An <a href="https://signin.aws.amazon.com/signup" rel="noopener" target="_blank">AWS account</a>.</li> 
 <li>Install or update to the latest version of the <a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html" rel="noopener" target="_blank">AWS Command Line Interface (AWS CLI)</a>.</li> 
 <li>Configure IAM Identity Center <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html" rel="noopener" target="_blank">authentication using the AWS CLI.</a></li> 
 <li>IAM Identity Center enabled. See <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/get-set-up-for-idc.html" rel="noopener" target="_blank">Enabling AWS IAM Identity Center</a>.</li> 
 <li>Access to SQL client tools (such as SQL Workbench/J, Pycharm, and so on) that support JDBC connections.</li> 
 <li>An <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a> bucket to store Athena query <a href="https://docs.aws.amazon.com/athena/latest/ug/querying.html" rel="noopener" target="_blank">results</a>.</li> 
 <li>Knowledge of using <a href="https://aws.amazon.com/lake-formation/" rel="noopener" target="_blank">AWS Lake Formation</a> and enabling Lake Formation to manage permissions to a set of tables.</li> 
 <li>A Lake Formation administrator role. See <a href="https://docs.aws.amazon.com/lake-formation/latest/dg/permissions-reference.html" rel="noopener" target="_blank">Lake Formation personas and IAM permissions reference</a> for information on creating a data lake administrator.</li> 
 <li>Tables and databases are populated in your <a href="https://docs.aws.amazon.com/glue/latest/dg/catalog-and-crawler.html" rel="noopener" target="_blank">AWS Glue Data Catalog</a>.</li> 
 <li>Create two <a href="https://docs.aws.amazon.com/athena/latest/ug/user-created-workgroups.html" rel="noopener" target="_blank">Athena workgroups</a> (for example, sensitive and non-sensitive).</li> 
</ul> 
<blockquote>
 <p><strong>Note: </strong>Lake Formation only supports a single role in the SAML assertion. Multiple roles cannot be used.</p>
</blockquote> 
<h2>Solution overview</h2> 
<div class="wp-caption aligncenter" id="attachment_36694" style="width: 790px;">
 <img alt="Figure 1: Solution architecture" class="size-large wp-image-36694" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/27/img1_v2-1024x864.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-36694">Figure 1: Solution architecture</p>
</div> 
<p>To implement the solution, complete the steps below as shown in Figure 1:</p> 
<ol> 
 <li>An IAM Identity Center delegated administrator creates two <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/permissionsetsconcept.html" rel="noopener" target="_blank">custom permission sets</a> within Identity Center.</li> 
 <li>An IAM Identity Center delegated administrator assign permission sets to AWS accounts and users and groups. The user has permissions to single sign-on roles that are provisioned in the data lake account. The role created by Identity Center has a name that begins with <code style="color: #000000;">AWSReservedSSO</code>.</li> 
 <li>A Lake Formation administrator grants <a href="https://aws.amazon.com/what-is/sso/" rel="noopener" target="_blank">single sign-on </a>roles permissions to the corresponding database and tables.</li> 
</ol> 
<p>The solution workflow consists of the following high-level steps as shown in Figure 1:</p> 
<ol start="4"> 
 <li>The user configures IAM Identity Center authentication using the AWS CLI.</li> 
 <li>The AWS CLI redirects the user to the AWS access portal URL. The user enters workforce identity credentials (username and password). Then chooses <strong>Sign in</strong>.</li> 
 <li>The AWS access portal verifies the user’s identity. IAM Identity Center redirects the request to the Identity Center authentication service to validate the user’s credentials.</li> 
 <li>If MFA is enabled for the user, then they are prompted to authenticate their MFA device.</li> 
 <li>The user enters or approves the MFA details. The user’s MFA is successfully completed.</li> 
 <li>The user selects the AWS account to use from the displayed list. Then select the IAM single sign-on role to use from the displayed list.</li> 
 <li>The user tests the SQL client connection and then uses the client to run a SQL query.</li> 
 <li>The client makes a call to Athena to retrieve the table and associated metadata from the Data Catalog.</li> 
 <li>Athena requests access to the data from Lake Formation. Lake Formation invokes the&nbsp;<a href="https://docs.aws.amazon.com/STS/latest/APIReference/welcome.html" rel="noopener" target="_blank">AWS Security Token Service&nbsp;(AWS STS)</a>.</li> 
 <li>Lake Formation invokes AWS STS. 
  <ol> 
   <li>Lake Formation obtains temporary AWS credentials with the permissions of the defined IAM role (sensitive or non-sensitive) associated with the data lake location.</li> 
   <li>Lake Formation returns temporary credentials to Athena.</li> 
  </ol> </li> 
 <li>Athena uses the temporary credentials to retrieve data objects from Amazon S3.</li> 
 <li>The Athena engine successfully runs the query and returns the results to the client.</li> 
</ol> 
<h2>Solution walkthrough</h2> 
<p>The walkthrough includes five sections that will guide you through the process of creating permission sets, assigning permission sets to AWS Accounts, managing permission sets access using Lake Formation, and setting up third-party SQL clients such as SQL Workbench to connect to your data store and query your data through Athena.</p> 
<h3>Step 1: Federate onboarding</h3> 
<p>Federating onboarding is done within the IAM Identity Center account. As part of federated onboarding, you need to create IAM Identity Center users and groups. Groups are a collection of people who have the same security rights and permissions. You can create groups and add users to the groups. Create one IAM Identity Center group for sensitive data and another for non-sensitive data to provide distinct access to different classes of data sets. You can assign access to IAM Identity Center permission sets to a user or group.</p> 
<p><strong>To federate onboarding:</strong></p> 
<ol> 
 <li>Open the AWS Management Console using the IAM Identity Center account and go to IAM Identity Center.</li> 
 <li>Choose <strong>Groups</strong>.</li> 
 <li>Choose <strong>Create group</strong>.</li> 
 <li>Enter a <strong>Group name</strong> and <strong>Description </strong>.</li> 
 <li>Choose <strong>Create group</strong>.</li> 
</ol> 
<p><strong>To add a user as a member of a group:</strong></p> 
<ol> 
 <li>Open the IAM Identity Center console.</li> 
 <li>Choose <strong>Groups</strong>.</li> 
 <li>Select the <strong>group name</strong> that you want to update.</li> 
 <li>On the group details page, under <strong>Users in this group</strong>, choose <strong>Add users to group</strong>.</li> 
 <li>On the <strong>Add users to group</strong> page, under <strong>Other users</strong>, locate the users you want to add as members and select the check box next to each of them.</li> 
 <li>Choose <strong>Add users to group</strong>.</li> 
</ol> 
<p style="line-height: 1.25em;"></p>
<div class="wp-caption aligncenter" id="attachment_36690" style="width: 790px;">
 <img alt="Figure 2: Assigning users to a group" class="size-large wp-image-36690" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/26/img2-1024x383.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-36690">Figure 2: Assigning users to a group</p>
</div>
<p></p> 
<h3>Step 2: Create permission sets</h3> 
<p>For this step, create two permission sets (<code style="color: #000000;">sensitive-iam-role</code> and <code style="color: #000000;">non-sensitive-iam-role</code>). These permission sets can be assigned to users or groups in IAM Identity Center, granting them specific access to AWS account resources.</p> 
<p><strong>To create custom permission sets:</strong></p> 
<ol> 
 <li>In the IAM Identity Center administrator account, under <strong>Multi-Account permissions</strong>, choose <strong>Permission sets.</strong></li> 
 <li>Choose <strong>Create permission set</strong>.</li> 
 <li>On the <strong>Select permission set type</strong> page, under <strong>Permission set type</strong>, choose <strong>Custom permission set.</strong> <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_36691" style="width: 750px;">
   <img alt="Figure 3: Selecting a permission set" class="size-large wp-image-36691" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/26/img3-1024x371.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-36691">Figure 3: Selecting a permission set</p>
  </div><p></p> </li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>On the <strong>Specify policies and permission boundary</strong> page, expand <strong>Inline policy</strong> to add custom JSON-formatted policy text.</li> 
 <li>Insert the following policy and update the S3 bucket name (<code style="color: #ff0000; font-style: italic;">&lt;s3-bucket-name&gt;</code>), AWS Region (<code style="color: #ff0000; font-style: italic;">&lt;region&gt;</code>) account ID (<code style="color: #ff0000; font-style: italic;">&lt;account-id&gt;</code>), CloudWatch alarm name (<code style="color: #ff0000; font-style: italic;">&lt;AlarmName&gt;</code>), Athena workgroup name (sensitive or non-sensitive) (<code style="color: #ff0000; font-style: italic;">&lt;WorkGroupName&gt;</code>), KMS key alias name (<code style="color: #ff0000; font-style: italic;">&lt;KMS-key-alias-name&gt;</code>), and organization ID (<code style="color: #ff0000; font-style: italic;">&lt;aws-PrincipalOrgID&gt;</code>). 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">{
  "Statement": [
    {
      "Action": [
        "lakeformation:SearchTablesByLFTags",
        "lakeformation:SearchDatabasesByLFTags",
        "lakeformation:ListLFTags",
        "lakeformation:GetResourceLFTags",
        "lakeformation:GetLFTag",
        "lakeformation:GetDataAccess",
        "glue:SearchTables",
        "glue:GetTables",
        "glue:GetTable",
        "glue:GetPartitions",
        "glue:GetDatabases",
        "glue:GetDatabase"
      ],
      "Effect": "Allow",
      "Resource": "*",
      "Sid": "LakeformationAccess"
    },
    {
      "Action": [
        "s3:PutObject",
        "s3:ListMultipartUploadParts",
        "s3:ListBucketMultipartUploads",
        "s3:ListBucket",
        "s3:GetObject",
        "s3:GetBucketLocation",
        "s3:CreateBucket",
        "s3:AbortMultipartUpload"
      ],
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::<span style="color: #ff0000; font-style: italic;">&lt;s3-bucket-name&gt;</span>/*",
        "arn:aws:s3:::<span style="color: #ff0000; font-style: italic;">&lt;s3-bucket-name&gt;</span>"
      ],
      "Sid": "S3Access"
    },
    {
      "Action": "s3:ListAllMyBuckets",
      "Effect": "Allow",
      "Resource": "*",
      "Sid": "AthenaS3ListAllBucket"
    },
    {
      "Action": [
        "cloudwatch:PutMetricAlarm",
        "cloudwatch:DescribeAlarms"
      ],
      "Effect": "Allow",
      "Resource": [
        "arn:aws:cloudwatch:<span style="color: #ff0000; font-style: italic;">&lt;region&gt;</span>:<span style="color: #ff0000; font-style: italic;">&lt;account-id&gt;</span>:alarm:<span style="color: #ff0000; font-style: italic;">&lt;AlarmName&gt;</span>"
      ],
      "Sid": "CloudWatchLogs"
    },
    {
      "Action": [
        "athena:UpdatePreparedStatement",
        "athena:StopQueryExecution",
        "athena:StartQueryExecution",
        "athena:ListWorkGroups",
        "athena:ListTableMetadata",
        "athena:ListQueryExecutions",
        "athena:ListPreparedStatements",
        "athena:ListNamedQueries",
        "athena:ListEngineVersions",
        "athena:ListDatabases",
        "athena:ListDataCatalogs",
        "athena:GetWorkGroup",
        "athena:GetTableMetadata",
        "athena:GetQueryResultsStream",
        "athena:GetQueryResults",
        "athena:GetQueryExecution",
        "athena:GetPreparedStatement",
        "athena:GetNamedQuery",
        "athena:GetDatabase",
        "athena:GetDataCatalog",
        "athena:DeletePreparedStatement",
        "athena:DeleteNamedQuery",
        "athena:CreatePreparedStatement",
        "athena:CreateNamedQuery",
        "athena:BatchGetQueryExecution",
        "athena:BatchGetNamedQuery"
      ],
      "Effect": "Allow",
      "Resource": [
        "arn:aws:athena:<span style="color: #ff0000; font-style: italic;">&lt;region&gt;</span>:<span style="color: #ff0000; font-style: italic;">&lt;account-id&gt;</span>:workgroup/<span style="color: #ff0000; font-style: italic;">&lt;WorkGroupName&gt;</span>",
        "arn:aws:athena:{Region}:{Account}:datacatalog/{DataCatalogName}"
      ],
      "Sid": "AthenaAllow"
    },
    {
      "Action": [
        "kms:GenerateDataKey",
        "kms:DescribeKey",
        "kms:Decrypt"
      ],
      "Condition": {
        "ForAnyValue:StringLike": {
          "kms:ResourceAliases": "<span style="color: #ff0000; font-style: italic;">&lt;KMS-key-alias-name&gt;</span>"
        }
      },
      "Effect": "Allow",
      "Resource": "*",
      "Sid": "kms"
    },
    {
      "Action": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:PrincipalOrgID": "<span style="color: #ff0000; font-style: italic;">&lt;aws-PrincipalOrgID&gt;</span>"
        }
      },
      "Effect": "Deny",
      "Resource": "*",
      "Sid": "denyRule"
    }
  ],
  "Version": "2012-10-17"
}</code></pre> 
  </div> </li> 
 <li>Update the custom policy to add the corresponding Athena workgroup ARN for the sensitive and non-sensitive IAM roles.<br /> 
  <blockquote>
   <p><strong>Note</strong>: See the documentation for information about <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html" rel="noopener" target="_blank">AWS global condition context keys</a>.</p>
  </blockquote> </li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>On the <strong>Specify permission set details</strong> page, enter a name to identify this permission set in IAM Identity Center. The name that you specify for this permission set appears in the AWS access portal as an available role. Users sign in to the AWS access portal, choose an AWS account, and then choose the role.</li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>On the <strong>Review and create</strong> page, review the selections that you made, and then choose <strong>Create</strong>.</li> 
</ol> 
<h3>Step 3: Assign permission sets to AWS accounts</h3> 
<p>You can add and remove permissions sets for an IAM user or group by attaching and detaching permission sets. Permission sets define what actions an identity can perform on which AWS resources.</p> 
<p><strong>To assign permission sets to AWS accounts:</strong></p> 
<ol> 
 <li>In the IAM Identity Center administrator account, under <strong>Multi-account permissions</strong>, choose <strong>AWS accounts</strong>.</li> 
 <li>On the <strong>AWS accounts</strong> page, select one or more AWS accounts that you want to assign single sign-on access to.</li> 
 <li>Choose <strong>Assign users or groups</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_36697" style="width: 750px;">
   <img alt="Figure 4: Selecting users and groups" class="size-large wp-image-36697" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/27/img4-1-1024x457.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-36697">Figure 4: Selecting users and groups</p>
  </div><p></p> </li> 
 <li>On the <strong>Assign users and groups to “&lt;AWS account name&gt;”</strong>, for <strong>Selected users and groups</strong>, choose the users that you want to create the permission set for. Choose <strong>Next</strong>.</li> 
 <li>Select permission sets: On the <strong>Assign permission sets to “AWS-account-name”</strong> page, select one or more permission sets.</li> 
 <li>On the <strong>Review and submit assignments to AWS-account-name</strong> page, for <strong>Review and submit</strong>, choose <strong>Submit</strong>.</li> 
</ol> 
<h3>Step 4. Grant permissions to IAM (single sign-on) roles</h3> 
<p>A data lake administrator has the broad ability to grant a principal (including themselves) permissions on Data Catalog resources. This includes the ability to manage access controls and permissions for the data lake. When you grant Lake Formation permissions on a specific Data Catalog table, you can also include data filtering specifications. This allows you to further restrict access to certain data within the table, limiting what users can see in their query results based on those filtering rules.</p> 
<p><strong>To grant permissions to IAM roles:</strong></p> 
<p>In the Lake Formation console, under&nbsp;<strong>Permissions</strong>&nbsp;in the navigation pane, select <strong>Data Lake permissions</strong>, and then choose <strong>Grant</strong>.</p> 
<p><strong>To grant Database permissions to IAM roles:</strong></p> 
<ol> 
 <li>Under <strong>Principals</strong>, select the IAM role name (for example, <strong>Sensitive-IAM-Role</strong>).</li> 
 <li>Under <strong>Named Data Catalog resources</strong>, go to <strong>Databases</strong> and select a database (for example, <strong>demo</strong>). <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_36698" style="width: 750px;">
   <img alt="Figure 5: Select an IAM role and database" class="size-large wp-image-36698" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/27/img5-1024x734.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-36698">Figure 5: Select an IAM role and database</p>
  </div><p></p> </li> 
 <li>Under <strong>Database permissions</strong>, select <strong>Describe</strong> and then choose <strong>Grant</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_36699" style="width: 750px;">
   <img alt="Figure 6: Grant database permissions to an IAM role" class="size-full wp-image-36699" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/27/img6.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-36699">Figure 6: Grant database permissions to an IAM role</p>
  </div><p></p> </li> 
</ol> 
<p><strong>To grant tables permissions to IAM roles:</strong></p> 
<ol> 
 <li>Repeat steps 1 and 2 of the preceding procedure.</li> 
 <li>Under <strong>Tables – optional</strong>, select a table name (for example, <strong>demo2</strong>). <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_36700" style="width: 750px;">
   <img alt="Figure 7: Select tables within a database to grant access" class="size-large wp-image-36700" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/27/img7-1024x674.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-36700">Figure 7: Select tables within a database to grant access</p>
  </div><p></p> </li> 
 <li>Select the desired <strong>Table Permissions</strong> (for example, <strong>select and describe</strong>), and then choose <strong>Grant</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_36701" style="width: 750px;">
   <img alt="Figure 8: Grant access to tables within the database" class="size-large wp-image-36701" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/27/img8-1-1024x930.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-36701">Figure 8: Grant access to tables within the database</p>
  </div><p></p> </li> 
 <li>Repeat steps 1 through 4 to grant access for the respective database and tables for the non-sensitive IAM role.</li> 
</ol> 
<h3>Step 5: Client-side setup using JDBC</h3> 
<p>You can use a JDBC connection to connect Athena and SQL client applications (for example, PyCharm or SQL Workbench) to enable analytics and reporting on the data that Athena returns from Amazon S3 databases. To use the Athena JDBC driver, you must specify the driver class from the JAR file. Additionally, you must pass in some parameters to change the authentication mechanism so the athena-sts-auth libraries are used:</p> 
<ul> 
 <li>S3 output location – Where in S3 the Athena service can write its output. For example, <code style="color: #000000;"><a href="https://docs.aws.amazon.com/athena/latest/ug/jdbc-v3-driver-basic-connection-parameters.html" rel="noopener" target="_blank">s3://path/to/query/bucket/</a></code>.</li> 
 <li>The IAM Identity Center administrator can configure the session duration for the AWS access portal. The session duration can be set from a minimum of 15 minutes to a maximum of 90 days.</li> 
</ul> 
<p><strong>To set up PyCharm </strong></p> 
<ol> 
 <li>Install Athena JDBC 3.x driver from <a href="https://docs.aws.amazon.com/athena/latest/ug/jdbc-v3-driver.html" rel="noopener" target="_blank">Athena JDBC 3.x driver</a>. 
  <ol> 
   <li>In the left navigation pane, select <strong>JDBC 3.x </strong>and then <strong>Getting started</strong>. Select <strong>Uber jar</strong> to download a .jar file, which contains the driver and its dependencies. <p style="line-height: 1.25em;"></p>
    <div class="wp-caption aligncenter" id="attachment_36702" style="width: 710px;">
     <img alt="Figure 9: Download Athena JDBC jar" class="size-large wp-image-36702" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/27/img9-1-1024x482.png" style="border: 1px solid #bebebe;" width="700" />
     <p class="wp-caption-text" id="caption-attachment-36702">Figure 9: Download Athena JDBC jar</p>
    </div><p></p> </li> 
  </ol> </li> 
 <li>Open PyCharm and create a new project. 
  <ol> 
   <li>Enter a <strong>Name</strong> for your project</li> 
   <li>Select the desired project <strong>Location </strong></li> 
   <li>Choose <strong>Create</strong></li> 
  </ol> <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_36703" style="width: 750px;">
   <img alt="Figure 10: Create a new project in PyCharm" class="size-large wp-image-36703" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/27/img10-1-1024x817.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-36703">Figure 10: Create a new project in PyCharm</p>
  </div><p></p> </li> 
 <li>Configure <strong>Data Source</strong> and drivers. Select <strong>Data Source</strong>, and then choose the plus sign or <strong>New</strong> to configure new data sources and drivers. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_36704" style="width: 750px;">
   <img alt="Figure 11: Add database source properties" class="size-large wp-image-36704" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/27/img11-1024x681.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-36704">Figure 11: Add database source properties</p>
  </div><p></p> </li> 
 <li>Configure the Athena driver by selecting the <strong>Drivers</strong> tab, and then choose the plus sign to add a new driver. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_36705" style="width: 750px;">
   <img alt="Figure 12: Add database drivers" class="size-large wp-image-36705" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/27/img12-1024x745.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-36705">Figure 12: Add database drivers</p>
  </div><p></p> </li> 
 <li>Under <strong>Driver Files</strong>, upload the custom JAR file that you downloaded in the Step 1. Select the Athena class dropdown. Enter the driver’s name (for example <code style="color: #000000;">Athena JDBC Driver</code>). Then choose <strong>Apply</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_36706" style="width: 750px;">
   <img alt="Figure 13: Add database driver files" class="size-large wp-image-36706" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/27/img13-1024x740.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-36706">Figure 13: Add database driver files</p>
  </div><p></p> </li> 
 <li>Configure a new data source. Choose the plus sign and select your driver’s name from the driver dropdown.</li> 
 <li>Enter the data source name (for example, <code style="color: #000000;">Athena Demo</code>). For the authentication method, select <strong>User &amp; Password</strong>. Then choose <strong>Apply</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_36707" style="width: 750px;">
   <img alt="Figure 14: Create a project data source profile" class="size-large wp-image-36707" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/27/img14-1024x999.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-36707">Figure 14: Create a project data source profile</p>
  </div><p></p> </li> 
 <li>Select the <strong>SSH/SSL</strong> tab and select <strong>Use SSL</strong>. Verify that the <strong>Use truststore</strong> options for IDE, JAVA, and system are all selected. Then choose <strong>Apply</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_36708" style="width: 750px;">
   <img alt="Figure 15: Enable data source profile SSL" class="size-large wp-image-36708" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/27/img15-1024x1007.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-36708">Figure 15: Enable data source profile SSL</p>
  </div><p></p> </li> 
 <li>Select the <strong>Options</strong> tab and then select <strong>Single Session Mode</strong>. Then choose <strong>Apply</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_36709" style="width: 750px;">
   <img alt="Figure 16: Configure single session mode in PyCharm" class="size-large wp-image-36709" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/27/img16-991x1024.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-36709">Figure 16: Configure single session mode in PyCharm</p>
  </div><p></p> </li> 
 <li>Select the <strong>General</strong> tab and enter the JDBC and single sign-on URL. The following is a sample JDBC URL based on the SAML application: 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">jdbc:athena://;CredentialsProvider= ProfileCredentials; ProfileName=<span style="color: #ff0000; font-style: italic;">&lt;name-of-the-profile&gt;</span>;WorkGroup=<span style="color: #ff0000; font-style: italic;">&lt;name-of-the-WorkGroup&gt;</span>; </code></pre> 
  </div> 
  <ol> 
   <li>Choose <strong>Apply</strong>.</li> 
   <li>Choose <strong>Test Connection</strong>. If the profile has expired, refresh the single sign-on session by running <code style="color: #000000;">aws sso login --profile <span style="color: #ff0000; font-style: italic;">&lt;profile-name&gt;</span></code> with the corresponding profile.</li> 
  </ol> <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_36710" style="width: 750px;">
   <img alt="Figure 17: Test the data source connection" class="size-large wp-image-36710" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/27/img17-1024x828.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-36710">Figure 17: Test the data source connection</p>
  </div><p></p> </li> 
 <li>After the connection is successful, select the <strong>Schemas</strong> tab and select <strong>All databases </strong>and <strong>All schemas</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_36711" style="width: 750px;">
   <img alt="Figure 18: Select data source databases and schemas" class="size-large wp-image-36711" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/27/img18-1024x727.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-36711">Figure 18: Select data source databases and schemas</p>
  </div><p></p> </li> 
 <li>Run a sample test query: <code style="color: #000000;">SELECT &lt;table-names&gt; FROM &lt;database-name&gt; limit 10;</code></li> 
 <li>Verify that the credentials and permissions are working as expected.</li> 
</ol> 
<p><strong>To set up SQL Workbench</strong></p> 
<ol> 
 <li>Open SQL Workbench.</li> 
 <li>Configure an Athena driver by selecting <strong>File</strong> and then <strong>Manage Drivers</strong>.</li> 
 <li>Enter the <code style="color: #000000;">Athena JDBC Driver</code> as the name and set the library to browse the path for the location where you downloaded the driver. Enter <code style="color: #000000;">amazonaws.athena.jdbc.AthenaDriver</code> as the <strong>Classname</strong>.</li> 
 <li>Enter the following URL, replacing <code style="color: #ff0000; font-style: italic;">&lt;name-of-the-WorkGroup&gt;</code> with your workgroup name. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">jdbc:athena://;CredentialsProvider=ProfileCredentials;ProfileName=<span style="color: #ff0000; font-style: italic;">&lt;name-of-the-profile&gt;</span>;WorkGroup=<span style="color: #ff0000; font-style: italic;">&lt;name-of-the-WorkGroup&gt;</span>;</code></pre> 
  </div> </li> 
 <li>Choose <strong>OK</strong>.</li> 
 <li>Run a test query, replacing <code style="color: #ff0000; font-style: italic;">&lt;table-names&gt;</code> and <code style="color: #ff0000; font-style: italic;">&lt;database-name&gt;</code> with your table and database names: 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">SELECT <em>&lt;table-names&gt;</em> FROM <span style="color: #ff0000; font-style: italic;">&lt;database-name&gt;</span> limit 10;</code></pre> 
  </div> </li> 
 <li>Verify that the credentials and permissions are working as expected.</li> 
</ol> 
<h2>Conclusion</h2> 
<p>In this post, we covered how to use JDBC drivers to connect to Athena from third-party SQL client tools. You were able to set this up without creating IAM users or any type of long-lived credentials that would need to be stored on your developers’ workstations. You learned how to configure IAM Identity Center users and groups, create permission sets, and assign permission sets to AWS Accounts. You also learned how to grant permissions to single sign-on roles using Lake Formation to create distinct access to different classes of data sets and connect to Athena through an SQL client tool (such as PyCharm). This setup can also work with other&nbsp;<a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/step2.html" rel="noopener" target="_blank">supported identity sources</a> such as <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/manage-your-identity-source-sso.html" rel="noopener" target="_blank">IAM Identity Center</a>,&nbsp;<a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/connectonpremad.html" rel="noopener" target="_blank">self-managed or on-premises Active Directory</a>, or an&nbsp;<a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/manage-your-identity-source-idp.html" rel="noopener" target="_blank">external IdP</a>.</p> 
<p>&nbsp;<br />If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.<br />&nbsp;</p> 
<footer> 
 <div class="blog-author-box">
  <img alt="Ajay Rawat" class="alignleft size-full" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2022/08/30/Ajay-Rawat.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Ajay Rawat</span>
  <br />Ajay is a Senior Security Consultant, focusing on AWS Identity and Access Management (IAM), data protection, incident response, and operationalizing AWS security services to increase security effectiveness and reduce risk. Ajay is a technology enthusiast and enjoys working with customers to solve their technical challenges and to improve their security posture in the cloud.
 </div> 
 <div class="blog-author-box">
  <img alt="Mihir Borkar" class="alignleft size-full" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/07/30/bormihir.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Mihir Borkar</span>
  <br />Mihir is an AWS Data Architect who excels at simplifying customer challenges with innovative cloud data solutions. Specializing in AWS Lake Formation and AWS Glue, he designs scalable data lakes and analytics platforms, demonstrating expertise in crafting efficient solutions within the AWS Cloud.
 </div> 
</footer>
