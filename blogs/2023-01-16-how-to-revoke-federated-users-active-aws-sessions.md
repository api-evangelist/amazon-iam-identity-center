---
title: "How to revoke federated users’ active AWS sessions"
url: "https://aws.amazon.com/blogs/security/how-to-revoke-federated-users-active-aws-sessions/"
date: "Mon, 16 Jan 2023 17:43:34 +0000"
author: "Matt Howard"
feed_url: "https://aws.amazon.com/blogs/security/tag/aws-iam-identity-center/feed/"
---
February 6, 2023: Updates added to explain an additional detail regarding the sourceIdentity field. In addition to using the sourceIdentity field to reference the user through various roles they have assumed, you may also construct your IAM trust policies to enforce acceptable sourceIdentity values or ensure any value for sourceIdentity is set. When you use […]
