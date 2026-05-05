---
title: "Simplify workforce identity management using IAM Identity Center and trusted token issuers"
url: "https://aws.amazon.com/blogs/security/simplify-workforce-identity-management-using-iam-identity-center-and-trusted-token-issuers/"
date: "Thu, 07 Dec 2023 16:15:19 +0000"
author: "Roberto Migli"
feed_url: "https://aws.amazon.com/blogs/security/tag/aws-iam-identity-center/feed/"
---
<blockquote>
 <p><strong>December 12, 2023: </strong>We’ve updated this post to clarify that you can use both <span style="font-family: courier;">sts:audit_context</span> and <span style="font-family: courier;">sts:identity_context</span> can be used to create an identity-enhanced session.</p>
</blockquote> 
<hr /> 
<p><a href="https://aws.amazon.com/iam" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> roles are a powerful way to manage permissions to resources in the <a href="https://aws.amazon.com" rel="noopener" target="_blank">Amazon Web Services (AWS)</a> Cloud. IAM roles are useful when granting permissions to users whose workloads are static. However, for users whose access patterns are more dynamic, relying on roles can add complexity for administrators who are faced with provisioning roles and making sure the right people have the right access to the right roles.</p> 
<p>The typical solution to handle dynamic workforce access is the <a href="https://datatracker.ietf.org/doc/html/rfc6749" rel="noopener" target="_blank">OAuth 2.0 framework</a>, which you can use to propagate an authenticated user’s identity to resource services. Resource services can then manage permissions based on the user—their attributes or permissions—rather than building a complex role management system. <a href="https://aws.amazon.com/iam/identity-center/" rel="noopener" target="_blank">AWS IAM Identity Center</a> recently introduced <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/trustedidentitypropagation-overview.html" rel="noopener" target="_blank">trusted identity propagation</a> based on OAuth 2.0 to support dynamic access patterns.</p> 
<p>With trusted identity propagation, your requesting application obtains tokens from IAM Identity Center and passes them to an AWS resource service. The AWS resource service trusts tokens that Identity Center generates and grants permissions based on the Identity Center tokens.</p> 
<p>What happens if the application you want to deploy uses an external OAuth authorization server, such as Okta Universal Directory or Microsoft Entra ID, but the AWS service uses IAM Identity Center? How can you use the tokens from those applications with your applications that AWS hosts?</p> 
<p>In this blog post, we show you how you can use IAM Identity Center <em>trusted token issuers </em>to help address these challenges. You also review the basics of Identity Center and OAuth and how Identity Center enables the use of external OAuth authorization servers.</p> 
<h2>IAM Identity Center and OAuth</h2> 
<p>IAM Identity Center acts as a central identity service for your AWS Cloud environment. You can bring your workforce users to AWS and authenticate them from an identity provider (IdP) that’s external to AWS (such as Okta or Microsoft&nbsp;Entra), or you can create and authenticate the users on AWS.</p> 
<p>Trusted identity propagation in IAM Identity Center lets AWS workforce identities use OAuth 2.0, helping applications that need to share who’s using them with AWS services. In OAuth, a client application and a resource service both trust the same authorization server. The client application gets a token for the user and sends it to the resource service. Because both services trust the OAuth server, the resource service can identify the user from the token and set permissions based on their identity.</p> 
<p>AWS supports two OAuth patterns:</p> 
<ul> 
 <li><strong>AWS applications authenticate directly with IAM Identity Center:</strong> Identity Center redirects authentication to your identity source, which generates tokens that the <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/awsapps.html" rel="noopener" target="_blank">AWS managed application</a> uses to access AWS services. This is the default pattern because the AWS services that support trusted identity propagation use Identity Center as their authorization server.</li> 
 <li><strong>Third-party, non-AWS applications authenticate outside of AWS (typically to your IdP) and access AWS resources:</strong> During authentication, these <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/customermanagedapps-saml2-oauth2.html#oidc-concept" rel="noopener" target="_blank">third-party applications</a> obtain a token from an OAuth&nbsp;authorization server outside of AWS. In this pattern, the AWS services aren’t connected to the same OAuth&nbsp;authorization server as the client application. To enable this pattern, AWS introduced a model called the trusted token issuer.</li> 
</ul> 
<h2>Trusted token issuer</h2> 
<p>When AWS services use IAM Identity Center as their authentication service, directory, and&nbsp;authorization server, the AWS services that use tokens require that Identity Center issues the tokens. However, most third-party applications federate with an external IdP and obtain tokens from an external authorization server. Although the identities in Identity Center and the external authorization server might be for the same person, the identities exist in separate domains, one in Identity Center, the other in the external authorization server. This is required to manage authorization of workforce identities with AWS services.</p> 
<p>The <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/trustedidentitypropagation.html" rel="noopener" target="_blank">trusted token issuer (TTI)</a> feature provides a way to securely associate one identity from the external IdP with the other identity in IAM Identity Center.</p> 
<p>When using third-party applications to access AWS services, there’s an external OAuth authorization server for the third-party application, and IAM Identity Center is the OAuth authorization server for AWS services; each has its own domain of users. The Identity Center TTI feature connects these two systems so that tokens from the external OAuth authorization server can be exchanged for tokens from Identity Center that AWS services can use to identify the user in the AWS domain of users.&nbsp;A TTI is the external OAuth authorization server that Identity Center trusts to provide tokens that third-party applications use to call AWS services, as shown in Figure 1.</p> 
<div class="wp-caption aligncenter" id="attachment_32704" style="width: 828px;">
 <img alt="Figure 1: Conceptual model using a trusted token issuer and token exchange" class="size-full wp-image-32704" height="570" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/12/06/fig1-2215.jpg" width="818" />
 <p class="wp-caption-text" id="caption-attachment-32704">Figure 1: Conceptual model using a trusted token issuer and token exchange</p>
</div> 
<h2>How the trust model and token exchange work</h2> 
<p>There are two levels of trust involved with TTIs. First, the IAM Identity Center administrator must add the TTI, which makes it possible to exchange tokens. This involves connecting Identity Center to the Open ID Connect (OIDC) discovery URL of the external OAuth authorization server and defining an attribute-based mapping between the user from the external OAuth authorization server and a corresponding user in Identity Center. Second, the applications that exchange externally generated tokens must be configured to use the TTI. There are two models for how tokens are exchanged:</p> 
<ul> 
 <li><strong>Managed AWS service-driven token exchange:</strong> A third-party application uses an AWS driver or API to access a managed AWS service, such as accessing <a href="https://aws.amazon.com/redshift/" rel="noopener" target="_blank">Amazon Redshift</a> by using Amazon Redshift drivers. This works only if the managed AWS service has been designed to accept and exchange tokens. The application passes the external token to the AWS service through an API call. The AWS service then makes a call to IAM Identity Center to exchange the external token for an Identity Center token. The service uses the Identity Center token to determine who the corresponding Identity Center user is and authorizes resource access based on that identity.</li> 
 <li><strong>Third-party application-driven token exchange:</strong> A third-party application not managed by AWS exchanges the external token for an IAM Identity Center token before calling AWS services. This is different from the first model, where the application that exchanges the token is the managed AWS service. An example is a third-party application that uses <a href="https://aws.amazon.com/s3/features/access-grants/" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3) Access Grants</a> to access S3. In this model, the third-party application obtains a token from the external OAuth authorization server and then calls Identity Center to exchange the external token for an Identity Center token. The application can then use the Identity Center token to call AWS services that use Identity Center as their OAuth authorization server. In this case, the Identity Center administrator must register the third-party application and authorize it to exchange tokens from the TTI.</li> 
</ul> 
<h2>TTI trust details</h2> 
<p>When using a TTI, IAM Identity Center trusts that the TTI authenticated the user and authorized them to use the AWS service. This is expressed in an identity token or access token from the external OAuth authorization server (the TTI).</p> 
<p>These are the requirements for the external OAuth authorization server (the TTI) and the token it creates:</p> 
<ul> 
 <li>The token must be a signed <a href="https://jwt.io/" rel="noopener" target="_blank">JSON Web Token (JWT)</a>. The JWT must contain a subject (sub) claim, an audience (aud) claim, an issuer (iss), a user attribute claim, and a JWT ID (JTI) claim. 
  <ul> 
   <li>The subject in the JWT is the authenticated user and the audience is a value that represents the AWS service that the application will use.</li> 
   <li>The audience claim value must match the value that is configured in the application that exchanges the token.</li> 
   <li>The issuer claim value must match the value configured in the issuer URL in the TTI.</li> 
   <li>There must be a claim in the token that specifies a user attribute that IAM Identity Center can use to find the corresponding user in the Identity Center directory.</li> 
   <li>The JWT token must contain the JWT ID claim. This claim is used to help prevent replay attacks. If a new token exchange is attempted after the initial exchange is complete, IAM Identity Center rejects the new exchange request.</li> 
  </ul> </li> 
 <li>The TTI must have an OIDC discovery URL that IAM Identity Center can use to obtain keys that it can use to verify the signature on JWTs created by your TTI. Identity Center appends the suffix <span style="font-family: courier;">/.well-known/openid-configuration</span>&nbsp;to the provider URL that you configure to identify where to fetch the signature keys.</li> 
</ul> 
<p><strong>Note:</strong> Typically, the IdP that you use as your identity source for IAM Identity Center is your TTI. However, your TTI doesn’t have to be the IdP that Identity Center uses as an identity source. If the users from a TTI can be mapped to users in Identity Center, the tokens can be exchanged. You can have as many as 10 TTIs configured for a single Identity Center instance.</p> 
<h2>Details for applications that exchange tokens</h2> 
<p>Your OAuth authorization server service (the TTI) provides a way to authorize a user to access an AWS service. When a user signs in to the client application, the OAuth authorization server generates an ID token or an access token that contains the subject (the user) and an audience (the AWS services the user can access). When a third-party application accesses an AWS service, the audience must include an identifier of the AWS service. The third-party client application then passes this token to an AWS driver or an AWS service.</p> 
<p>To use IAM Identity Center and exchange an external token from the TTI for an Identity Center token, you must configure the application that will exchange the token with Identity Center to use one or more of the TTIs. Additionally, as part of the configuration process, you specify the audience values that are expected to be used with the external token.</p> 
<ul> 
 <li>If the applications are managed AWS services, AWS performs most of the configuration process. For example, the Amazon Redshift administrator connects Amazon Redshift to IAM Identity Center, and then connects a specific Amazon Redshift cluster to Identity Center. The Amazon Redshift cluster exchanges the token and must be configured to do so, which is done through the Amazon Redshift administrative console or APIs and doesn’t require additional configuration.</li> 
 <li>If the applications are third-party and not managed by AWS, your IAM Identity Center administrator must register the application and configure it for token exchange. For example, suppose you create an application that obtains a token from Okta Universal Directory and calls S3 Access Grants. The Identity Center administrator must add this application as a customer managed application and must grant the application permissions to exchange tokens.</li> 
</ul> 
<h2>How to set up TTIs</h2> 
<p>To create new TTIs, go to the <strong>Trusted token issuer</strong> section (Figure 2) of authentication settings in the AWS Management Console for IAM Identity Center. In this section, I show an example of how to use the console to create a new TTI to exchange tokens with my Okta IdP, where I already created my OIDC application to use with my new IAM Identity Center application.</p> 
<div class="wp-caption aligncenter" id="attachment_32705" style="width: 824px;">
 <img alt="Figure 2: Configure the TTI in the IAM Identity Center console" class="size-full wp-image-32705" height="976" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/12/06/fig2-2215.jpg" style="border: 1px solid #bebebe;" width="814" />
 <p class="wp-caption-text" id="caption-attachment-32705">Figure 2: Configure the TTI in the IAM Identity Center console</p>
</div> 
<p>TTI uses the issuer URL to discover the OpenID configuration. Because I use Okta, I can verify that my IdP discovery URL is accessible at&nbsp;https://{my-okta-domain}.okta.com/.well-known/openid-configuration. I can also verify that the OpenID configuration URL responds with a JSON that contains the&nbsp;<span style="font-family: courier;">jwks_uri</span>&nbsp;attribute, which contains a URL that lists the keys that are used by my IdP to sign the JWT tokens. Trusted token issuer requires that both URLs are publicly accessible.</p> 
<p>I then configure the attributes I want to use to map the identity of the Okta user with the user in IAM Identity Center in the <strong>Map attributes</strong> section. I can get the attributes from an OIDC identity token issued by Okta:</p> 
<div class="hide-language"> 
 <pre><code class="lang-text">{
    "sub": "00u22603n2TgCxTgs5d7",
    "email": "&lt;masked&gt;",
    "ver": 1,
    "iss": "https://&lt;masked&gt;.okta.com",
    "aud": "123456nqqVBTdtk7890",
    "iat": 1699550469,
    "exp": 1699554069,
    "jti": "ID.MojsBne1SlND7tCMtZPbpiei9p-goJsOmCiHkyEhUj8",
    "amr": [
        "pwd"
    ],
    "idp": "&lt;masked&gt;",
    "auth_time": 1699527801,
    "at_hash": "ZFteB9l4MXc9virpYaul9A"
}</code></pre> 
</div> 
<p>I’m requesting a token with an additional <span style="font-family: courier;">email</span> scope, because I want to use this attribute to match against the email of my IAM Identity Center users.&nbsp;In most cases, your Identity Center users are synchronized with your central identity provider by using <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/provision-automatically.html" rel="noopener" target="_blank">automatic provisioning with the SCIM protocol</a>. In this case, you can use the Identity Center external ID attribute to match with <span style="font-family: courier;">oid</span> or <span style="font-family: courier;">sub</span> attributes. The only requirement for TTI is that those attributes create a one-to-one mapping between the two IdPs.</p> 
<p>Now that I have created my TTI, I can associate it with my IAM Identity Center applications. As explained previously, there are two use cases. For the managed AWS service-driven token exchange use case, use the service-specific interface to do so. For example, I can use my TTI with Amazon Redshift, as shown in Figure 3:</p> 
<div class="wp-caption aligncenter" id="attachment_32706" style="width: 615px;">
 <img alt="Figure 3: Configure the TTI with Amazon Redshift" class="size-full wp-image-32706" height="1018" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/12/06/fig3-2215.jpg" style="border: 1px solid #bebebe;" width="605" />
 <p class="wp-caption-text" id="caption-attachment-32706">Figure 3: Configure the TTI with Amazon Redshift</p>
</div> 
<p>I selected Okta as the TTI to use for this integration, and I now need to configure the <span style="font-family: courier;">aud</span>claim value that the application will use to accept the token. I can find it when creating the application from the IdP side–in this example, the value is&nbsp;<span style="font-family: courier;">123456nqqVBTdtk7890</span>, and I can obtain it by using the preceding example OIDC identity token.</p> 
<p>I can also use the <a href="https://aws.amazon.com/cli" rel="noopener" target="_blank">AWS Command Line Interface (AWS CLI)</a> to configure the IAM Identity Center application with the appropriate application grants:</p> 
<div class="hide-language"> 
 <pre><code class="lang-text">aws sso put-application-grant \
    --application-arn "&lt;my-application-arn&gt;" \
    --grant-type "urn:ietf:params:oauth:grant-type:jwt-bearer" \
    --grant '
    {
        "JwtBearer": { 
            "AuthorizedTokenIssuers": [
                {
                    "TrustedTokenIssuerArn": "&lt;my-tti-arn&gt;", 
                    "AuthorizedAudiences": [
                        "123456nqqVBTdtk7890"
                    ]
                 }
            ]
       }
    }'</code></pre> 
</div> 
<h2>Perform a token exchange</h2> 
<p>For AWS service-driven use cases, the token exchange between your IdP and IAM Identity Center is performed automatically by the service itself. For third-party application-driven token exchange, such as when building your own Identity Center application with S3 Access Grants, your application performs the token exchange by using the Identity Center OIDC API action <a href="https://docs.aws.amazon.com/singlesignon/latest/OIDCAPIReference/API_CreateTokenWithIAM.html#API_CreateTokenWithIAM_RequestSyntax" rel="noopener" target="_blank">CreateTokenWithIAM</a>:</p> 
<div class="hide-language"> 
 <pre><code class="lang-text">aws sso-oidc create-token-with-iam \  
    --client-id "&lt;my-application-arn&gt;" \ 
    --grant-type "urn:ietf:params:oauth:grant-type:jwt-bearer" \
    --assertion "&lt;jwt-from-idp&gt;"</code></pre> 
</div> 
<p>This action is performed by an IAM principal, which then uses the result to interact with AWS services.</p> 
<p>If successful, the result looks like the following:</p> 
<div class="hide-language"> 
 <pre><code class="lang-text">{
    "accessToken": "&lt;idc-access-token&gt;",
    "tokenType": "Bearer",
    "expiresIn": 3600,
    "idToken": "&lt;jwt-idc-identity-token&gt;",
    "issuedTokenType": "urn:ietf:params:oauth:token-type:access_token",
    "scope": [
        "sts:identity_context",
        "openid",
        "aws"
    ]
}</code></pre> 
</div> 
<p>The value of the <span style="font-family: courier;">scope</span>&nbsp;attribute varies depending on the IAM Identity Center application that you’re interacting with, because it defines the permissions associated with the application.</p> 
<p>You can also inspect the <span style="font-family: courier;">idToken</span> attribute because it’s JWT-encoded:</p> 
<div class="hide-language"> 
 <pre><code class="lang-text">{
    "aws:identity_store_id": "d-123456789",
    "sub": "93445892-f001-7078-8c38-7f2b978f686f",
    "aws:instance_account": "12345678912",
    "iss": "https://identitycenter.amazonaws.com/ssoins-69870e74abba8440",
    "sts:audit_context": "AQoJb3JpZ2…",
    "sts:identity_context": "AQoJb3JpZ2luX2V…",
    "aws:identity_store_arn": "arn:aws:identitystore::12345678912:identitystore/d-996701d649",
    "aud": "20bSatbAF2kiR7lxX5Vdp2V1LWNlbnRyYWwtMQ",
    "aws:instance_arn": "arn:aws:sso:::instance/ssoins-69870e74abba8440",
    "aws:credential_id": "&lt;masked&gt;",
    "act": {
      "sub": "arn:aws:sso::12345678912:trustedTokenIssuer/ssoins-69870e74abba8440/c38448c2-e030-7092-0f0a-b594f83fcf82"
    },
    "aws:application_arn": "arn:aws:sso::12345678912:application/ssoins-69870e74abba8440/apl-0ed2bf0be396a325",
    "auth_time": "2023-11-10T08:00:08Z",
    "exp": 1699606808,
    "iat": 1699603208
  }</code></pre> 
</div> 
<p>The token contains:</p> 
<ul> 
 <li>The AWS account and the IAM Identity Center instance and application that accepted the token exchange</li> 
 <li>The unique user ID of the user that was matched in IAM Identity Center (attribute <span style="font-family: courier;">sub</span>)</li> 
</ul> 
<p>AWS services can now use the tokens found in the attributes <span style="font-family: courier;">sts:audit_context</span> and <span style="font-family: courier;">sts:identity_context</span> to create identity-enhanced sessions with the STS AssumeRole API by using the parameter <span style="font-family: courier;">ProvidedContexts</span>. You can audit the API calls performed by the identity-enhanced sessions in <a href="https://aws.amazon.com/cloudtrail" rel="noopener" target="_blank">AWS CloudTrail</a>, by inspecting the attribute <span style="font-family: courier;">onBehalfOf</span> within the field&nbsp;<a href="https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-user-identity.html" rel="noopener" target="_blank">userIdentity</a>. In this example, you can see an API call that was performed with an identity-enhanced session:</p> 
<div class="hide-language"> 
 <pre><code class="lang-text">"userIdentity": {
    ...
    "onBehalfOf": {
        "userId": "93445892-f001-7078-8c38-7f2b978f686f",
        "identityStoreArn": "arn:aws:identitystore::425341151473:identitystore/d-996701d649"
    }
}</code></pre> 
</div> 
<p>You can thus quickly filter actions that an AWS principal performs on behalf of your IAM Identity Center user.</p> 
<h2>Troubleshooting TTI</h2> 
<p>You can troubleshoot token exchange errors by verifying that:</p> 
<ul> 
 <li>The OpenID discovery URL is publicly accessible.</li> 
 <li>The OpenID discovery URL response conforms with the <a href="https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig" rel="noopener" target="_blank">OpenID standard</a>.</li> 
 <li>The OpenID keys URL referenced in the discovery response is&nbsp;publicly accessible.</li> 
 <li>The issuer URL that you configure in the TTI exactly matches the value of the <span style="font-family: courier;">iss</span>&nbsp;scope that your IdP returns.</li> 
 <li>The user attribute that you configure in the&nbsp;TTI exists in the JWT that your IdP returns.</li> 
 <li>The user attribute value that you configure in the&nbsp;TTI matches exactly one existing IAM Identity Center user on the target attribute.</li> 
 <li>The&nbsp;<span style="font-family: courier;">aud</span>&nbsp;scope exists in the token returned from your IdP and exactly matches what is configured in the requested IAM Identity Center application.</li> 
 <li>The&nbsp;<span style="font-family: courier;">jti</span>&nbsp;claim exists in the token returned from your IdP.</li> 
 <li>If you use an IAM Identity Center application that <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/assignuserstoapp.html" rel="noopener" target="_blank">requires user or group assignments</a>, the matched Identity Center user is already assigned to the application or belongs to a group assigned to the application.</li> 
</ul> 
<p><strong>Note:</strong> When an IAM Identity Center application doesn’t require user or group assignments, the token exchange will succeed if the preceding conditions are met. This configuration implies that the connected AWS service requires additional security assignments. For example, Amazon Redshift administrators need to configure access to the data within Amazon Redshift. The token exchange doesn’t grant implicit&nbsp;access to the AWS services.</p> 
<h2>Conclusion</h2> 
<p>In this blog post, we introduced the trust token issuer feature of IAM Identity Center and what it offers, how it’s configured, and how you can use it to integrate your IdP with AWS services. You learned how to use TTI with AWS-managed applications and third-party applications by configuring the appropriate parameters. You also learned how to troubleshoot token-exchange issues and audit access through CloudTrail.</p> 
<p>If you have feedback about this post, submit comments in the <strong>Comments</strong> section below. If you have questions about this post, start a new thread on the <a href="https://repost.aws/tags/TAJNFEvp8UQUaLplKZtOsAaw/aws-iam-identity-center" rel="noopener" target="_blank">AWS IAM Identity Center re:Post</a> or contact <a href="https://console.aws.amazon.com/support/home" rel="noopener" target="_blank">AWS Support</a>.</p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Roberto Migli" class="aligncenter size-full wp-image-31211" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/10/05/Roberto-Migli.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Roberto Migli</h3> 
  <p>Roberto is a Principal Solutions Architect at AWS. Roberto supports global financial services customers, focusing on security and identity and access management. In his free time, he enjoys building electronic gadgets, learning about space, and spending time with his family.</p> 
  <p></p>
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Ron Cully" class="aligncenter size-full wp-image-11575" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/12/06/cully-photo.jpeg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Ron Cully</h3> 
  <p>Ron is a Principal Product Manager at AWS where he leads feature and roadmap planning for workforce identity products at AWS. Ron has over 20 years of industry experience in product and program management in networking and directory related products. He is passionate about delivering secure, reliable solutions that help make it simple for customers to migrate directory-aware applications and workloads to the cloud.</p> 
  <p></p>
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Rafael Koike" class="aligncenter size-full wp-image-32714" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/12/06/Rafael_Koike.png" width="120" /> 
  </div> 
  <h3 class="lb-h4">Rafael Koike</h3> 
  <p>Rafael is a Principal Solutions Architect supporting enterprise customers in the Southeast and is a Storage SME. Rafael has a passion to build, and his expertise in security, storage, networking, and application development has been instrumental in helping customers move to the cloud quickly and securely.</p> 
  <p></p>
 </div> 
</footer>
