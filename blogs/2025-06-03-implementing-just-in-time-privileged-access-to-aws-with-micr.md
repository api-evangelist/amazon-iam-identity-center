---
title: "Implementing just-in-time privileged access to AWS with Microsoft Entra and AWS IAM Identity Center"
url: "https://aws.amazon.com/blogs/security/implementing-just-in-time-privileged-access-to-aws-with-microsoft-entra-and-aws-iam-identity-center/"
date: "Tue, 03 Jun 2025 16:45:37 +0000"
author: "Rodney Underkoffler"
feed_url: "https://aws.amazon.com/blogs/security/tag/aws-iam-identity-center/feed/"
---
<blockquote>
 <p><strong>June 19, 2025</strong>: We made a correction to the windows of access that a user could have when using this solution.</p>
</blockquote> 
<hr /> 
<p>Controlling access to your privileged and sensitive resources is critical for all AWS customers. Preventing direct human interaction with services and systems through automation is the primary means of accomplishing this. For those infrequent times when automation is not yet possible or implemented, providing a secure method for temporary elevated access is the next best option. In a privileged access management solution, there are several elements that should be included:</p> 
<ul> 
 <li>User access should follow the principle of least privileged</li> 
 <li>Users should be granted only the minimum amount of access required to perform their job duties</li> 
 <li>Access granted should persist only for the time necessary to perform the assigned tasks</li> 
 <li>The solution should include: 
  <ul> 
   <li>An eligibility process for granting access</li> 
   <li>An approval process for granting access</li> 
   <li>Auditing of the access grants and activities taken</li> 
  </ul> </li> 
</ul> 
<p>Entra Privileged Identity Management (PIM) is a third-party solution that provides dynamic group management, access control, and audit capabilities that integrate with <a href="https://aws.amazon.com/iam/identity-center" rel="noopener" target="_blank">AWS IAM Identity Center</a>.</p> 
<p>In this post, we show you how to configure just-in-time access to AWS using Entra PIM’s integration with IAM Identity Center.</p> 
<h2 id="just-in-time-privileged-access-with-entra-pim-and-iam-identity-center">Just-in-time privileged access with Entra PIM and IAM Identity Center</h2> 
<p>Privileged Identity Management is a Microsoft Entra ID feature that enables management, control, and access monitoring of your important cloud resources. There are many different configuration options when it comes to eligibility and assignment to privileged security groups, including time-bound access with start and end dates, multi-factor authentication (MFA) enforcement, justification tracking, and so on. You can read more about those options in <a href="https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-configure" rel="noopener" target="_blank">Microsoft’s product documentation</a>.</p> 
<p>Figure 1 shows the just-in-time access solution powered by Entra PIM group activation requests. In this solution, Entra PIM is integrated with IAM Identity Center to provide temporary, limited access to AWS resources based on user requests and approvals. Entra ID users can submit requests for specific access to specific AWS permissions sets, which are then automatically granted for a set duration.</p> 
<div class="wp-caption aligncenter" id="attachment_38512" style="width: 1022px;">
 <img alt="Figure 1 – Entra PIM solution integrated with IAM Identity Center" class="size-full wp-image-38512" height="477" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/28/Just-in-time-privileged-access-1.png" width="1012" />
 <p class="wp-caption-text" id="caption-attachment-38512">Figure 1 – Entra PIM solution integrated with IAM Identity Center</p>
</div> 
<h2 id="prerequisites">Prerequisites</h2> 
<p>To try the solution described in this post, you need to have the following in place:</p> 
<ul> 
 <li>An AWS account with an organization instance of IAM Identity Center enabled. For more information, see <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/step1.html" rel="noopener" target="_blank">What is IAM Identity Center</a>.</li> 
 <li>An Azure account subscription with Entra ID P1 or P2 licensing.</li> 
 <li>Entra ID as an external identity provider (IdP) for IAM Identity Center as described in&nbsp;<a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/idp-microsoft-entra.html#step1-entra-microsoft-prep" rel="noopener" target="_blank">Configure SAML and SCIM with Microsoft Entra ID and IAM Identity Center</a>. Follow Steps 1.1, 3.1, 3.2, and 3.3.</li> 
 <li>A sample user in Entra ID as described in the step 4.1 of <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/idp-microsoft-entra.html#step4-entra-scim" rel="noopener" target="_blank">Step 4: Configure and test your SCIM synchronization</a> or use an existing user within Entra ID for testing.</li> 
 <li>Automatic provisioning enabled in Entra ID and IAM Identity Center. Follow Steps 4.2 and 4.3 in this&nbsp;<a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/idp-microsoft-entra.html#step4-entra-scim" rel="noopener" target="_blank">Step 4: Configure and test your SCIM synchronization</a>.</li> 
</ul> 
<h2 id="step-by-step-configuration">Step-by-step configuration</h2> 
<p>In the following steps, you create configurations to enable Entra PIM for Groups to automatically assign users to groups based on approval criteria. The groups will be Entra ID security groups that use direct assignment. Note that, at the time of this writing, <a href="https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/concept-pim-for-groups#relationship-between-role-assignable-groups-and-pim-for-groups" rel="noopener" target="_blank">dynamic groups and groups that you have synchronized from a self-managed Active Directory cannot be used with Microsoft Entra PIM</a>. While it might be possible to also populate these groups using a third-party synchronization tool, for the purposes of this exercise, we assume that administration is occurring solely within Entra ID.</p> 
<p>In the example scenario, the role corresponds to a specific job function within your organization. We use a group called <em>AWS – Amazon EC2 Admin</em>, which corresponds to a DevOps on-call site reliability engineer (SRE) lead.</p> 
<h3 id="step-1-create-a-group-representing-a-specific-privilege-level.">Step 1: Create a group representing a specific privilege level.</h3> 
<p>Create a group in Entra ID that represents a specific privilege level that your employees can request for access to the AWS Management Console.</p> 
<ol type="1"> 
 <li>Sign in to the&nbsp;<a href="https://entra.microsoft.com/" rel="noopener" target="_blank">Microsoft Entra admin center</a> with your credentials.</li> 
 <li>Select <strong>Groups</strong> and then&nbsp;<strong>All groups</strong>.</li> 
 <li>Choose&nbsp;<strong>New group</strong>.</li> 
 <li>Specify <strong>Security</strong> in the <strong>Group type</strong> dropdown list. 
  <ul> 
   <li>In the&nbsp;<strong>Group name</strong> field, enter&nbsp;<code style="color: #000000;">AWS - Amazon EC2 Admin</code>.</li> 
   <li>In the&nbsp;<strong>Group description</strong>&nbsp;field, enter&nbsp;<code style="color: #000000;">Amazon EC2 administrator permissions</code>.</li> 
   <li>Choose <strong>Create</strong>.</li> 
  </ul> </li> 
</ol> 
<h3 id="step-2-assign-access-for-the-group-in-entra-id">Step 2: Assign access for the group in Entra ID</h3> 
<p>Now you need to assign the newly created group to your enterprise application.</p> 
<ol type="1"> 
 <li>Sign in to the&nbsp;<a href="https://entra.microsoft.com/" rel="noopener" target="_blank">Microsoft Entra admin center</a> with your credentials</li> 
 <li>Select <strong>Applications</strong> and then&nbsp;<strong>Enterprise applications</strong> and select the IAM Identity Center application that you created.</li> 
 <li>Select <strong>Users and groups</strong> from the <strong>Manage</strong> menu group and select <strong>+ Add user/group</strong>.</li> 
 <li>Select the <strong>None selected</strong> option from the <strong>Users and groups</strong> section.</li> 
 <li>Select the <strong>AWS – Amazon EC2 Admin</strong> group checkbox.</li> 
 <li>Choose <strong>Select</strong> and then choose <strong>Assign</strong>.</li> 
 <li>Select <strong>Provisioning</strong> from the <strong>Manage</strong> menu group and begin synchronizing the empty group by selecting the <strong>Start provisioning</strong> option.</li> 
</ol> 
<p>When you first enable provisioning, <a href="https://learn.microsoft.com/en-us/entra/identity/app-provisioning/how-provisioning-works" rel="noopener" target="_blank">the initial Microsoft Entra ID sync is triggered immediately. After that, subsequent syncs are triggered every 40 minutes</a>, with the exact frequency depending on the number of users and groups in the application.</p> 
<p>When the initial sync completes, the AWS – Amazon EC2 Admin group will be ready for configuration in IAM Identity Center.</p> 
<h3 id="step-3-create-permission-sets-in-iam-identity-center">Step 3: Create permission sets in IAM Identity Center</h3> 
<p>As you prepare to configure your permission set, let’s consider session duration from both the AWS and Entra PIM perspectives. There are two session durations on the AWS side: AWS access portal session duration and permission set session duration. The AWS access portal session duration defines the maximum length of time that a user can be signed in to the AWS access portal without reauthenticating. The default session duration is 8 hours but can be configured anywhere between 15 minutes and 7 days.</p> 
<blockquote>
 <p><strong>Note:</strong> Entra does not pass the <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/configure-user-session.html" rel="noopener" target="_blank">SessionNotOnOrAfter</a> attribute to IAM Identity Center as part of the SAML assertion. Meaning the duration of the AWS access portal session is controlled by the duration set in IAM Identity Center.</p>
</blockquote> 
<p>The session duration defined within a permission set specifies the length of time that a user can have a session for an AWS account. The default and minimum value is 1 hour (with a maximum value of 12). Entra PIM allows you to configure an <em>activation</em> <em>maximum duration</em>. The activation maximum duration is the length of time that the specified group will contain the activated user account. The activation maximum duration has a default value of 8 hours but can be configured between 30 minutes and 24 hours.</p> 
<p>You should carefully consider the values that you provide for each of these durations. The AWS access portal will display permission sets that the user had access to at the time that they signed in for the duration of the active AWS access portal session.</p> 
<p>When you set the permission set session duration, you need to keep in mind that active sessions are not terminated when the Entra PIM activation maximum duration has been reached. Let’s look at an example:</p> 
<ul> 
 <li>AWS access portal session duration: default (8 hours)</li> 
 <li>Session duration defined in the permission set: 1 hour</li> 
 <li>Entra PIM group activation maximum duration: 1 hour</li> 
</ul> 
<p>You might be inclined to think that an hour after being added to the group in Entra, the user would no longer have access to AWS resources. This is not necessarily the case. Even after the Entra PIM group activation period expires, <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/authconcept.html#sessionsconcept" rel="noopener" target="_blank">it can take up to 30 minutes for the user to lose access to applications, and in some cases, it can take up to an hour</a>. Additionally, their session would remain active for the duration of the session setting defined in the permission set, which is one hour in this case. In the example shown in Figure 2, we have a potential window of access of up to three hours.</p> 
<div class="wp-caption aligncenter" id="attachment_39008" style="width: 946px;">
 <img alt="Figure 2 – Calculating session duration" class="size-full wp-image-39008" height="205" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/06/19/Picture1-11.png" width="936" />
 <p class="wp-caption-text" id="caption-attachment-39008">Figure 2 – Calculating session duration</p>
</div> 
<blockquote>
 <p><strong>Note</strong>: In most situations, Entra ID group membership changes will be synchronized to IAM Identity Center within 2–10 minutes but can revert to the standard 40-minute interval if activity runs up against Entra PIM throttling limits.</p>
</blockquote> 
<p>With this in mind, configure your test environment with the default setting of 8 hours for the AWS access portal and 1 hour for the permission set session duration value.</p> 
<ol type="1"> 
 <li>Open the&nbsp;<a href="https://console.aws.amazon.com/singlesignon" rel="noopener" target="_blank">IAM Identity Center console</a>.</li> 
 <li>Under&nbsp;<strong>Multi-account permissions</strong>, choose&nbsp;<strong>Permission sets</strong>.</li> 
 <li>Choose&nbsp;<strong>Create permission set</strong>.</li> 
 <li>On the&nbsp;<strong>Select permission set type</strong>&nbsp;page, under&nbsp;<strong>Permission set type</strong>, select&nbsp;<strong>Custom permission set</strong>, and then choose&nbsp;<strong>Next</strong>.</li> 
 <li>On the&nbsp;<strong>Specify policies&nbsp;and permissions boundary</strong> page, expand&nbsp;<strong>AWS managed policies</strong>.</li> 
 <li>Search for and select&nbsp;<strong>AmazonEC2FullAccess</strong>&nbsp;policy, and then choose&nbsp;<strong>Next</strong>.</li> 
 <li>On the&nbsp;<strong>Specify permission set details</strong>&nbsp;page, enter <code style="color: #000000;">EC2AdminAccess</code> for the <strong>Permission set name</strong> and choose <strong>Next</strong>.</li> 
 <li>On the&nbsp;<strong>Review and create</strong>&nbsp;page, review the selections, and choose&nbsp;<strong>Create</strong>.</li> 
</ol> 
<h3 id="step-4-assign-group-access-in-your-organization">Step 4: Assign group access in your organization</h3> 
<p>At this point, you’re ready to assign the Microsoft Entra group to the corresponding permission set in IAM Identity Center. This allows users who are members of the group to be granted the appropriate access level in AWS.</p> 
<ol type="1"> 
 <li>In the navigation pane, under&nbsp;<strong>Multi-account permissions</strong>, choose&nbsp;<strong>AWS accounts</strong>.</li> 
 <li>On the&nbsp;<strong>AWS accounts</strong>&nbsp;page, select the check box next to one or more AWS accounts to which you want to assign access.</li> 
 <li>Choose&nbsp;<strong>Assign users or groups</strong>.</li> 
 <li>On the&nbsp;<strong>Groups</strong>&nbsp;tab, select&nbsp;<strong>AWS – Amazon EC2 Admin</strong>&nbsp;and choose&nbsp;<strong>Next</strong></li> 
 <li>On the&nbsp;<strong>Assign permission sets to <em>“&lt;AWS-account-name&gt;”</em></strong>&nbsp;page, select the&nbsp;<strong>EC2AdminAccess</strong>&nbsp;permission set.</li> 
 <li>Check that the correct permission set was selected and choose&nbsp;<strong>Next</strong>.</li> 
 <li>On the&nbsp;<strong>Review and submit</strong> page, verify that the correct group and permission set are selected, and choose&nbsp;<strong>Submit</strong>.</li> 
</ol> 
<h3 id="step-5-configure-entra-pim">Step 5: Configure Entra PIM</h3> 
<p>To use this Microsoft Entra group with Entra PIM, you bring the group under the management of PIM by using the Entra admin console to activate the group. You can read more about group management with PIM in the <a href="https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/groups-discover-groups" rel="noopener" target="_blank">Microsoft documentation</a>. Begin by activating the Entra group that you created.</p> 
<ol type="1"> 
 <li>Sign in to the&nbsp;<a href="https://entra.microsoft.com/" rel="noopener" target="_blank">Microsoft Entra admin center</a> with your credentials.</li> 
 <li>Select <strong>Groups</strong> and then&nbsp;<strong>All groups</strong></li> 
 <li>Select the <strong>AWS – Amazon EC2 Admin</strong> group.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38531" style="width: 2870px;">
   <img alt="Figure 3 – Selecting groups for PIM enablement" class="size-full wp-image-38531" height="1652" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/28/Just-in-time-privileged-access-3.png" style="border: 1px solid #bebebe;" width="2860" />
   <p class="wp-caption-text" id="caption-attachment-38531">Figure 3 – Selecting groups for PIM enablement</p>
  </div> </li> 
 <li>Select <strong>Privileged Identity Management</strong> under the <strong>Activity</strong> menu list.</li> 
 <li>Choose <strong>Enable PIM for this group</strong>.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38532" style="width: 2910px;">
   <img alt="Figure 4 – Enable PIM for this group" class="size-full wp-image-38532" height="1930" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/28/Just-in-time-privileged-access-4.png" style="border: 1px solid #bebebe;" width="2900" />
   <p class="wp-caption-text" id="caption-attachment-38532">Figure 4 – Enable PIM for this group</p>
  </div> </li> 
</ol> 
<p>Now, you will configure the <a href="https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/groups-role-settings" rel="noopener" target="_blank">PIM settings</a> for the group. These settings define <em>Member</em> or <em>Owner</em> properties and requirements. It’s here that you can establish MFA requirements, configure notifications, conditional access, approvals, durations, and so on. The Owner role can elevate their permissions using just-in-time access to manage a group, while the Member role is limited to requesting just-in-time membership within the group. In this example, you use the Member properties to demonstrate group membership level temporary elevated access and set a 1-hour duration for the group assignment.</p> 
<ol type="1"> 
 <li>Sign in to the&nbsp;<a href="https://entra.microsoft.com/" rel="noopener" target="_blank">Microsoft Entra admin center</a> with your credentials.</li> 
 <li>Select <strong>Identity Governance</strong>, <strong>Privileged Identity Management</strong>, and then&nbsp;<strong>Groups</strong>.</li> 
 <li>Select the <strong>AWS – Amazon EC2 Admin</strong> group.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38533" style="width: 2986px;">
   <img alt="Figure 5 – Selecting groups for PIM configuration" class="size-full wp-image-38533" height="1652" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/28/Just-in-time-privileged-access-5.png" style="border: 1px solid #bebebe;" width="2976" />
   <p class="wp-caption-text" id="caption-attachment-38533">Figure 5 – Selecting groups for PIM configuration</p>
  </div> </li> 
 <li>From the <strong>Manage</strong> menu select <strong>Settings</strong>.</li> 
 <li>Choose <strong>Member</strong> to view the default role setting details.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38534" style="width: 2986px;">
   <img alt="Figure 6 – Settings option for the Member role" class="size-full wp-image-38534" height="1652" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/28/Just-in-time-privileged-access-6.png" style="border: 1px solid #bebebe;" width="2976" />
   <p class="wp-caption-text" id="caption-attachment-38534">Figure 6 – Settings option for the Member role</p>
  </div> </li> 
 <li>Review the default settings. The activation maximum duration should be set to 1 hour and require a justification from the user.</li> 
 <li>Close the <strong>Role setting details – Member</strong> blade.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38535" style="width: 2986px;">
   <img alt="Figure 7 – Closing the Role setting details – Member blade" class="size-full wp-image-38535" height="522" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/28/Just-in-time-privileged-access-7.png" style="border: 1px solid #bebebe;" width="2976" />
   <p class="wp-caption-text" id="caption-attachment-38535">Figure 7 – Closing the Role setting details – Member blade</p>
  </div> </li> 
 <li>From the <strong>Manage</strong> menu select <strong>Assignments</strong> and choose <strong>+ Add assignments</strong>.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38536" style="width: 2874px;">
   <img alt="Figure 8 – Adding eligibility assignments to the PIM enabled groups" class="size-full wp-image-38536" height="1890" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/28/Just-in-time-privileged-access-8.png" style="border: 1px solid #bebebe;" width="2864" />
   <p class="wp-caption-text" id="caption-attachment-38536">Figure 8 – Adding eligibility assignments to the PIM enabled groups</p>
  </div> </li> 
 <li>Select <strong>Member</strong> from the <strong>Select role</strong> dropdown menu and choose <strong>No member selected</strong>. Select the test account, Rich Roe in this example, and then choose <strong>Select</strong>.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38537" style="width: 3578px;">
   <img alt="Figure 9 – Adding the test user as an eligible identity for PIM activation to the group" class="size-full wp-image-38537" height="1890" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/28/Just-in-time-privileged-access-9.png" style="border: 1px solid #bebebe;" width="3568" />
   <p class="wp-caption-text" id="caption-attachment-38537">Figure 9 – Adding the test user as an eligible identity for PIM activation to the group</p>
  </div> </li> 
 <li>Choose <strong>Next</strong> and leave the default setting of 1 year of eligibility. Duration eligibility defines the period that the user can request activation for the group. Depending on your use case, you will define this as permanent or for a set period. For testing purposes, keep the default setting. Choose <strong>Assign</strong>.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38538" style="width: 2522px;">
   <img alt="Figure 10 – Completing the eligibility assignment" class="size-full wp-image-38538" height="1488" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/28/Just-in-time-privileged-access-10.png" style="border: 1px solid #bebebe;" width="2512" />
   <p class="wp-caption-text" id="caption-attachment-38538">Figure 10 – Completing the eligibility assignment</p>
  </div> </li> 
</ol> 
<h2 id="test-the-configuration">Test the configuration</h2> 
<p>You should now have a test configuration of Entra PIM and IAM Identity Center. Use the test account to verify just-in-time access.</p> 
<ol type="1"> 
 <li>Sign in to the&nbsp;<a href="https://entra.microsoft.com/" rel="noopener" target="_blank">Microsoft Entra admin center</a> using the test account (Rich Roe in this example).</li> 
 <li>Select <strong>Identity Governance</strong>, <strong>Privileged Identity Management</strong>, and then <strong>My roles</strong>.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38544" style="width: 2916px;">
   <img alt="Figure 11 – Browsing to the My Roles section of the Entra admin center" class="size-full wp-image-38544" height="1930" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/29/Just-in-time-privileged-access-11.png" style="border: 1px solid #bebebe;" width="2906" />
   <p class="wp-caption-text" id="caption-attachment-38544">Figure 11 – Browsing to the My Roles section of the Entra admin center</p>
  </div> </li> 
 <li>From the <strong>Activate</strong> menu list, select <strong>Groups</strong>. Your eligible group assignments should be listed.</li> 
 <li>Choose <strong>Activate</strong> for the <strong>AWS – Amazon EC2 Admin</strong> group.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38545" style="width: 3418px;">
   <img alt="Figure 12 – Activating the just-in-time group membership" class="size-full wp-image-38545" height="1652" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/29/Just-in-time-privileged-access-12.png" style="border: 1px solid #bebebe;" width="3408" />
   <p class="wp-caption-text" id="caption-attachment-38545">Figure 12 – Activating the just-in-time group membership</p>
  </div> </li> 
 <li>In the <strong>Activate – Member</strong> blade, enter a justification for the access request and choose <strong>Activate</strong>.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38546" style="width: 2880px;">
   <img alt="Figure 13 – Providing a justification for access" class="size-full wp-image-38546" height="1652" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/29/Just-in-time-privileged-access-13.png" style="border: 1px solid #bebebe;" width="2870" />
   <p class="wp-caption-text" id="caption-attachment-38546">Figure 13 – Providing a justification for access</p>
  </div> </li> 
</ol> 
<p>In this example, there are no approval workflow processes configured for the group, so Entra validates the eligibility requirements and adds the test account to the AWS – Amazon EC2 Admin group. If you want to dive deeper into the approval workflow process, you can read more about it on the <a href="https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/groups-role-settings" rel="noopener" target="_blank">Configure PIM for Groups settings page</a>. Because the group is assigned to the enterprise application and configured for provisioning, the updated group membership is automatically synchronized using the SCIM protocol with the connected IAM Identity Center instance. The provisioning time can vary based on the number of PIM enabled users that are activating their memberships within a given 10-second period. In most situations, group memberships are synchronized within 2–10 minutes, but can revert to the standard 40-minute interval if activity runs up against Entra PIM throttling limits. IAM Identity Center responds to SCIM requests as they arrive from Entra ID.</p> 
<p>To test access with the newly activated group assignment, use a separate browser or a private window.</p> 
<ol type="1"> 
 <li>Sign in to the <a href="https://myapps.microsoft.com/" rel="noopener" target="_blank">My Apps portal</a> with the test user credentials and select the IAM Identity Center app that you created for testing. If you experience an error or don’t see the expected permission set, wait a few minutes until the group membership has synchronized to IAM Identity Center and try again.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38547" style="width: 2686px;">
   <img alt="Figure 14 – Accessing IAM Identity Center through the My apps portal" class="size-full wp-image-38547" height="1586" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/29/Just-in-time-privileged-access-14.png" style="border: 1px solid #bebebe;" width="2676" />
   <p class="wp-caption-text" id="caption-attachment-38547">Figure 14 – Accessing IAM Identity Center through the My apps portal</p>
  </div> </li> 
 <li>Expand the associated AWS account and confirm the <strong>EC2ReadOnly</strong> permission set has been granted.</li> 
 <li>Close the AWS tab. Wait for the access to be revoked, which has been set to 60 minutes in this example.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38548" style="width: 2686px;">
   <img alt="Figure 15 – Just-in-time access to the EC2AdminAccess permission set" class="size-full wp-image-38548" height="1586" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/29/Just-in-time-privileged-access-15.png" style="border: 1px solid #bebebe;" width="2676" />
   <p class="wp-caption-text" id="caption-attachment-38548">Figure 15 – Just-in-time access to the EC2AdminAccess permission set</p>
  </div> </li> 
 <li>Sign back in to the <a href="https://myapps.microsoft.com/" rel="noopener" target="_blank">My Apps portal</a> and select the <strong>AWS IAM Identity Center</strong> app. Notice that the <strong>EC2ReadOnly</strong> permission set has been revoked.</li> 
</ol> 
<h2 id="conclusion">Conclusion</h2> 
<p>The combination of AWS IAM Identity Center and Entra PIM provides a robust solution for managing just-in-time elevated access to AWS. By using security groups in Entra and mapping them to permission sets in IAM Identity Center, you can automate the provisioning and deprovisioning of privileged access based on defined policies and approval workflows. This approach helps to make sure the principle of least privilege is enforced, with access granted only for the duration required to complete a task. The detailed auditing capabilities of both services also provide comprehensive visibility into privileged access activities.</p> 
<p>For AWS customers seeking a comprehensive, secure, and scalable privileged access management solution, the Entra PIM and IAM Identity Center integration is a common option that’s worth investigating to see if it’s a good fit for your use case.</p> 
<p>If you have feedback about this post, submit comments in the <strong>Comments</strong> section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener" target="_blank">contact AWS Support</a>.</p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Rodney Underkoffler" class="aligncenter size-full wp-image-38542" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/29/Rodney-Underkoffler.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Rodney Underkoffler</h3> 
  <p>Rodney is a Senior Solutions Architect at Amazon Web Services, focused on guiding enterprise customers on their cloud journey. He has a background in infrastructure, security, and IT business practices. He is passionate about technology and enjoys building and exploring new solutions and methodologies.</p> 
  <p></p>
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Aidan Keane" class="aligncenter size-full wp-image-15987" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2020/09/15/Aidan-Keane-Author.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Aidan Keane</h3> 
  <p>Aidan is a Senior Specialist Solutions Architect at Amazon Web Services, focused on Microsoft Workloads. He partners with enterprise customers to optimize their Microsoft environments on AWS and accelerate their cloud journey. Outside of work, he is a sports enthusiast who enjoys golf, biking, and watching Liverpool FC, while also enjoying family time and travelling to Ireland and South America.</p> 
  <p></p>
 </div> 
</footer>
