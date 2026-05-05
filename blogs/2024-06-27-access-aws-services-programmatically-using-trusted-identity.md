---
title: "Access AWS services programmatically using trusted identity propagation"
url: "https://aws.amazon.com/blogs/security/access-aws-services-programmatically-using-trusted-identity-propagation/"
date: "Thu, 27 Jun 2024 15:57:34 +0000"
author: "Roberto Migli"
feed_url: "https://aws.amazon.com/blogs/security/tag/aws-iam-identity-center/feed/"
---
<blockquote>
 <p><strong>March 7, 2025:</strong> This post was republished to update the code, architecture, and narrative introducing the launch of Single Sign-on and trusted identity propagation support for Amazon Redshift Data API with AWS IAM Identity Center.</p>
</blockquote> 
<hr /> 
<p>With the introduction of <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/trustedidentitypropagation.html" rel="noopener" target="_blank">trusted identity propagation</a>, applications can now propagate a user’s workforce identity from their identity provider (IdP) to applications running in <a href="https://aws.amazon.com/aws" rel="noopener" target="_blank">Amazon Web Services (AWS)</a> and to storage services backing those applications, such as <a href="https://aws.amazon.com/pm/serv-s3/" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a> or <a href="https://aws.amazon.com/glue/" rel="noopener" target="_blank">AWS Glue</a>. Since access to applications and data can now be granted directly to a workforce identity, a seamless single sign-on experience can be achieved, eliminating the need for users to be aware of different <a href="https://aws.amazon.com/iam" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> roles to assume to access data or use of local database credentials.</p> 
<p>While AWS managed applications such as <a href="https://aws.amazon.com/quicksight/" rel="noopener" target="_blank">Amazon QuickSight</a>, <a href="https://aws.amazon.com/lake-formation/" rel="noopener" target="_blank">AWS Lake Formation</a>, or <a href="https://aws.amazon.com/emr/features/studio/" rel="noopener" target="_blank">Amazon EMR Studio</a> offer a native setup experience with trusted identity propagation, there are use cases where custom integrations must be built. You might want to integrate your workforce identities into custom applications storing data in Amazon S3 or build an interface on top of existing applications using Java Database Connectivity (JDBC) drivers, allowing these applications to propagate those identities into AWS to access resources on behalf of their users. AWS resource owners can manage authorization directly in their AWS applications such as Lake Formation or <a href="https://aws.amazon.com/s3/features/access-grants/" rel="noopener" target="_blank">Amazon S3 Access Grants</a>.</p> 
<p>This blog post introduces a sample command-line interface (CLI) application that enables users to access AWS services using their workforce identity from IdPs such as Okta or Microsoft Entra ID.</p> 
<p>The solution relies on users authenticating with their chosen IdP using standard OAuth 2.0 authentication flows to receive an identity token. This token can then be exchanged against <a href="https://docs.aws.amazon.com/STS/latest/APIReference/welcome.html" rel="noopener" target="_blank">AWS Security Token Service (AWS STS)</a> and <a href="https://aws.amazon.com/iam/identity-center/" rel="noopener" target="_blank">AWS IAM Identity Center</a> to access data on behalf of the workforce identity that was used to sign in to the IdP.</p> 
<p>Finally, an integration with the <a href="https://aws.amazon.com/cli/" rel="noopener" target="_blank">AWS Command Line Interface (AWS CLI)</a> is provided, enabling native access to AWS services on behalf of the signed-in identity.</p> 
<p>In this post, you will learn how to build and use the CLI application to access data on S3 Access Grants, query <a href="https://aws.amazon.com/athena/" rel="noopener" target="_blank">Amazon Athena</a> and <a href="https://aws.amazon.com/redshift/" rel="noopener" target="_blank">Amazon Redshift</a> tables, and programmatically interact with other AWS services that support trusted identity propagation.</p> 
<p>To set up the solution in this blog post, you should already be familiar with trusted identity propagation and S3 Access Grants concepts and features. If you are not, see the two-part blog post <a href="https://aws.amazon.com/blogs/storage/how-to-develop-a-user-facing-data-application-with-iam-identity-center-and-s3-access-grants/" rel="noopener" target="_blank">How to develop a user-facing data application with IAM Identity Center and S3 Access Grants (Part 1)</a> and <a href="https://aws.amazon.com/blogs/storage/how-to-develop-a-user-facing-data-application-with-iam-identity-center-and-s3-access-grants-part-2/" rel="noopener" target="_blank">Part 2</a>. In this post we guide you through a more hands-on setup of your own sample CLI application.</p> 
<h2>Architecture and token exchange flow</h2> 
<p>Before moving into the actual deployment, let’s talk about the architecture of the CLI and how it facilitates token exchange between different parties.</p> 
<p>Users, such as developers and data scientists, run the CLI on their machine without <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/security-creds.html" rel="noopener" target="_blank">AWS security credentials</a> pre-configured. To perform a token exchange of the OAuth 2.0 credentials vended by a source IdP towards IAM Identity Center by using the <a href="https://docs.aws.amazon.com/singlesignon/latest/OIDCAPIReference/API_CreateTokenWithIAM.html" rel="noopener" target="_blank">CreateTokenWithIAM</a> API, AWS security credentials are required. To fulfill this requirement, the solution uses IAM <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_oidc.html" rel="noopener" target="_blank">OpenID Connect (OIDC) federation</a> to first create an IAM role session using the <a href="https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRoleWithWebIdentity.html" rel="noopener" target="_blank">AssumeRoleWithWebIdentity</a> API with the initial IdP token—because this API doesn’t require credentials—and then uses the resulting IAM role to request the necessary single sign-on OIDC token.</p> 
<p>The process flow is shown in Figure 1:</p> 
<p style="line-height: 1.25em;"></p>
<div class="wp-caption aligncenter" id="attachment_37560" style="width: 790px;">
 <img alt="Figure 1: Architecture diagram of the application" class="size-full wp-image-37560" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/03/07/img1-1.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-37560">Figure 1: Architecture diagram of the application</p>
</div>
<p></p> 
<p>At a high level, the interaction flow shown in Figure 1 is the following:</p> 
<ol> 
 <li>The user interacts with the AWS CLI, which uses the CLI as a <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sourcing-external.html" rel="noopener" target="_blank">source credential provider</a>.</li> 
 <li>The user is asked to sign in through their browser with the source IdP they configured in the CLI, for example Okta.</li> 
 <li>If the authorization is successful, the CLI receives a JSON Web Token (JWT) and uses it to assume an IAM role through OIDC federation using <a href="https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRoleWithWebIdentity.html" rel="noopener" target="_blank">AssumeRoleWithWebIdentity</a>.</li> 
 <li>With this temporary IAM role session, the CLI exchanges the IdP token token with the IAM Identity Center <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/trustedidentitypropagation-using-customermanagedapps-setup.html" rel="noopener" target="_blank">customer managed application</a> on behalf of the user for another token using the <span style="font-family: courier;">CreateTokenWithIAM</span> API.</li> 
 <li>If successful, the CLI uses the token returned from Identity Center to create an <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/trusted-identity-propagation-using-aws-managed-applications.html#trustedidentitypropagation-identity-enhanced-iam-role-sessions" rel="noopener" target="_blank">identity-enhanced IAM role session</a> and returns the corresponding IAM credentials to the AWS CLI.</li> 
 <li>The AWS CLI uses the credentials to call AWS services that support the trusted identity propagation that has been configured in the Identity Center customer managed application. For example, to query Athena or Amazon Redshift. See <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/trustedidentitypropagation-using-customermanagedapps-setup.html#trustedidentitypropagation-using-customermanagedapps-specify-trusted-apps" rel="noopener" target="_blank">Specify trusted applications</a> in the Identity Center documentation to learn more.</li> 
 <li>If you need access to S3 Access Grants, the CLI also provides automatic credentials requests to S3 <a href="https://docs.aws.amazon.com/AmazonS3/latest/API/API_control_GetDataAccess.html" rel="noopener" target="_blank">GetDataAccess</a> with the identity-enhanced IAM role session previously created.</li> 
</ol> 
<p>Because both the Okta OAuth token and the identity-enhanced IAM role session credentials are short lived, the CLI provides functionality to refresh authentication automatically.</p> 
<p>Figure 2 is a swimlane diagram of the requests described in the preceding flow and depicted in Figure 1.</p> 
<div class="wp-caption aligncenter" id="attachment_34615" style="width: 790px;">
 <img alt="Figure 2: Swimlane diagram of the application token exchange. Dashed lines show SigV4 signed requests" class="size-full wp-image-34615" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/06/26/img2-4.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-34615">Figure 2: Swimlane diagram of the application token exchange. Dashed lines show SigV4 signed requests</p>
</div> 
<h2>Account prerequisites</h2> 
<p>In the following procedure, you will use two AWS accounts: one is the application account, where required IAM roles and OIDC federation will be deployed and where users will be granted access to Amazon S3 objects. This account already has S3 Access Grants and an Athena workgroup configured with IAM Identity Center. If you don’t have these already configured, see <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-grants-get-started.html" rel="noopener" target="_blank">Getting started with S3 Access Grants</a> and <a href="https://docs.aws.amazon.com/athena/latest/ug/workgroups-identity-center.html" rel="noopener" target="_blank">Using IAM Identity Center enabled Athena workgroups</a>. For Amazon Redshift, after you enable the cluster or workgroup with Identity Center, you need to grant access to your users or groups. Follow the steps in <a href="https://aws.amazon.com/blogs/big-data/integrate-identity-provider-idp-with-amazon-redshift-query-editor-v2-and-sql-client-using-aws-iam-identity-center-for-seamless-single-sign-on/" rel="noopener" target="_blank">this blog post</a> to enable trusted identity propagation for your Amazon Redshift cluster or workgroup and configure the grants.</p> 
<p>The other account is the <em>Identity Center admin account</em> which has IAM Identity Center set up with users and groups <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/manage-your-identity-source-idp.html" rel="noopener" target="_blank">synchronized through SCIM with your IdP of choice</a>, in this case Okta directory. The application also supports Entra ID and <a href="https://aws.amazon.com/cognito" rel="noopener" target="_blank">Amazon Cognito</a>.</p> 
<p>You don’t need to have permission sets configured with IAM Identity Center, because you will grant access through a customer managed application Identity Center. Users authenticate with the source IdP and don’t interact with an AWS account or IAM policy directly.</p> 
<p>To configure this application, you’ll complete the following steps:</p> 
<ol> 
 <li>Create an OIDC application in Okta</li> 
 <li>Create a customer managed application in IAM Identity Center</li> 
 <li>Install and configure the application with the AWS CLI</li> 
</ol> 
<h2>Create an OIDC application in Okta</h2> 
<p>Start by creating a custom application in Okta, which will act as the source IdP, and configuring it as a <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/using-apps-with-trusted-token-issuer.html" rel="noopener" target="_blank">trusted token issuer</a> in IAM Identity Center.</p> 
<p><strong>To create an OIDC application</strong></p> 
<ol> 
 <li>Sign in to your Okta administrator panel, go to the <strong>Applications</strong> section in the navigation pane, and choose <strong>Create App Integration</strong>. Because you’re using a CLI, select <strong>OIDC</strong> as the sign-in method and <strong>Native Application</strong> as the application type. Choose <strong>Next</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_34616" style="width: 750px;">
   <img alt="Figure 3: Okta screen to create a new application" class="size-full wp-image-34616" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/06/26/img3.jpg" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-34616">Figure 3: Okta screen to create a new application</p>
  </div><p></p> </li> 
 <li>In the <strong>Grant Type</strong> section, the authorization code is selected by default. Make sure to also select <strong>Refresh Token</strong>. The CLI application uses the refresh tokens if available to generate new access tokens without requiring the user to re-authenticate. Otherwise, the users will have to authenticate again when the token expires.</li> 
 <li>Change the sign-in redirect URIs to <code style="color: #000000;">http://localhost:8090/callback</code>. The application uses <a href="https://developer.okta.com/blog/2019/08/22/okta-authjs-pkce" rel="noopener" target="_blank">OAuth 2.0 Authorization Code with PKCE Flow</a> and waits for confirmation of authentication by listening locally on port 8090. The IdP will redirect your browser to this URL to send authentication information after successful sign-in.</li> 
 <li>Select the directory groups that you want to have access to this application. If you don’t want to restrict access to the application, choose <strong>Allow Everyone</strong>. Leave the remaining settings as-is.<br /> 
  <blockquote>
   <p><strong>Note:</strong> You must also assign and authorize users in AWS to allow access to the downstream AWS services.</p>
  </blockquote> <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_34617" style="width: 750px;">
   <img alt="Figure 4: Configure the general settings of the Okta application" class="size-full wp-image-34617" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/06/26/img4.jpg" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-34617">Figure 4: Configure the general settings of the Okta application</p>
  </div><p></p> </li> 
 <li>When done, choose <strong>Save</strong> and your Okta custom application will be created and you will see a detail page of your application. Note the Okta <strong>Client ID value</strong> to use in later steps.</li> 
</ol> 
<h2>Create a customer managed application in IAM Identity Center</h2> 
<p>After setting everything up in Okta, move on to creating the custom OAuth 2.0 application in IAM Identity Center. The CLI will use this application to exchange the tokens issued by Okta.</p> 
<p><strong>To create a customer managed application</strong></p> 
<ol> 
 <li>Sign in to the <a href="https://console.aws.amazon.com/" rel="noopener" target="_blank">AWS Management Console</a> of the account where you already have configured IAM Identity Center and go to the Identity Center console.</li> 
 <li>In the navigation pane, select <strong>Applications</strong> under <strong>Application assignments</strong>.</li> 
 <li>Choose <strong>Add application</strong> in the upper</li> 
 <li>Select <strong>I have an application I want to set up</strong> and select the <strong>OAuth 2.0</strong> type.</li> 
 <li> Enter a display name and description.</li> 
 <li>For <strong>User and group assignment method</strong>, select <strong>Do not require assignments</strong> (because you already configured your user and group assignments to this application in Okta). You can leave the Application URL field empty and select <strong>Not Visible</strong> under <strong>Application visibility in AWS access portal</strong>, because you won’t access the application from the Identity Center AWS access portal URL.</li> 
 <li>On the next screen, select the <strong>Trusted Token Issuer</strong> you set up in the prerequisites, and enter your Okta client ID from the Okta application you created in the previous section as the <strong>Aud claim</strong>. This will make sure your Identity Center custom application only accepts tokens issued by Okta for your Okta custom application.<br /> 
  <blockquote>
   <p><strong>Note: Automatically refresh user authentication for active application sessions</strong> isn’t needed for this application because the CLI application already refreshes tokens with Okta.</p>
  </blockquote> <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_34618" style="width: 750px;">
   <img alt="Figure 5: Trusted token issuer configuration of the custom application" class="size-full wp-image-34618" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/06/26/img5-1.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-34618">Figure 5: Trusted token issuer configuration of the custom application</p>
  </div><p></p> </li> 
 <li>Select <strong>Edit the application policy</strong> and edit the policy to specify <em>the application account</em> AWS Account ID as the allowed principal to perform the token exchange. You will update the application policy with the Amazon Resource Name (ARN) of the correct role after deploying the rest of the solution.</li> 
 <li>Continue to the next page to review the application configuration and finalize the creation process. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_34619" style="width: 750px;">
   <img alt="Figure 6: Configuration of the application policy" class="size-full wp-image-34619" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/06/26/img6.jpg" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-34619">Figure 6: Configuration of the application policy</p>
  </div><p></p> </li> 
 <li>Grant the application permission to call other AWS services on the users’ behalf. This step is necessary because the application will use a custom application to exchange tokens that will then be used with specific AWS services. Select the <strong>Customer managed</strong> tab, browse to the application you just created, and choose <strong>Specify trusted applications</strong> at the bottom of the page.</li> 
 <li>For this example, select <strong>All applications for service with same access</strong>, and then select <strong>Athena</strong>, <strong>Amazon Redshift</strong>, <strong>S3 Access Grants</strong>, and <strong>Lake Formation and AWS Glue data catalog</strong> as trusted services because they’re supported by the sample application. If you don’t plan to use your CLI with S3 or Athena and Lake Formation, you can skip this step. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_37561" style="width: 750px;">
   <img alt="Figure 7: Configure application user propagation" class="size-full wp-image-37561" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/03/07/img7.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-37561">Figure 7: Configure application user propagation</p>
  </div><p></p> </li> 
</ol> 
<p>You have completed the setup within IAM Identity Center. Switch to your application account to complete the next steps, which will deploy the backend application.</p> 
<h2>Application installation and configuration with the AWS CLI</h2> 
<p>The sample code for the application can be found on <a href="https://github.com/aws-samples/access-aws-services-programmatically-using-tip" rel="noopener" target="_blank">GitHub</a>. Follow the instructions in the README file to install the command-line application. You can use the CLI to generate an <a href="https://aws.amazon.com/cloudformation/" rel="noopener" target="_blank">AWS CloudFormation</a> template to create the required IAM roles for the token exchange process.</p> 
<p><strong>To install and configure the application</strong></p> 
<ol> 
 <li>You will need your Okta OAuth issuer URI and your Okta application client ID. The issuer URI can be found by going to your Okta Admin page, and on the left navigation pane, go to the API page under the Security section. Here you will see your <strong>Authorization servers</strong> and their <strong>Issuer URI</strong>. The application client ID is the same as you used earlier for the Aud claim in IAM Identity Center.</li> 
 <li>In your CLI, use the following commands to generate and deploy the CloudFormation template: 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">tip-cli configure generate-cfn-template https://dev-12345678.okta.com/oauth2/default aBc12345dBe6789FgH0 &gt; ~/tip-blog-iam-roles-cfn.yaml
aws cloudformation deploy --template-file ~/tip-blog-iam-roles-cfn.yaml --stack-name tip-blog-iam-stack --capabilities CAPABILITY_IAM</code></pre> 
  </div> </li> 
 <li>After the template is created, update the customer managed application policy you configured in step 2 to allow only the IAM role that CloudFormation just created to use the application. You can find this in your CloudFormation console, or by using <code style="color: #000000;">aws cloudformation describe-stacks --stack-name tip-blog-iam-stack</code> in your terminal.</li> 
 <li>Configure the CLI by running <code style="color: #000000;">tip-cli configure idp</code> and entering the IAM roles’ ARN and Okta information.</li> 
 <li>Test your configuration by running the <code style="color: #000000;">tip-cli auth</code> command to start the authorization with the configured IdP by opening your default browser and requesting to sign in. In the background, the application will wait for Okta to callback the browser on port 8090 to retrieve the authentication information, and if successful request an Okta OAuth 2.0 token.</li> 
 <li>You can then run <code style="color: #000000;">tip-cli auth get-iam-credentials</code>&nbsp;to&nbsp;exchange the token through your trusted IAM role and your Identity Center application for a set of identity-enhanced IAM role session credentials.&nbsp;These expire in 1 hour by default, but you can configure the expiration period by configuring the IAM role used to create the identity-enhanced IAM session accordingly. After you test the setup, you won’t need the CLI anymore because authentication, token refresh, and credentials refresh will be managed automatically through the AWS CLI.</li> 
</ol> 
<blockquote>
 <p><strong>Note: </strong>Follow the README instructions on configuring your AWS CLI to use the tip-cli application to source credentials. The advantage of the solution is that the AWS CLI will automatically handle the refresh of tokens and credentials using this application.</p>
</blockquote> 
<p>Now that the AWS CLI is configured, you can use it to call AWS services representing your Okta identity. In the following examples, we configured the AWS CLI to use the application to source credentials for the AWS CLI profile <code style="color: #000000;">my-tti-profile</code>.</p> 
<p>For example, you can query Athena tables through workgroups configured with trusted identity propagation and authorized through Lake Formation:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">QUERY_EXECUTION_ID=$(aws athena start-query-execution \
    --query-string "SELECT * FROM db_tip.table_example LIMIT 3" \
    --work-group "tip-workgroup" \
    --profile my-tti-profile \
    --output text \
    --query 'QueryExecutionId')

aws athena get-query-results \
    --query-execution-id $QUERY_EXECUTION_ID \
    --profile my-tti-profile \
    --output text</code></pre> 
</div> 
<p>Similarly, you can query Amazon Redshift by using the Amazon Redshift Data API. Following is an example that shows queries run with an Amazon Redshift workgroup:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">STATEMENT_ID=$(aws redshift-data execute-statement \
--sql " SELECT * FROM db_tip.table_example LIMIT 3" \
--workgroup-name tip-rs-workgroup \
--database dev \
--profile my-tti-profile | jq -r ".Id")

aws redshift-data get-statement-result \
--id $STATEMENT_ID \
--profile my-tti-profile</code></pre> 
</div> 
<p>The IAM role used by the backend application to create the identity-enhanced IAM session is allowed by default to use only Athena and Amazon S3. You can extend it to allow access to other services, such as <a href="https://aws.amazon.com/q/business/" rel="noopener" target="_blank">Amazon Q Business</a>. After you create an Amazon Q Business application and assign users to it, you can update the custom application you created earlier in IAM Identity Center and enable trusted identity propagation to your Amazon Q Business application.</p> 
<p>You can then, for example, retrieve conversations of the user for an Amazon Q business application with ID <code style="color: #000000;">a1b2c3d4-5678-90ab-cdef-EXAMPLE11111</code> as follows:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">aws qbusiness list-conversations --application-id a1b2c3d4-5678-90ab-cdef-EXAMPLE11111 --profile my-tti-profile</code></pre> 
</div> 
<p>The application also provides simplified access to S3 Access Grants. After you have configured the AWS CLI as documented in the README file, you can access S3 objects as follows:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">aws s3 ls s3://&lt;my-test-bucket-with-path&gt; --profile my-tti-profile-s3ag</code></pre> 
</div> 
<p>In this example, the AWS CLI uses the custom credential source to request data access to a specific S3 URI to the account instance of S3 Access Grants you want to target. If there’s a matching grant for the requested URI and your user identity or group, S3 Access Grants will return a new set of short-lived IAM credentials. The tip-cli application will cache these credentials for you and prompt you to refresh them whenever needed. You can review the list of credentials generated by the application with the command <code style="color: #000000;">tip-cli s3ag&nbsp;list-credentials</code>, and clear them with the command&nbsp;<code style="color: #000000;">tip-cli s3ag clear-credentials</code>.</p> 
<h2>Conclusion</h2> 
<p>In this post we showed you how a sample CLI application using trusted identity propagation works, enabling business users to bring their workforce identities for delegated access into AWS without needing IAM credentials or cloud backends.</p> 
<p>You set up custom applications within Okta (as the directory service) and IAM Identity Center and deployed the required application IAM roles and federation into your AWS account. You learned the architecture of this solution and how to use it in an application.</p> 
<p>Through this setup, you can use the AWS CLI to interact with AWS services such as S3 Access Grants or Athena on behalf of the directory user without the having to configure an IAM user or credentials for them.</p> 
<p>The code used in this post is also published as an AWS Sample, which can be found on <a href="https://github.com/aws-samples/access-aws-services-programmatically-using-tip" rel="noopener" target="_blank">GitHub</a> and is meant to serve as inspiration for integrating custom or third-party applications with trusted identity propagation.</p> 
<p>&nbsp;<br />If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.</p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image" style="padding-top: 15px; padding-bottom: 15px;"> 
   <img alt="Roberto Migli" class="aligncenter size-full wp-image-31211" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/10/05/Roberto-Migli.jpg" width="120" /> 
  </div> 
  <p> <span class="lb-h4">Roberto Migli</span><br /> Roberto is a Principal Solutions Architect at AWS. He supports global financial services customers, focusing on security and identity and access management. In his free time, he enjoys building electronic gadgets, learning about space, and spending time with his family. </p>
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image" style="padding-top: 15px; padding-bottom: 15px;"> 
   <img alt="Alessandro Fior" class="aligncenter size-full wp-image-34612" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/06/26/alefior.jpg" width="120" /> 
  </div> 
  <p> <span class="lb-h4">Alessandro Fior</span><br /> Alessandro is a Sr. Data Architect at AWS Professional Services. He is passionate about designing and building modern and scalable data platforms that boost companies’ efficiency in extracting value from their data. </p>
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image" style="padding-top: 15px; padding-bottom: 15px;"> 
   <img alt="Bruno Corijn" class="aligncenter size-full wp-image-34613" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/06/26/bcorijn.jpg" width="120" /> 
  </div> 
  <p> <span class="lb-h4">Bruno Corijn</span><br /> Bruno is a Cloud Infrastructure Architect at AWS based out of Brussels, Belgium. He works with Public Sector customers to accelerate and innovate in their AWS journey, bringing a decade of experience in application and infrastructure modernisation. In his free time he loves to ride and tinker with bikes. </p>
 </div> 
</footer>
