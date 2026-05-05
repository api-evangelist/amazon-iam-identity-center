---
title: "Managing identity source transition for AWS IAM Identity Center"
url: "https://aws.amazon.com/blogs/security/managing-identity-source-transition-for-aws-iam-identity-center/"
date: "Wed, 25 Sep 2024 17:53:12 +0000"
author: "Xiaoxue Xu"
feed_url: "https://aws.amazon.com/blogs/security/tag/aws-iam-identity-center/feed/"
---
<p><a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html" rel="noopener" target="_blank">AWS IAM Identity Center</a> manages user access to <a href="https://aws.amazon.com" rel="noopener" target="_blank">Amazon Web Services (AWS)</a> resources, including both <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/manage-your-accounts.html" rel="noopener" target="_blank">AWS accounts</a> and <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/manage-your-applications.html" rel="noopener" target="_blank">applications</a>. You can use IAM Identity Center to create and manage user identities within the Identity Center identity store or to connect seamlessly to other identity sources.</p> 
<p>Organizations might change the configuration of their identity source in IAM Identity Center for various reasons. These include switching identity providers (IdPs), expanding their identity footprint, adopting new features, and so on. These transitions can disrupt user access and require planning to minimize downtime.</p> 
<p>In this blog post, we walk you through the process of switching from one identity source to another and provide sample code that you can use to assist with the transition.</p> 
<h2>Background</h2> 
<p>The identity source configured in IAM Identity Center determines where users and groups are created and managed. Each organization can connect to only one identity source at a time. Identity Center supports three main identity source options:</p> 
<ol> 
 <li><a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/manage-your-identity-source-sso.html" rel="noopener" target="_blank">Identity Center directory</a>: This is the default identity store for IAM Identity Center. You can use it to directly create and administer your users and groups within Identity Center without relying on an external provider.</li> 
 <li><a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/manage-your-identity-source-ad.html" rel="noopener" target="_blank">Active Directory</a>: You can configure integration with an on-premises Active Directory or with <a href="https://docs.aws.amazon.com/directoryservice/latest/admin-guide/directory_microsoft_ad.html" rel="noopener" target="_blank">AWS Managed Microsoft AD</a> using <a href="https://aws.amazon.com/directoryservice/" rel="noopener" target="_blank">AWS Directory Service</a>. This integration enables you to use your existing Active Directory identities and group memberships.</li> 
 <li><a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/manage-your-identity-source-idp.html" rel="noopener" target="_blank">External IdP</a>: You can continue using your current third-party IdPs that support SAML 2.0, such as Okta Universal Directory or Microsoft Entra ID (formerly Azure AD).</li> 
</ol> 
<p>Understanding these identity source options can help you choose the source that best fits your user management needs based on your existing infrastructure and authentication requirements.</p> 
<p>Figure 1 explains the flow of users and groups accessing AWS resources. This access is granted through the following:</p> 
<ul> 
 <li>Assignment of <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/permissionsetsconcept.html" rel="noopener" target="_blank">permission sets</a> to users and groups in your directory, which enables assume role access to AWS accounts.</li> 
 <li>Assignment of applications to users and groups in your directory, providing access to both <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/awsapps.html" rel="noopener" target="_blank">AWS managed applications</a> and <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/customermanagedapps.html" rel="noopener" target="_blank">customer managed applications</a>.</li> 
</ul> 
<p style="line-height: 1.25em;"></p>
<div class="wp-caption aligncenter" id="attachment_35777" style="width: 790px;">
 <img alt="Figure 1: Granting access to AWS resources for users and groups managed by an identity source in IAM Identity Center" class="size-full wp-image-35777" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/16/img1-4.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-35777">Figure 1: Granting access to AWS resources for users and groups managed by an identity source in IAM Identity Center</p>
</div>
<p></p> 
<p>When you change the identity source, the work required varies depending on the original and new sources. <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/manage-your-identity-source-considerations.html" rel="noopener" target="_blank">AWS documentation</a> details these considerations. In minimal impact scenarios, assignments remain intact, although you need to force password resets or verify correct assertions from the new source. More disruptive scenarios delete users, groups, and their assignments. In those scenarios, you need to restore the deleted entries after changing the identity source.</p> 
<h2>Sample deployment</h2> 
<p>This deployment covers permission sets and application assignments’ backup and restore. These scripts associate assignments with unique user attributes such as <a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/API_User.html" rel="noopener" target="_blank">UserName</a>, and group attributes such as <a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/API_Group.html" rel="noopener" target="_blank">DisplayName</a>. Attributes that might change during the user and group restoration process, such as <code style="color: #000000;">UserId</code> and <code style="color: #000000;">GroupId</code>, aren’t used.</p> 
<p>What isn’t covered includes users, groups, permission sets, and applications backup and restore.</p> 
<ul> 
 <li>Users and groups backup and restore aren’t covered because they depend heavily on the format of the source and target IdPs.</li> 
 <li>Because we’re working with identity source switching, the permission sets will remain unchanged and applications will not be deleted.</li> 
 <li>If you’re changing IdPs as part of an AWS Region , the IAM Identity Center instance will be deleted. The applications and permission sets will be deleted in addition to assignments. In this case, you must redeploy the applications. See <a href="https://aws.amazon.com/blogs/security/how-to-automate-the-review-and-validation-of-permissions-for-users-and-groups-in-aws-iam-identity-center/" rel="noopener" target="_blank">How to automate the review and validation of permissions for users and groups in AWS IAM Identity Center</a> for information about backing up permission sets.</li> 
</ul> 
<p>The sample scripts and detailed steps are available on <a href="https://github.com/aws-samples/manage-identity-source-transition-for-aws-iam-identity-center" rel="noopener" target="_blank">GitHub</a>.</p> 
<blockquote>
 <p><strong>Note</strong>: This solution is available in the GitHub <a href="https://github.com/aws-samples" rel="noopener" target="_blank">aws-samples</a> repository. You can report bugs or make feature requests through <a href="https://github.com/aws-samples/manage-identity-source-transition-for-aws-iam-identity-center/issues" rel="noopener" target="_blank">GitHub Issues</a>. The builders of this solution can help with GitHub issues. <a href="https://aws.amazon.com/premiumsupport/plans/enterprise/" rel="noopener" target="_blank">Enterprise Support</a> customers can reach out to their Technical Account Manager (TAM) for further questions or feature requests.</p>
</blockquote> 
<h2>Walkthrough</h2> 
<p>In this section, we walk you through the process of transitioning to a new identity source in IAM Identity Center.</p> 
<h3>Step 1: Backup users, groups, and assignments from the current identity source</h3> 
<p>This step is critical to preserve users’ information and their associated access scope.</p> 
<p>How to backup users and groups:</p> 
<ul> 
 <li>When using the IAM Identity Center directory as your identity source, use <a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/API_ListUsers.html#singlesignon-ListUsers-response-Users" rel="noopener" target="_blank">ListUsers</a>, <a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/API_ListGroups.html" rel="noopener" target="_blank">ListGroups</a>, and <a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/API_ListGroupMembershipsForMember.html" rel="noopener" target="_blank">ListGroupMembershipsForMember</a> to back up metadata and attributes.</li> 
 <li>When using sources such as Active Directory or an external IdP, you can use compatible tools such as <a href="https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps" rel="noopener" target="_blank">Active Directory module for Windows PowerShell</a> and third-party scripts to <a href="https://activedirectorypro.com/export-users-active-directory/" rel="noopener" target="_blank">back up users</a> and <a href="https://activedirectorypro.com/powershell-export-active-directory-group-members/" rel="noopener" target="_blank">back up groups</a>.</li> 
</ul> 
<blockquote>
 <p><strong>Note</strong>: For some external IdPs, there are native integrations with Active Directory, such as <a href="https://help.okta.com/en-us/content/topics/directory/ad-agent-main.htm" rel="noopener" target="_blank">Okta AD integration</a> and <a href="https://docs.pingidentity.com/r/en-us/pingoneforenterprise/p14e_connect_adc" rel="noopener" target="_blank">Ping One AD Connect</a>. You can set up a native integration and sync users and groups data without needing to backup and restore that data.</p>
</blockquote> 
<p>Assignments can be backed up by running the <a href="https://github.com/aws-samples/manage-identity-source-transition-for-aws-iam-identity-center/blob/main/backup.py" rel="noopener" target="_blank">backup.py</a> file from GitHub. Replace <code><span style="color: #ff0000; font-style: italic;">&lt;IDC_STORE_ID&gt;</span></code> with your Identity Store ID (it looks like <em>d-1234567890</em>), and replace <code><span style="color: #ff0000; font-style: italic;">&lt;IDC_ARN&gt;</span></code> with the Amazon Resource Name (ARN) for your IAM Identity Center instance.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">python3 backup.py --idc-id <span style="color: #ff0000; font-style: italic;">&lt;IDC_STORE_ID&gt;</span> --idc-arn <span style="color: #ff0000; font-style: italic;">&lt;IDC_ARN&gt;</span></code></pre> 
</div> 
<p>This script uses both <a href="https://docs.aws.amazon.com/singlesignon/latest/APIReference/welcome.html" rel="noopener" target="_blank">IAM Identity Center APIs</a> and <a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/welcome.html" rel="noopener" target="_blank">Identity Store APIs</a> as shown in Figure 2 that generates permission set assignments backup files (<code style="color: #000000;">UserAssignments.json</code> and <code style="color: #000000;">GroupAssignments.json</code>) and an application assignments backup file (<code style="color: #000000;">AppAssignments.json</code>).</p> 
<blockquote>
 <p><strong>Note:</strong> If you are using the <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/disableuser.html" rel="noopener" target="_blank">disable user access</a> feature for IAM Identity Center, the ListUsers API will not be able to display the disabled status. You will need to go into <a href="https://aws.amazon.com/console/" rel="noopener" target="_blank">AWS Management Console</a> to manually inspect the disabled status. You can choose to:</p> 
 <ol> 
  <li>Remove disabled users from your backup files.</li> 
  <li>Re-disable the inspected users before restoring assignments in Step 4.</li> 
 </ol> 
</blockquote> 
<div class="wp-caption aligncenter" id="attachment_35782" style="width: 790px;">
 <img alt="Figure 2: APIs used for backing up assignments" class="size-full wp-image-35782" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/16/img2-4.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-35782">Figure 2: APIs used for backing up assignments</p>
</div> 
<h3>Step 2: Restore and validate the backed-up users and groups in the target identity source</h3> 
<p>The target will become the new authoritative identity source. When done, verify that the group memberships and attributes have been correctly transferred.</p> 
<ul> 
 <li>If the target is an IAM Identity Center directory, use APIs such as <a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/API_CreateUser.html" rel="noopener" target="_blank">CreateUser</a>, <a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/API_CreateGroup.html" rel="noopener" target="_blank">CreateGroup</a>, and <a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/API_CreateGroupMembership.html" rel="noopener" target="_blank">CreateGroupMembership</a> to restore from the previous backup file.</li> 
 <li>If the target is Active Directory or an external IdP, use the corresponding native import features or integration tools to restore.</li> 
</ul> 
<h3>Step 3: Configure IAM Identity Center to connect to the new identity source and synchronize users and groups</h3> 
<p>Update your IAM Identity Center configuration to point to the new source. If applicable, use tools such as <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/provision-users-from-ad-configurable-ADsync.html" rel="noopener" target="_blank">configurable AD sync</a> or <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/provision-automatically.html" rel="noopener" target="_blank">automatic provisioning with SCIM</a> to synchronize your restored identities.</p> 
<blockquote>
 <p><strong>WARNING</strong>: While the directory is being rebuilt, your users will not have access to AWS accounts or applications through IAM Identity Center until all assignments are restored in Step 4.</p>
</blockquote> 
<h3>Step 4: Restore assignments to users and groups in the new identity source</h3> 
<p>The APIs used to restore assignments are as shown in Figure 3.</p> 
<div class="wp-caption aligncenter" id="attachment_35783" style="width: 790px;">
 <img alt="Figure 3: APIs used for restoring assignments" class="size-full wp-image-35783" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/16/img3-3.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-35783">Figure 3: APIs used for restoring assignments</p>
</div> 
<p>Assignments can be restored by running the <a href="https://github.com/aws-samples/manage-identity-source-transition-for-aws-iam-identity-center/blob/main/restore.py" rel="noopener" target="_blank">restore.py</a> file from GitHub. Replace <code><span style="color: #ff0000; font-style: italic;">&lt;NEW_IDC_STORE_ID&gt;</span></code> with your newly configured Identity Store ID (it looks like <em>d-1234567890</em>) and replace <code><span style="color: #ff0000; font-style: italic;">&lt;IDC_ARN&gt;</span></code> with the ARN for your IAM Identity Center instance.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">python3 restore.py --idc-id <span style="color: #ff0000; font-style: italic;">&lt;NEW_IDC_STORE_ID&gt;</span> --idc-arn <span style="color: #ff0000; font-style: italic;">&lt;IDC_ARN&gt;</span></code></pre> 
</div> 
<p>This script uses the APIs illustrated in Figure 2 and picks up backup files (<code style="color: #000000;">UserAssignments.json</code>, <code style="color: #000000;">GroupAssignments.json</code>, and <code style="color: #000000;">AppAssignments.json</code>) from Step 1 by default. Account permission set assignment results are automatically retrieved five times using exponential backoff. If the result is other than SUCCEEDED after five retries, the principal ID will be marked as failed and exported in error logs.</p> 
<blockquote>
 <p><strong>Note</strong>: For AWS managed applications that maintain a separate identity source, using the <a href="https://docs.aws.amazon.com/singlesignon/latest/APIReference/API_CreateApplicationAssignment.html" rel="noopener" target="_blank">CreateApplicationAssignments</a> API to restore application assignments will not preserve user access. These applications typically have dependencies on the original identity source ID, or dependencies on <code style="color: #000000;">UserId</code> and <code style="color: #000000;">GroupId</code> from the original identity source. This dependency is represented by importing users or groups from IAM Identity Center during the AWS managed application creation process. Example AWS managed applications include <a href="https://aws.amazon.com/sagemaker/studio/" rel="noopener" target="_blank">Amazon SageMaker Studio</a> and <a href="https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/what-is.html" rel="noopener" target="_blank">Amazon Q Developer</a>. These applications must be restored on a case-by-case basis and can require redeployment of the application.</p>
</blockquote> 
<h3>Step 5: Validate user access using the new identity source</h3> 
<p>Make sure that users can still access the expected accounts and applications.</p> 
<h2>Conclusion</h2> 
<p>Transitioning your identity source in IAM Identity Center requires careful planning and implementation. This post outlined the steps to manage this transition. By following these steps, you can streamline the transition process, providing a smooth and efficient transfer of user access with minimal downtime. To get started, see the <a href="https://github.com/aws-samples/manage-identity-source-transition-for-aws-iam-identity-center" rel="noopener" target="_blank">GitHub</a> repository. For related posts, visit the <a href="https://aws.amazon.com/security/blogs" rel="noopener" target="_blank">AWS Security Blog</a> channel and search for <a href="https://aws.amazon.com/security/blogs/?awsf.blog-post-types-filter=*all&amp;awsf.blog-learning-levels-filter=*all&amp;awsf.blog-categories-filter=*all&amp;filtered-posts.q=AWS%2BIAM%2BIdentity%2BCenter&amp;filtered-posts.q_operator=AND" rel="noopener" target="_blank">IAM Identity Center</a>.</p> 
<p>&nbsp;<br />If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.<br />&nbsp;</p> 
<footer> 
 <div class="blog-author-box">
  <img alt="Xiaoxue Xu" class="alignleft size-full wp-image-33350" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/16/xuxiaoxu.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Xiaoxue Xu</span>
  <br />Xiaoxue Xu is a Solutions Architect for AWS based in Toronto. She primarily works with Financial Services customers to help secure their workload and design scalable solutions on the AWS Cloud.
 </div> 
 <div class="blog-author-box">
  <img alt="Kanwar Bajwa" class="alignleft size-full wp-image-33350" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/16/bajwkanw.png" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Kanwar Bajwa</span>
  <br />Kanwar Bajwa is a Principal Technical Account Manager at AWS who works with customers to optimize their use of AWS services and achieve their business objectives.
 </div> 
 <div class="blog-author-box">
  <img alt="Yee Fei Ooi" class="alignleft size-full wp-image-33350" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/16/yeefei.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Yee Fei Ooi</span>
  <br />Yee Fei is a Solutions Architect supporting independent software vendor (ISV) customers in Singapore and is part of the Containers TFC. She enjoys helping her customers to grow their businesses and build solutions that automate, innovate, and improve efficiency.
 </div> 
</footer>
