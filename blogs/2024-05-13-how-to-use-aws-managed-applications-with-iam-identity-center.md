---
title: "How to use AWS managed applications with IAM Identity Center: Enable Amazon Q without migrating existing IAM federation flows"
url: "https://aws.amazon.com/blogs/security/how-to-use-aws-managed-applications-with-iam-identity-center/"
date: "Mon, 13 May 2024 19:55:00 +0000"
author: "Liam Wadman"
feed_url: "https://aws.amazon.com/blogs/security/tag/aws-iam-identity-center/feed/"
---
<blockquote>
 <p><strong>June 9, 2025</strong>: We added a section on how to use a service control policy to block permission sets from being used in your organization’s member accounts.</p>
</blockquote> 
<hr /> 
<p><a href="https://aws.amazon.com/iam/identity-center" rel="noopener" target="_blank">AWS IAM Identity Center</a> is the preferred way to provide workforce access to <a href="https://aws.amazon.com/" rel="noopener" target="_blank">Amazon Web Services (AWS)</a> accounts, and enables you to provide workforce access to many AWS managed applications, such as <a href="https://aws.amazon.com/codewhisperer" rel="noopener" target="_blank">Amazon Q Developer (Formerly known as Code Whisperer)</a>.</p> 
<p>As we continue to release more AWS managed applications, customers have told us they want to onboard to IAM Identity Center to use AWS managed applications, but some aren’t ready to migrate their existing IAM federation for AWS account management to Identity Center. </p> 
<p>In this blog post, I’ll show you how you can enable Identity Center and use AWS managed applications—such as Amazon Q—without migrating existing IAM federation flows to Identity Center. While the example in this post uses <a href="https://aws.amazon.com/q/developer/" rel="noopener" target="_blank">Amazon Q Developer</a>, the same approach and guidance applies to Amazon Q Business and other AWS managed applications integrated with Identity Center.</p> 
<h2>A recap on AWS managed applications and trusted identity propagation</h2> 
<p>Just before re:Invent 2023, AWS launched <em>trusted identity propagation</em>, a technology that allows you to use a user’s identity and groups when accessing AWS services. This allows you to assign permissions directly to users or groups, rather than model entitlements in <a href="https://aws.amazon.com/iam" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a>. This makes permissions management simpler for users. For example, with trusted identity propagation, you can grant&nbsp;users and groups access to specific <a href="https://aws.amazon.com/redshift" rel="noopener" target="_blank">Amazon Redshift</a> clusters without modeling all possible unique combinations of permissions in IAM. Trusted identity propagation is available today for Redshift and <a href="https://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a>, with more services and features coming over time.</p> 
<p>In 2023, we released <a href="https://aws.amazon.com/q/developer/" rel="noopener" target="_blank">Amazon Q Developer</a>, which is integrated with IAM Identity Center, generally available as an AWS managed application. When you’re using Amazon Q Developer outside of AWS in integrated development environments (IDEs) such as Microsoft Visual Studio Code, Identity Center is used to sign in to Amazon Q Developer.</p> 
<p>Amazon Q Developer is one of many AWS managed applications that are integrated with the OAuth 2.0 functionality of IAM Identity Center, and it doesn’t use IAM credentials to access the Q Developer service from within your IDEs.&nbsp;AWS managed applications and trusted identity propagation don’t require you to use the permission sets feature of Identity Center and instead use OpenID Connect to grant your workforce access to AWS applications and features.</p> 
<h2>IAM Identity Center for AWS application access only</h2> 
<p>In the following section, we use IAM Identity Center to sign in to Amazon Q Developer as an example of an AWS managed application.</p> 
<h3>Prerequisites</h3> 
<ul> 
 <li>The steps in this post require that you have administrative level access to an organization in <a href="https://aws.amazon.com/organizations/" rel="noopener" target="_blank">AWS Organizations</a>.</li> 
 <li>Specific <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/get-started-prereqs-considerations.html" rel="noopener" target="_blank">prerequisites and considerations</a> for deploying IAM Identity Center can be found in the <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/get-set-up-for-idc.html" rel="noopener" target="_blank">documentation.</a></li> 
</ul> 
<h3>Step 1: Enable an organization instance of IAM Identity Center</h3> 
<p>To begin, you must enable an organization instance of IAM Identity Center.&nbsp;While it’s possible to use IAM Identity Center without an <a href="https://aws.amazon.com/organizations/" rel="noopener" target="_blank">AWS Organizations</a> organization, we generally recommend that customers operate with such an organization.</p> 
<p>The IAM Identity Center documentation provides the steps to <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/organization-instances-identity-center.html" rel="noopener" target="_blank">enable an organizational instance of IAM Identity Center</a>, as well as prerequisites and considerations. One consideration I would emphasize here is the <em>identity source.</em> We recommend, wherever possible, that you integrate with an external identity provider (IdP), because this provides the most flexibility and allows you to take advantage of the advanced security features of modern identity services.</p> 
<p>IAM Identity Center is available at no additional cost.</p> 
<blockquote>
 <p><strong>Note: </strong>In late 2023, AWS launched <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/account-instances-identity-center.html" rel="noopener" target="_blank"><em>account instances</em> for IAM Identity Center</a>. Account instances allow you to create additional Identity Center instances within member accounts of your organization. Wherever possible, we recommend that customers use an organization instance of IAM Identity Center to give them a centralized place to manage their identities and permissions.&nbsp;AWS recommends account instances when you want to perform a proof of concept using Identity Center, when there isn’t a central IdP or directory that contains all the identities you want to use on AWS and you want to use AWS managed applications with distinct directories, or when your AWS account is a member of an organization in AWS Organizations that is managed by another party and you don’t have access to set up an organization instance.</p>
</blockquote> 
<h3>Step 2: Set up your IdP and&nbsp;synchronize identities and groups</h3> 
<p>After you’ve enabled your IAM Identity Center instance, you need to set up your instance to work with your chosen IdP and synchronize your identities and groups. The&nbsp;<a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/tutorials.html" rel="noopener" target="_blank">IAM Identity Center documentation includes examples</a> of how to do this with many popular IdPs.</p> 
<p>After your identity source is connected, IAM Identity Center can act as the single source of identity and authentication for AWS managed applications, bridging your external identity source and AWS managed applications. You don’t have to create a bespoke relationship between each AWS application and your IdP, and you have a single place to manage user permissions.</p> 
<h3>Step 3: Set up delegated administration for IAM Identity Center</h3> 
<p>As a best practice, we recommend that you only access the management account of your <a href="https://aws.amazon.com/organizations/" rel="noopener" target="_blank">AWS Organizations</a> organization when absolutely necessary. IAM Identity Center supports <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/delegated-admin.html" rel="noopener" target="_blank">delegated administration</a>, which allows you to manage Identity Center from a member account of your organization.</p> 
<h4>To set up delegated administration</h4> 
<ol> 
 <li>Go to the AWS Management Console and navigate to IAM Identity Center.</li> 
 <li>In the left navigation pane, select <strong>Settings</strong>. Then select the <strong>Management</strong> tab and choose <strong>Register account</strong>.</li> 
 <li>From the menu that follows, select the AWS account that will be used for delegated administration for IAM Identity Center. Ideally, this member account is dedicated solely to the purpose of administrating IAM Identity Center and is only accessible to users who are responsible for maintaining IAM Identity Center.</li> 
</ol> 
<p style="line-height: 1.25em;"></p>
<div class="wp-caption aligncenter" id="attachment_34139" style="width: 790px;">
 <img alt="Figure 1: Set up delegated administration" class="size-full wp-image-34139" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/05/10/img1-3.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-34139">Figure 1: Set up delegated administration</p>
</div>
<p></p> 
<h3>Step 4: Configure Amazon Q Developer</h3> 
<p>You now have IAM Identity Center set up with the users and groups from your directory, and you’re ready to configure AWS managed applications with IAM Identity Center. From a member account within your organization, you can now enable Amazon Q Developer. This can be any member account in your organization and should not be the one where you set up delegated administration of IAM Identity Center, or the management account.</p> 
<blockquote>
 <p><strong>Note:</strong> If you’re doing this step immediately after configuring IAM Identity Center with an external IdP with SCIM synchronization, be aware that the users and groups from your external IdP might not yet be synchronized to Identity Center by your external IdP. Identity Center updates user information and group membership as soon as the data is received from your external IdP. How long it takes to finish synchronizing after the data is received depends on the number of users and groups being synchronized to Identity Center.</p>
</blockquote> 
<h4>To enable Amazon Q Developer </h4> 
<ol> 
 <li>Open the Amazon Q Developer console. This will take you to the setup for Amazon Q Developer. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_34140" style="width: 750px;">
   <img alt="Figure 2: Open the Amazon Q Developer console" class="size-full wp-image-34140" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/05/10/img2-3.png" width="740" />
   <p class="wp-caption-text" id="caption-attachment-34140">Figure 2: Open the Amazon Q Developer console</p>
  </div><p></p> </li> 
 <li>Choose <strong>Subscribe to Amazon Q</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_34141" style="width: 750px;">
   <img alt="Figure 3: The Amazon Q developer console" class="size-full wp-image-34141" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/05/10/img3-3.png" width="740" />
   <p class="wp-caption-text" id="caption-attachment-34141">Figure 3: The Amazon Q developer console</p>
  </div><p></p> </li> 
 <li>You’ll be taken to the <strong>Amazon Q</strong> console. Choose <strong>Subscribe</strong> to subscribe to Amazon Q Developer Pro. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_34142" style="width: 643px;">
   <img alt="Figure 4: Subscribe to Amazon Q Developer Pro" class="size-full wp-image-34142" height="368" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/05/10/img4-3.png" width="633" />
   <p class="wp-caption-text" id="caption-attachment-34142">Figure 4: Subscribe to Amazon Q Developer Pro</p>
  </div><p></p> </li> 
 <li>After choosing <strong>Subscribe</strong>, you will be prompted to select users and groups you want to enroll for Amazon Q Developer. Select the users and groups you want and then choose <strong>Assign</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_34143" style="width: 658px;">
   <img alt="Figure 5: Assign user and group access to Amazon Q Developer" class="size-full wp-image-34143" height="633" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/05/10/img5-1.png" style="border: 1px solid #bebebe;" width="648" />
   <p class="wp-caption-text" id="caption-attachment-34143">Figure 5: Assign user and group access to Amazon Q Developer</p>
  </div><p></p> </li> 
</ol> 
<p>After you perform these steps, the setup of Amazon Q Developer as an AWS managed application is complete, and you can now use Amazon Q Developer. No additional configuration is required within your external IdP or on-premises Microsoft Active Directory, and no additional user profiles have to be created or synchronized to Amazon Q Developer. </p> 
<blockquote>
 <p><strong>Note</strong>: There are charges associated with using the Amazon Q Developer service.</p>
</blockquote> 
<h3>Step 5: Set up Amazon Q Developer in the IDE</h3> 
<p>Now that Amazon Q Developer is configured, users and groups that you have granted access to can <a href="https://docs.aws.amazon.com/codewhisperer/latest/userguide/setting-up.html" rel="noopener" target="_blank">use Amazon Q Developer from their supported IDE</a>.</p> 
<p>In their IDE, a user can sign in to Amazon Q Developer by entering the start URL and AWS Region and choosing <strong>Sign in</strong>. Figure 6 shows what this looks like in Visual Studio Code. The Amazon Q extension for Visual Studio Code is available to download within Visual Studio Code.</p> 
<div class="wp-caption aligncenter" id="attachment_34144" style="width: 558px;">
 <img alt="Figure 6: Signing in to the Amazon Q Developer extension in Visual Studio Code" class="size-full wp-image-34144" height="711" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/05/10/img6.png" width="548" />
 <p class="wp-caption-text" id="caption-attachment-34144">Figure 6: Signing in to the Amazon Q Developer extension in Visual Studio Code</p>
</div> 
<p>After choosing <strong>Use with Pro license</strong>, and entering their Identity Center’s start URL and Region, the user will be directed to authenticate with IAM Identity Center and grant the Amazon Q Developer application access to use the Amazon Q Developer service.</p> 
<p>When this is successful, the user will have the Amazon Q Developer functionality available in their IDE. This was achieved without migrating existing federation or AWS account access patterns to IAM Identity Center.</p> 
<h2>Clean up</h2> 
<p>If you don’t wish to continue using IAM Identity Center or Amazon Q Developer, you can delete the Amazon Q Developer Profile and Identity Center instance within their respective consoles, within the AWS account they are deployed into. Deleting your Identity Center instance won’t make changes to existing federation or AWS account access that is not done through IAM Identity Center.</p> 
<h2>Blocking the use of permissions sets in member accounts</h2> 
<p>If you are adopting IAM Identity Center primarily for using AWS managed applications, you can use an additional control with a <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html" rel="noopener" target="_blank">service control policy</a> to help prevent the use of permission sets in your AWS organization member accounts. </p> 
<p>The following policy will deny IAM roles created for use by permission sets from taking IAM actions on your AWS resources. The IAM path <code style="color: #000000;">/aws-reserved/sso.amazonaws.com/</code> is used by identity center permission sets, and targeting that with the <code style="color: #000000;">aws:PrincipalArn</code> <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-principalarn" rel="noopener" target="_blank">condition key</a> makes this policy statement apply to those IAM roles and not others.</p> 
<div class="hide-language"> 
 <pre><code class="lang-text">{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "ArnLike": {
          "aws:PrincipalArn": "arn:aws:iam::*:role/aws-reserved/sso.amazonaws.com/*"
        }
      }
    }
  ]
}
</code></pre> 
</div> 
<h2>Conclusion</h2> 
<p>In this post, we talked about some recent significant launches of AWS managed applications and features that integrate with IAM Identity Center and discussed how you can use these features without migrating your AWS account management to permission sets. We also showed how you can set up Amazon Q Developer with IAM Identity Center. While the example in this post uses Amazon Q Developer, the same approach and guidance applies to Amazon Q Business and other AWS managed applications integrated with Identity Center.</p> 
<p>To learn more about the benefits and use cases of IAM Identity Center, <a href="https://aws.amazon.com/iam/identity-center/" rel="noopener" target="_blank">visit the product page,</a> and to learn more about Amazon Q Developer, visit the <a href="https://aws.amazon.com/q/developer/" rel="noopener" target="_blank">Amazon Q Developer product page</a>.</p> 
<p>&nbsp;<br />If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.</p> 
<p><strong>Want more AWS Security news? Follow us on <a href="https://twitter.com/AWSsecurityinfo" rel="noopener noreferrer" target="_blank" title="Twitter">X</a>.</strong></p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Liam Wadman" class="aligncenter size-full wp-image-27515" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2022/11/01/liwadman.jpg" width="120" /> 
  </div> 
  <p class="lb-h4">Liam Wadman</p> 
  <p>Liam is a Senior Solutions Architect with the Identity Solutions team. When he’s not building exciting solutions on AWS or helping customers, he’s often found in the mountains of British Columbia on his mountain bike. Liam points out that you cannot spell LIAM without IAM.</p> 
 </div> 
</footer>
