---
title: "Important changes to CloudTrail events for AWS IAM Identity Center"
url: "https://aws.amazon.com/blogs/security/modifications-to-aws-cloudtrail-event-data-of-iam-identity-center/"
date: "Tue, 19 Nov 2024 18:20:01 +0000"
author: "Arthur Mnev"
feed_url: "https://aws.amazon.com/blogs/security/tag/aws-iam-identity-center/feed/"
---
<blockquote>
 <p><strong>November 26, 2025</strong>: All changes to the CloudTrail events of IAM Identity Center described in this blog post are now deployed.</p>
</blockquote> 
<blockquote>
 <p><strong>December 30, 2024</strong>: In response to customer feedback, we updated the effective date for the announced changes from January 13, 2025, to July 14, 2025, and clarified that these changes apply exclusively to IAM Identity Center CloudTrail events.</p>
</blockquote> 
<hr /> 
<p>We are streamlining <a href="https://aws.amazon.com/cloudtrail/" rel="noopener" target="_blank">AWS CloudTrail</a> events for <a href="https://aws.amazon.com/iam/identity-center/" rel="noopener" target="_blank">AWS IAM Identity Center</a>, retaining only essential fields needed for workflows such as audit and incident response. These changes aim to simplify user identification in CloudTrail events of IAM Identity Center, addressing customer feedback. They also enhance correlation between users in the IAM Identity Center directory and external directory services, such as Okta Universal Directory or Microsoft Active Directory. Importantly, these changes do not affect CloudTrail events of other AWS services.</p> 
<p>Effective July 14, 2025, IAM Identity Center will stop emitting <code style="color: #000000;">userName</code> and <code style="color: #000000;">principalId</code> fields under the <a href="https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-user-identity.html" rel="noopener" target="_blank">user identity element</a> in CloudTrail events. These fields will be excluded from the CloudTrail events that are initiated when users sign in to IAM Identity Center, use the <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/using-the-portal.html" rel="noopener" target="_blank">AWS access portal</a>, and access AWS accounts through the <a href="https://awscli.amazonaws.com/v2/documentation/api/latest/reference/configure/sso.html" rel="noopener" target="_blank">AWS CLI</a>. Instead, IAM Identity Center now emits user ID and Identity Store Amazon Resource Name (ARN) fields to replace the <code style="color: #000000;">userName</code> and <code style="color: #000000;">principalId</code> fields, simplifying user identification. IAM Identity Center CloudTrail events will also specify <code style="color: #000000;">IdentityCenterUser</code> as the <a href="https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-user-identity.html" rel="noopener" target="_blank">identity</a> type instead of <code style="color: #000000;">Unknown</code>, providing a clear identifier for users. Additionally, IAM Identity Center will omit the value of a <a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/API_Group.html" rel="noopener" target="_blank">group’s displayName</a> in CloudTrail events when you <a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/API_CreateGroup.html" rel="noopener" target="_blank">create</a> or <a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/API_UpdateGroup.html" rel="noopener" target="_blank">update</a> a group. You can access group attributes, such as <code style="color: #000000;">displayName</code>, by using the Identity Store <a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/API_DescribeGroup.html" rel="noopener" target="_blank">DescribeGroup API operation</a> for authorized workflows.</p> 
<p>We recommend that you update your workflows that process the <code style="color: #000000;">userName</code>, <code style="color: #000000;">principalId</code>, <code style="color: #000000;">userIdentity</code> type, or group <code style="color: #000000;">displayName</code> fields in CloudTrail events for IAM Identity Center before these changes take effect on July 14, 2025. This blog post provides guidance for these updates.</p> 
<h2>How to prepare your workflows for the upcoming changes to IAM Identity Center user identification in CloudTrail</h2> 
<p>To simplify user identification, IAM Identity Center is making changes to the <a href="https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-user-identity.html" rel="noopener" target="_blank">user identity element</a> for its CloudTrail events. Based on these changes, you can update your workflows to link CloudTrail events to a specific user, associate users with their external directories, and track user activity within the same session. The updated user identity element for a sample CloudTrail event is shared at the end of this section.</p> 
<p>IAM Identity Center will update the <code style="color: #000000;">userIdentity</code> type for CloudTrail events that are emitted when users sign in, use the <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/using-the-portal.html" rel="noopener" target="_blank">AWS access portal</a>, and access AWS accounts through the <a href="https://awscli.amazonaws.com/v2/documentation/api/latest/reference/configure/sso.html" rel="noopener" target="_blank">AWS CLI</a>. For authenticated users, the <code style="color: #000000;">userIdentity</code> type will change from <code style="color: #000000;">Unknown</code> to <code style="color: #000000;">IdentityCenterUser</code>. For unauthenticated users, the <code style="color: #000000;">userIdentity</code> type will remain <code style="color: #000000;">Unknown</code>. We recommend that you update your workflows to accept both values.</p> 
<p>To identify the user linked to a CloudTrail event, IAM Identity Center now emits <code style="color: #000000;">userId</code> and <code style="color: #000000;">identityStoreArn</code> fields to replace the <code style="color: #000000;">userName</code> and <code style="color: #000000;">principalId</code> fields. The <code style="color: #000000;">userId</code> is a unique and immutable user identifier that IAM Identity Center assigns to every user in the Identity Store, its native directory referenced by the <code style="color: #000000;">identityStoreArn</code>. These new fields enhance user identification and action tracking in CloudTrail and are present in the CloudTrail entries where the <code style="color: #000000;">userIdentity</code> type is <code style="color: #000000;">IdentityCenterUser</code>. For an example of the user identity element with the new fields and the <code style="color: #000000;">describe-user</code> CLI command to retrieve user attributes using the user ID and Identity Store ARN, see the <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/sso-cloudtrail-use-cases.html#user-session-iam-identity-center" rel="noopener" target="_blank">Identifying the user and session in IAM Identity Center user-initiated CloudTrail events</a> section of the IAM Identity Center User Guide.</p> 
<p>Among other user attributes, you can use the <code style="color: #000000;">describe-user</code> CLI command to retrieve the external ID associated with a user in the Identity Store. You can use the external ID to associate Identity Store users with their external directories. The external ID maps the user to an immutable user identifier in their external directory, such as Microsoft Active Directory or Okta Universal Directory.</p> 
<blockquote>
 <p><strong>Note:</strong> IAM Identity Center doesn’t emit an external ID in CloudTrail. You need access to the Identity Store to retrieve an external ID based on the <code style="color: #000000;">userId</code> and <code style="color: #000000;">identityStoreArn</code> fields in CloudTrail.</p>
</blockquote> 
<p>If you have access to the CloudTrail events but not the Identity Store, you can use the <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/username-sign-in-cloudtrail-events.html" rel="noopener" target="_blank">UserName field emitted under the additionalEventData element</a> to correlate your users with their external directories. This field represents the username that the user authenticates or federates with when signing in to IAM Identity Center. For more details, see the <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/sso-cloudtrail-use-cases.html#correlating-users" rel="noopener" target="_blank">Correlating users between IAM Identity Center and external directories</a> section of the IAM Identity Center User Guide.</p> 
<blockquote>
 <p><strong>Notes:</strong></p> 
 <ul> 
  <li>When the identity source is the AWS Directory Service, the <code style="color: #000000;">UserName</code> value logged in the <code style="color: #000000;">additionalEventData</code> element in CloudTrail is equal to the username that the user enters during authentication. For example, a user who has the username anyuser@company.com, can authenticate with anyuser, anyuser@company.com, or company.com\anyuser, and in each case the entered value is emitted in CloudTrail respectively.</li> 
  <li>For a sign-in failure caused by incorrect username input, IAM Identity Center emits the <code style="color: #000000;">UserName</code> field in its CloudTrail event as a fixed-text value of <code style="color: #000000;">HIDDEN_DUE_TO_SECURITY_REASONS</code>. This is because the username value input by the user in such a scenario could contain sensitive information, such as a user’s password.</li> 
 </ul> 
</blockquote> 
<p>To track user activity within the same session, IAM Identity Center now emits the <code style="color: #000000;">credentialId</code> field in CloudTrail events for user actions that take place in the AWS access portal or that use the AWS CLI. The <code style="color: #000000;">credentialId</code> field contains the AWS access portal session ID for a user, to help you track user actions during their session.</p> 
<p>The following table shows a CloudTrail event example that illustrates the fields, highlighted in yellow, that will change on July 14, 2025. IAM Identity Center recently started emitting <code style="color: #000000;">userId</code>, <code style="color: #000000;">identityStoreArn</code>, <code style="color: #000000;">credentialId</code>, and <code style="color: #000000;">UserName</code> in the additional event data for its CloudTrail events. Therefore, this example considers them as existing fields.</p> 
<table style="border-collapse: collapse; border: 1px solid #808080;" width="100%"> 
 <tbody>
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="100%"><strong>Before the upcoming changes</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="100%"> <pre><code class="lang-json">"eventName": "CredentialChallenge",
"eventSource": "signin.amazonaws.com",
"userIdentity": {
  <span style="background-color: yellow;">"type": "Unknown",</span>
  <span style="background-color: yellow;">"userName": "anyuser",</span>
  "accountId": "123456789012",
  <span style="background-color: yellow;">"principalId": "123456789012",</span>
  <span style="font-weight: 900;">"onBehalfOf"</span>: {
    <span style="font-weight: 900;">"userId"</span>: "a11111-1111-1111-11a1-111aa111aa11",
    <span style="font-weight: 900;">"identityStoreArn"</span>: "arn:aws:identitystore::111111111:identitystore/d-111111a1a"
  },
  <span style="font-weight: 900;">"credentialId"</span>: "1111a111111111a1a11111a1a[…]"
},
"additionalEventData": {
    "CredentialType": "PASSWORD",
    "UserName": "anyuser"
}</code></pre> </td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="100%"><strong>After the upcoming changes</strong></td> 
  </tr> 
  <tr></tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="100%"> <pre><code class="lang-json">"eventName": "CredentialChallenge",
"eventSource": "signin.amazonaws.com",
"userIdentity": {
  <span style="background-color: yellow;">"type": "IdentityCenterUser",</span>
  "accountId": "123456789012",
  <span style="font-weight: 900;">"onBehalfOf"</span>: {
    <span style="font-weight: 900;">"userId"</span>: "a11111-1111-1111-11a1-111aa111aa11",
    <span style="font-weight: 900;">"identityStoreArn"</span>: "arn:aws:identitystore::111111111:identitystore/d-111111a1a"
  },
  <span style="font-weight: 900;">"credentialId"</span>: "1111a111111111a1a11111a1a[…]"
},
"additionalEventData": {
    "CredentialType": "PASSWORD",
    "UserName": "anyuser"
}</code></pre> </td> 
  </tr> 
 </tbody>
</table> 
<h2>How to prepare your workflows for the upcoming changes to IAM Identity Center group management events in CloudTrail</h2> 
<p>Your workflows that require access to group attributes, such as <code style="color: #000000;">displayName</code>, can retrieve them by using the Identity Store <a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/API_DescribeGroup.html" rel="noopener" target="_blank">DescribeGroup API operation</a>. Beginning July 14, 2025, IAM Identity Center will replace the <code style="color: #000000;">displayName</code> value in the administrative CloudTrail events for <a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/API_CreateGroup.html" rel="noopener" target="_blank">CreateGroup</a> and <a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/API_UpdateGroup.html" rel="noopener" target="_blank">UpdateGroup</a> with a fixed text value of <code style="color: #000000;">HIDDEN_DUE_TO_SECURITY_REASONS</code>. This update restricts access to the group <code style="color: #000000;">displayName</code> only to workflows that are authorized to access group attributes in the Identity Store.</p> 
<p>The following table shows a CloudTrail event example that illustrates the upcoming change in the <code style="color: #000000;">displayName</code> field, which is highlighted in yellow.</p> 
<table style="border-collapse: collapse; border: 1px solid #808080;" width="100%"> 
 <tbody>
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="100%"><strong>Before the upcoming changes</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; padding-bottom: 0px; border: 1px solid #808080;" width="100%"> <pre><code class="lang-json">"eventName": "CreateGroup",
"eventSource": "sso-directory.amazonaws.com",
"userIdentity": {
  "type": "AssumedRole",
  "userName": "GroupManagerRole",
  "accountId": "123456789012",
  "principalId": "123456789012"
}
…
"group": {
    "groupId": "11a1a111-1111-1010-aaa1-01111a1111a0",
    <span style="background-color: yellow;"><span style="font-weight: 900;">"displayName"</span>: "PowerUserGroup",</span>
    "groupAttributes": {
        "description": {
            "stringValue": "HIDDEN_DUE_TO_SECURITY_REASONS"
        }
    }
}</code></pre> </td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="100%"><strong>After the upcoming changes</strong></td> 
  </tr> 
  <tr></tr> 
  <tr> 
   <td style="padding: 8px; padding-bottom: 0px; border: 1px solid #808080;" width="100%"> <pre><code class="lang-json">"eventName": "CreateGroup",
"eventSource": "sso-directory.amazonaws.com",
"userIdentity": {
  "type": "AssumedRole",
  "userName": "GroupManagerRole",
  "accountId": "123456789012",
  "principalId": "123456789012"
}
…
"group": {
    "groupId": "11a1a111-1111-1010-aaa1-01111a1111a0",
    <span style="background-color: yellow;"><span style="font-weight: 900;">"displayName"</span>: "HIDDEN_DUE_TO_SECURITY_REASONS",</span>
    "groupAttributes": {
        "description": {
            "stringValue": "HIDDEN_DUE_TO_SECURITY_REASONS"
        }
    }
}</code></pre> </td> 
  </tr> 
 </tbody>
</table> 
<h2>Gain a deeper understanding of the specific CloudTrail events impacted by the changes</h2> 
<p>Earlier in this post, we said that IAM Identity Center emits the relevant CloudTrail events when users sign in to IAM Identity Center, use the <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/using-the-portal.html" rel="noopener" target="_blank">AWS access portal</a>, and access AWS accounts through the <a href="https://awscli.amazonaws.com/v2/documentation/api/latest/reference/configure/sso.html" rel="noopener" target="_blank">AWS CLI</a>, or when administrators create and update groups. These CloudTrail events belong to four event groups that the <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/logging-using-cloudtrail.html" rel="noopener" target="_blank">IAM Identity Center User Guide</a> refers to as AWS access portal, OIDC, Sign-in, and Identity Store events. The following list provides more details about the use cases that lead to the emission of these CloudTrail events:</p> 
<ol> 
 <li>The <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/sso-info-in-cloudtrail.html#cloudtrail-events-access-portal-operations" rel="noopener" target="_blank">AWS access Portal events</a> cover sign-in and sign-out from the AWS access portal, as well as the retrieval of a user’s account and application assignments, which are necessary to display the portal. IAM Identity Center also emits these events when <a href="https://awscli.amazonaws.com/v2/documentation/api/latest/reference/configure/sso.html" rel="noopener" target="_blank">configuring AWS CLI</a> or IDE toolkits for access to AWS accounts as an IAM Identity Center user.</li> 
 <li>The relevant <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/sso-info-in-cloudtrail.html#cloudtrail-events-oidc-operations" rel="noopener" target="_blank">OpenID Connect (OIDC) event</a> is <code style="color: #000000;">CreateToken</code>. IAM Identity Center emits this event when starting a session for an authenticated user (for example, to access assigned AWS accounts through AWS CLI or IDE toolkits).</li> 
 <li>The <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/understanding-sign-in-events.html" rel="noopener" target="_blank">Sign-in events</a> cover password-based and federated authentication, as well as multi-factor authentication (MFA).</li> 
 <li>The relevant <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/sso-info-in-cloudtrail.html#cloudtrail-events-identity-store-operations" rel="noopener" target="_blank">Identity Store events</a> include the <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/user-device-registration.html" rel="noopener" target="_blank">end-user management of MFA devices inside the AWS access portal</a> and the two administrative Identity Store events, <a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/API_CreateGroup.html" rel="noopener" target="_blank">CreateGroup</a> and <a href="https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/API_UpdateGroup.html" rel="noopener" target="_blank">UpdateGroup</a>.</li> 
</ol> 
<p>Note that some of the API operations behind the CloudTrail events in scope are also available as AWS CLI commands:</p> 
<ul> 
 <li><a href="https://awscli.amazonaws.com/v2/documentation/api/latest/reference/sso/index.html" rel="noopener" target="_blank">Portal commands</a></li> 
 <li><a href="https://awscli.amazonaws.com/v2/documentation/api/latest/reference/sso-oidc/index.html" rel="noopener" target="_blank">OIDC commands</a></li> 
 <li><a href="https://awscli.amazonaws.com/v2/documentation/api/latest/reference/identitystore/index.html" rel="noopener" target="_blank">Identity Store commands</a></li> 
</ul> 
<p>The two tables in this section provide a detailed record of the changes and their relation to CloudTrail events.</p> 
<p>The following table lists the changes to fields emitted by IAM Identity Center and the relevant CloudTrail events.</p> 
<table style="border-collapse: collapse; border: 1px solid #808080;" width="100%"> 
 <tbody>
  <tr style="background-color: #f0f0f0;"> 
   <td style="vertical-align: bottom; padding: 8px; border: 1px solid #808080;" width="28%"><strong>Changes</strong></td> 
   <td style="vertical-align: top; padding: 8px; border: 1px solid #808080;" width="18%"><strong><a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/sso-info-in-cloudtrail.html#cloudtrail-events-access-portal-operations" rel="noopener" target="_blank">AWS access portal</a><br />(Use of the portal)<br /> </strong></td> 
   <td style="vertical-align: top; padding: 8px; border: 1px solid #808080;" width="18%"><strong><a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/sso-info-in-cloudtrail.html#cloudtrail-events-oidc-operations" rel="noopener" target="_blankl">OIDC</a><br />(Sign-in to IAM Identity Center through AWS CLI and IDE toolkits)<br /> </strong></td> 
   <td style="vertical-align: top; padding: 8px; border: 1px solid #808080;" width="18%"><strong><a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/understanding-sign-in-events.html" rel="noopener" target="_blank">Sign-in</a><br />(authentication, including MFA, federation)</strong></td> 
   <td style="vertical-align: top; padding: 8px; border: 1px solid #808080;" width="18%"><strong><a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/sso-info-in-cloudtrail.html#cloudtrail-events-identity-store-operations" rel="noopener" target="_blank">Identity Store</a><br />(MFA device and group management)</strong></td> 
  </tr> 
  <tr> 
   <td colspan="5" style="padding: 8px; border: 1px solid #808080;" width="100%"><strong>Available as of July 14, 2025</strong></td> 
  </tr> 
  <tr style="background-color: #f0f0f0;"> 
   <td style="padding: 8px; border: 1px solid #808080;" width="28%">Exclusion of userName from the <code style="color: #000000; background-color: #f0f0f0; border: 0px; padding: 0;">userIdentity</code> element</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">Yes</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">Yes, limited to the <code style="color: #000000; background-color: #f0f0f0; border: 0px; padding: 0;">CreateToken</code> event</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">Yes</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">Yes, limited to MFA management in the AWS access portal</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="28%">Exclusion of <code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">principalId</code> from the <code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">userIdentity</code> element</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">Yes</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">Yes, limited to the <code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">CreateToken</code> event</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">Yes</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">Yes, limited to MFA management in the AWS access portal</td> 
  </tr> 
  <tr style="background-color: #f0f0f0;"> 
   <td style="padding: 8px; border: 1px solid #808080;" width="28%">Modified <code style="color: #000000; background-color: #f0f0f0; border: 0px; padding: 0;">userIdentity</code>’s type value from <code style="color: #000000; background-color: #f0f0f0; font-weight: 900; border: 0px; padding: 0;">Unknown</code> to <code style="color: #000000; background-color: #f0f0f0; font-weight: 900; border: 0px; padding: 0;">IdentityCenterUser</code></td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">Yes</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">Yes, limited to the <code style="color: #000000; background-color: #f0f0f0; border: 0px; padding: 0;">CreateToken</code> event</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">Yes, limited to successful authentications</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">Yes, limited to MFA management in the AWS access portal</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="28%">Exclusion of the group <code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">displayName</code> value from the <code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">requestParameters</code> and <code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">responseElements</code> elements</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">No</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">No</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">No</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">Yes, limited to administrative <code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">CreateGroup</code> and <code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">UpdateGroup</code> events</td> 
  </tr> 
  <tr style="background-color: #f0f0f0;"> 
   <td style="padding: 8px; border: 1px solid #808080;" width="28%">Exclusion of the UserName (in the <code style="color: #000000; background-color: #f0f0f0; border: 0px; padding: 0;">additionalEventData</code> element) a user keys in on failed authentication attempts</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">No</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">No</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">Yes, limited to the <code style="color: #000000; background-color: #f0f0f0; border: 0px; padding: 0;">CredentialChallenge</code> event</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">No</td> 
  </tr> 
  <tr> 
   <td colspan="5" style="padding: 8px; border: 1px solid #808080;" width="100%"><strong>Available as of October 2024</strong></td> 
  </tr> 
  <tr style="background-color: #f0f0f0;"> 
   <td style="padding: 8px; border: 1px solid #808080;" width="28%">Addition of the <code style="color: #000000; background-color: #f0f0f0; border: 0px; padding: 0;">onBehalfOf</code> element with <code style="color: #000000; background-color: #f0f0f0; border: 0px; padding: 0;">userId</code> and <code style="color: #000000; background-color: #f0f0f0; border: 0px; padding: 0;">identityStoreArn</code>, and <code style="color: #000000; background-color: #f0f0f0; border: 0px; padding: 0;">credentialId</code> in the <code style="color: #000000; background-color: #f0f0f0; border: 0px; padding: 0;">userIdentity</code> element</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">Yes</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">Yes, limited to the <code style="color: #000000; background-color: #f0f0f0; border: 0px; padding: 0;">CreateToken</code> event</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">Yes, limited to authentication attempts when username is present in the Identity Store</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">Yes, limited to MFA management in the AWS access portal</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="28%">Addition of <code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">UserName</code> in <code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">additionalEventData</code> element</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">No</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">No</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">Yes, limited to <code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">CredentialChallenge</code> and <code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">UserAuthentication</code> events <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/username-sign-in-cloudtrail-events.html" rel="noopener" target="_blank">in specific cases</a></td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="18%">No</td> 
  </tr> 
 </tbody>
</table> 
<p>The following table summarizes the relevant IAM Identity Center CloudTrail event groups, event sources, and event names.</p> 
<table style="border-collapse: collapse; border: 1px solid #808080;" width="100%"> 
 <tbody>
  <tr style="background-color: #f0f0f0;"> 
   <td style="padding: 8px; border: 1px solid #808080;" width="25%"><strong>Event group</strong></td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="35%"><strong>Source</strong></td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="40%"><strong>Event names</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="25%"><a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/sso-info-in-cloudtrail.html#cloudtrail-events-access-portal-operations" rel="noopener" target="_blank">AWS access portal</a></td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="35%"><code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">sso.amazonaws.com</code></td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="40%"><code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">Authenticate<br />Federate<br />ListAccountRoles<br />ListAccounts<br />ListApplications<br />ListProfilesForApplication<br />GetRoleCredentials<br />Logout</code></td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="25%"><a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/sso-info-in-cloudtrail.html#cloudtrail-events-oidc-operations" rel="noopener" target="_blank">OIDC</a></td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="35%"><code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">sso.amazonaws.com</code></td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="40%"><code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">CreateToken</code></td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="25%"><a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/understanding-sign-in-events.html" rel="noopener" target="_blank">Sign-in</a></td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="35%"><code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">signin.amazonaws.com</code></td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="40%"><code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">CredentialChallenge<br />CredentialVerification<br />UserAuthentication</code></td> 
  </tr> 
  <tr> 
   <td rowspan="2" style="padding: 8px; border: 1px solid #808080;" width="25%"><a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/sso-info-in-cloudtrail.html#cloudtrail-events-identity-store-operations" rel="noopener" target="_blank">Identity Store</a></td> 
   <td style="padding: 8px; border: 1px solid #808080; border-bottom: 0px;" width="35%"><code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">sso-directory.amazonaws.com</code> or<br /><code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">identitystore.amazonaws.com</code></td> 
   <td style="padding: 8px; border: 1px solid #808080; border-bottom: 0px;" width="40%"><code style="color: #000000; background-color: #ffffff; border: 0px; padding: 0;">ListMfaDevicesForUser<br />DeleteMfaDeviceForUser<br />UpdateMfaDeviceForUser<br />StartWebAuthnDeviceRegistration<br />StartVirtualMfaDeviceRegistration<br />CompleteWebAuthnDeviceRegistration<br />CompleteVirtualMfaDeviceRegistration<br />CreateGroup<br />UpdateGroup</code></td> 
  </tr> 
 </tbody>
</table> 
<h2>Conclusion</h2> 
<p>In this post, we reviewed several important upcoming and recently completed changes to CloudTrail events that IAM Identity Center emits. We recommend that you update your CloudTrail based workflows before July 14, 2025 if they rely on the <code style="color: #000000; font-weight: 900;">userName</code>, <code style="color: #000000; font-weight: 900;">principalId</code>, or <code style="color: #000000;">type</code> fields in the CloudTrail user identity element when users sign in to IAM Identity Center, use the <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/using-the-portal.html" rel="noopener" target="_blank">AWS access portal</a>, access AWS accounts through the <a href="https://awscli.amazonaws.com/v2/documentation/api/latest/reference/configure/sso.html" rel="noopener" target="_blank">AWS CLI</a>, or set a group’s <code style="color: #000000;">displayName</code> field in group management administrative events. AWS has recently introduced the fields <code style="color: #000000;">userId</code>, <code style="color: #000000;">identityStoreArn</code>, and <code style="color: #000000;">credentialId</code> in the CloudTrail user identity element to help you complete your updates.</p> 
<p>Please contact your AWS account team or <a href="https://aws.amazon.com/support" rel="noopener" target="_blank">AWS support</a> if you need additional assistance.</p> 
<footer> 
 <div class="blog-author-box"> 
  <img alt="Arthur Mnev" class="alignleft size-full" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2021/03/10/Arthur-Mnev-Author.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Arthur Mnev</span>
  <br />Arthur is a Senior Specialist Security Architect for AWS Industries. He spends his day working with customers and designing innovative approaches to help customers move forward with their initiatives, improve their security posture, and reduce security risks in their cloud journeys. Outside of work, Arthur enjoys being a father, skiing, scuba diving, and Krav Maga.
 </div> 
 <div class="blog-author-box"> 
  <img alt="Alex Milanovic" class="alignleft size-full" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/18/emilanov.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Alex Milanovic</span>
  <br />Alex is a Senior Product Manager at AWS Identity, with over a decade of expertise in Identity and Access Management (IAM) and more than 25 years in the tech sector. His work centers on empowering organizations of all sizes, from large enterprises to small and medium-sized businesses, to effectively adopt and implement IAM cloud services.
 </div> 
</footer>
