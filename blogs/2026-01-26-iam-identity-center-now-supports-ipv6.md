---
title: "IAM Identity Center now supports IPv6"
url: "https://aws.amazon.com/blogs/security/iam-identity-center-now-supports-ipv6/"
date: "Mon, 26 Jan 2026 20:17:19 +0000"
author: "Suchintya Dandapat"
feed_url: "https://aws.amazon.com/blogs/security/tag/aws-iam-identity-center/feed/"
---
<p><a href="https://aws.amazon.com" rel="noopener" target="_blank">Amazon Web Services (AWS)</a> recommends using <a href="https://aws.amazon.com/iam/identity-center/" rel="noopener" target="_blank">AWS IAM Identity Center</a> to provide your workforce access to AWS managed applications—such as <a href="https://aws.amazon.com/iam/identity-center/q/" rel="noopener" target="_blank">Amazon Q Developer</a>—and AWS accounts. Today, we announced IAM Identity Center support for IPv6. To learn more about the advantages of IPv6, visit the <a href="https://aws.amazon.com/vpc/ipv6/" rel="noopener" target="_blank">IPv6 product page</a>.</p> 
<p>When you enable IAM Identity center, it provides an access portal for workforce users to access their AWS applications and accounts either by signing in to the access portal using a URL or by using a bookmark for the application URL. In either case, the access portal handles user authentication before granting access to applications and accounts. Supporting both IPv4 and IPv6 connectivity to the access portal helps facilitate seamless access for clients, such as browsers and applications, regardless of their network configuration.</p> 
<p>The launch of IPv6 support in IAM Identity Center introduces new dual-stack endpoints that support both IPv4 and IPv6, so that users can connect using IPv4, IPv6, or dual-stack clients. Current IPv4 endpoints continue to function with no action required. The dual stack capability offered by Identity Center extends to managed applications. When users access the application dual-stack endpoint, the application automatically routes to the Identity Center dual-stack endpoint for authentication. To use Identity Center from IPv6 clients, you must direct your workforce to use the new dual-stack endpoints, and update configurations on your external identity provider (IdP), if you use one.</p> 
<p>In this post, we show you how to update your configuration to allow IPv6 clients to connect directly to IAM Identity Center endpoints without requiring network address translation services. We also show you how to monitor which endpoint users are connecting to. Before diving into the implementation details, let’s review the key phases of the transition process.</p> 
<h2>Transition overview</h2> 
<p>To use IAM Identity Center from an IPv6 network and client, you need to use the new dual-stack endpoints. Figure 1 shows what the transition from IPv4 to IPv6 over dual-stack endpoints looks like when using Identity Center. The figure shows:</p> 
<ul> 
 <li>A before state where clients use the IPv4 endpoints.</li> 
 <li>The transition phase, when your clients use a combination of IPv4 and dual-stack endpoints.</li> 
 <li>After the transition is complete, your clients will connect to dual-stack endpoints using their IPv4 or IPv6, depending on their preferences.</li> 
</ul> 
<p style="line-height: 1.25em;"></p>
<div class="wp-caption aligncenter" id="attachment_41200" style="width: 799px;">
 <img alt="Figure 1: Transition from IPv4-only to dual-stack endpoints" class="size-full wp-image-41200" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2026/01/20/img1-1.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-41200">Figure 1: Transition from IPv4-only to dual-stack endpoints</p>
</div>
<p></p> 
<h2>Prerequisites</h2> 
<p>You must have the following prerequisites in place to enable IPv6 access for your workforce users and administrators:</p> 
<ul> 
 <li>An existing IAM Identity Center instance</li> 
 <li><a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/enable-identity-center-portal-access.html" rel="noopener" target="_blank">Updated firewalls or gateways</a> to include the new dual-stack endpoints</li> 
 <li>IPv6 capable clients and networks</li> 
</ul> 
<p>Work with your network administrators to update the configuration of your firewalls and gateways and to verify that your clients, such as laptops or desktops, are ready to accept IPv6 connectivity. If you have already enabled IPv6 connectivity for other AWS services, you might be familiar with these changes. Next, implement the two steps that follow.</p> 
<h2>Step 1: Update your IdP configuration</h2> 
<p>You can skip this step If you don’t use an external IdP as your identity source.</p> 
<p>In this step, you update the Assertion Consumer Service (ACS) URL from your IAM Identity Center instance into your IdP’s configuration for single sign-on and the SCIM configuration for user provisioning. Your IdP’s capability determines how you update the ACS URLs. If your IdP supports multiple ACS URLs, configure both IPv4 and dual-stack URLs to enable a flexible transition. With that configuration, some users can continue using IPv4-only endpoints while others use dual-stack endpoints for IPv6. If your IdP supports only one ACS URL, to use IPv6 you must update the new dual-stack ACS URL in your IdP and transition all users to using dual-stack endpoints. If you don’t use an external IdP, you can skip this step and go to the next step.</p> 
<h3>Update both the SAML single sign-on and the SCIM provisioning configurations:</h3> 
<ol> 
 <li>Update the single sign-on settings in your IdP to use the new dual-stack URLs. First, locate the URLs in the AWS Management Console for IAM Identity Center. 
  <ol> 
   <li>Choose <strong>Settings</strong> in the navigation pane and then select <strong>Identity source</strong>.</li> 
   <li>Choose <strong>Actions</strong> and select <strong>Manage authentication</strong>.</li> 
   <li>in Under <strong>Manage SAML 2.0 authentication</strong>, you will find the following URLs under <strong>Service provider metadata</strong>: 
    <ul> 
     <li>AWS access portal sign-in URL</li> 
     <li>IAM Identity Center Assertion Consumer Service (ACS) URL</li> 
     <li>IAM Identity Center issuer URL</li> 
    </ul> </li> 
  </ol> </li> 
 <li>If your IdP supports multiple ACS URLs, then add the dual-stack URL to your IdP configuration alongside existing IPv4 one. With this setting, you and your users can decide when to start using the dual-stack endpoints, without all users in your organization having to switch together. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_41201" style="width: 750px;">
   <img alt="Figure 2: Dual-stack single sign-on URLs" class="size-full wp-image-41201" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2026/01/20/img2.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-41201">Figure 2: Dual-stack single sign-on URLs</p>
  </div><p></p> </li> 
 <li>If your IdP does not support multiple ACS URLs, replace the existing IPv4 URL with the new dual-stack URL, and switch your workforce to use only the dual-stack endpoints.</li> 
 <li>Update the provisioning endpoint in your IdP. Choose <strong>Settings</strong> in the navigation pane and under <strong>Identity source</strong>, choose <strong>Actions</strong> and select <strong>Manage provisioning</strong>. Under <strong>Automatic provisioning</strong>, copy the new <strong>SCIM endpoint</strong> that ends in <code style="color: #000000;">api.aws</code>. Update this new URL in your external IdP. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_41202" style="width: 750px;">
   <img alt="Figure 3: Dual-stack SCIM endpoint URL" class="size-full wp-image-41202" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2026/01/20/img3-1.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-41202">Figure 3: Dual-stack SCIM endpoint URL</p>
  </div><p></p> </li> 
</ol> 
<h2>Step 2: Locate and share the new dual-stack endpoints</h2> 
<p>Your organization needs two kinds of URLs for IPv6 connectivity. The first is the new dual-stack access portal URL that your workforce users use to access their assigned AWS applications and accounts. The dual-stack access portal URL is available in the IAM Identity Center console, listed as the <strong>Dual-stack </strong>in the <strong>Settings summary</strong> (you might need to expand the <strong>Access portal URLs</strong> section, shown in Figure 4).</p> 
<div class="wp-caption aligncenter" id="attachment_41203" style="width: 545px;">
 <img alt="Figure 4: Locate dual-stack access portal endpoints" class="size-full wp-image-41203" height="687" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2026/01/20/img4.png" style="border: 1px solid #bebebe;" width="535" />
 <p class="wp-caption-text" id="caption-attachment-41203">Figure 4: Locate dual-stack access portal endpoints</p>
</div> 
<p>This dual-stack URL ends with <code style="color: #000000;">app.aws</code> as its top-level domain (TLD). Share this URL with your workforce and ask them to use this dual-stack URL to connect over IPv6. As an example, if your workforce uses the access portal to access AWS accounts, they will need to sign in through the new dual-stack access portal URL when using IPv6 connectivity. Alternately, if your workforce accesses the application URL, you need to enable the dual-stack application URL following application-specific instructions. For more information, see <a href="https://docs.aws.amazon.com/vpc/latest/userguide/aws-ipv6-support.html" rel="noopener" target="_blank">AWS services that support IPv6</a>.</p> 
<p>The URLs that administrators use to manage IAM Identity Center are the second kind of URL your organization needs. The new dual-stack service endpoints end in <code style="color: #000000;">api.aws</code> as their TLD and are listed in the <a href="https://docs.aws.amazon.com/general/latest/gr/sso.html#sso_region" rel="noopener" target="_blank">Identity Center service endpoints</a>. Administrators can use these service endpoints to manage users and groups in Identity Center, update their access to applications and resources, and perform other management operations. As an example, if your administrator uses <code style="color: #000000;">identitystore.{region}.amazonaws.com</code> to manage users and groups in Identity Center, they should now use the dual-stack version of the same service endpoint which is <code style="color: #000000;">identitystore.{region}.api.aws</code>, so they can connect to service endpoints using IPv6 clients and networks.</p> 
<p>If your users or administrators use an AWS SDK to access AWS applications and accounts or manage services, follow <a href="https://docs.aws.amazon.com/sdkref/latest/guide/feature-endpoints.html" rel="noopener" target="_blank">Dual-stack and FIPS endpoints</a> to enable connectivity to the dual-stack endpoints.</p> 
<p>After completing these two steps, your workforce and administrators can connect to IAM Identity Center using IPv6. Remember, these endpoints also support IPv4, so clients not yet IPv6-capable can continue to connect using IPv4.</p> 
<h2>Monitoring dual-stack endpoint usage</h2> 
<p>You can optionally monitor <a href="https://aws.amazon.com/cloudtrail" rel="noopener" target="_blank">AWS CloudTrail</a> logs to track usage of dual-stack endpoints. The key difference between IPv4-only and dual-stack endpoint usage is the TLD and appears in the <code style="color: #000000;">clientProvidedHostHeader</code> field. The following example shows the difference between these CloudTrail events for the <code style="color: #000000;">CreateTokenWithIAM</code> API call.</p> 
<table style="border-collapse: collapse; border: 1px solid #808080;" width="100%"> 
 <tbody>
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%"><strong>IPv4-only endpoints</strong></td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%"><strong>Dual-stack endpoints</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%"> 
    <div class="hide-language"> 
     <pre class="unlimited-height-code"><code class="lang-text">"CloudTrailEvent": {
  "eventName": "CreateToken",
  "tlsDetails": {
     "tlsVersion": "TLSv1.3",
     "cipherSuite": "TLS_AES_128_GCM_SHA256",
     <span style="font-weight: 900;">"clientProvidedHostHeader": "oidc.us-east-1.amazonaws.com"</span>
  }
}</code></pre> 
    </div> </td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%"> 
    <div class="hide-language"> 
     <pre class="unlimited-height-code"><code class="lang-text">"CloudTrailEvent": {
  "eventName": "CreateToken",
  "tlsDetails": {
     "tlsVersion": "TLSv1.3",
     "cipherSuite": "TLS_AES_128_GCM_SHA256",
     <span style="font-weight: 900;">"clientProvidedHostHeader": "oidc.us-east-1.api.aws"</span>
  }
}</code></pre> 
    </div> </td> 
  </tr> 
 </tbody>
</table> 
<h2>Conclusion</h2> 
<p>IAM Identity Center now allows clients to connect over IPv6 natively with no network address translation infrastructure. This post showed you how to transition your organization to use IPv6 with Identity Center and its integrated applications. Remember that existing IPv4 endpoints will continue to function, so you can transition at your own pace. Also, no immediate action is required by you. However, we recommend planning your transition to take advantage of IPv6 benefits and meet compliance requirements. If you have questions, comments, or concerns, contact&nbsp;<a href="https://console.aws.amazon.com/support/home" rel="noopener" target="_blank">AWS Support</a><u>,</u> or start a new thread in the <a href="https://repost.aws/tags/TAJNFEvp8UQUaLplKZtOsAaw/aws-iam-identity-center" rel="noopener" target="_blank">IAM Identity Center re:Post channel</a>.</p> 
<p>&nbsp;<br />If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.<br />&nbsp;</p> 
<footer> 
 <div class="blog-author-box">
  <img alt="Suchintya Dandapat" class="alignleft size-full" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2026/01/20/suchinto.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Suchintya Dandapat</span>
  <br />Suchintya Dandapat is a Principal Product Manager for AWS where he partners with enterprise customers to solve their toughest identity challenges, enabling secure operations at global scale. 
 </div> 
</footer>
