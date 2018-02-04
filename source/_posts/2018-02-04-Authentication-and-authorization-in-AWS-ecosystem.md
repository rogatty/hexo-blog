---
title: Authentication and authorization in AWS ecosystem
description: AWS authentication and authorization services are much more complex than I need them to be. I had to do a lot of research to understand how various elements fit together. There are enough moving parts that in my case manual integration didn't make sense. It's better to use external libraries like `amazon-cognito-identity-js` and `aws-sign-web`. While the complexity is high, I think it's a powerful and flexible setup which may become useful if I ever want to scale.
date: 2018-02-04 10:27:57
---
AWS authentication and authorization services are much more complex than I need them to be. I had to do a lot of research to understand how various elements fit together. There are enough moving parts that in my case manual integration didn't make sense. It's better to use external libraries like `amazon-cognito-identity-js` and `aws-sign-web`. While the complexity is high, I think it's a powerful and flexible setup which may become useful if I ever want to scale.
<!-- more --> 

## AWS Cognito
Official definition from the [Cognito page](https://aws.amazon.com/cognito/):
>Amazon Cognito lets you add user sign-up/sign-in and access control to your web and mobile apps quickly and easily. Cognito scales to millions of users, and supports sign-in with social identity providers such as Facebook, Google, and Amazon, and enterprise identity providers via SAML 2.0.

AWS Cognito consists of two elements: User Pool and Identity Pool.

Cognito User Pool is an identity provider. It allows you to register, authenticate and recover accounts using nothing but AWS. You don't need it to use the rest of Cognito, it's replaceable by e.g. Google or Facebook identity providers. I plan to use the Google one in the future but decided that AWS authentication is good enough for the MVP.

Cognito Identity Pool is a place where you map an identity provider or a group of them to specific IAM roles. One for authenticated users and one for unauthenticated users.

{% asset_img "cognito identity pool.png" "Cognito Identity Pool settings for Journal" %}

## Authentication in Journal
I used [amazon-cognito-identity-js](https://github.com/aws/amazon-cognito-identity-js) which handles the security. I needed to create a login page with a form, logout button, set up the redirects, synchronize the login state with React app and call the library where needed. I mostly followed the steps on [Serverless stack](https://serverless-stack.com/chapters/create-a-login-page.html) and had no issues worth mentioning. You can check [two](https://github.com/rogatty/journal/commit/c33887d02ffa7255ff5a0a30001d8c94185c54bf) [commits](https://github.com/rogatty/journal/commit/41cca8f1b6841a8aa7ff02fb4890af4d4931c7fe) if you want to see the code.

{% asset_img "login page.png" "Journal Login page" %}

I checked the requests made by `amazon-cognito-identity-js` and had a pleasant surprise. The password is never sent as a plain text which is the case on [Fandom](https://www.wikia.com/signin) where I work. AWS Cognito uses two steps for authentication: `InitiateAuth` and `RespondToAuthChallenge`. If you use Cognito User Pool as I do, then you'll see two POST requests to `https://cognito-idp.eu-central-1.amazonaws.com/`.

Authentication flow initiation:
```json
{
  "AuthFlow":"USER_SRP_AUTH",
  "ClientId":"xxx",
  "AuthParameters": {
    "USERNAME":"my@email.com",
    "SRP_A":"xxx"
  }
}
```

Which returns:
```json
{
  "ChallengeName": "PASSWORD_VERIFIER",
  "ChallengeParameters": {
    "SALT": "xxx",
    "SECRET_BLOCK": "xxx",
    "SRP_B": "xxx",
    "USERNAME": "0097425b-00a9-424d-8b55-13ea71dd1cf0",
    "USER_ID_FOR_SRP": "0097425b-00a9-424d-8b55-13ea71dd1cf0"
  }
}
```

Challenge response: 
```json
{
  "ChallengeName": "PASSWORD_VERIFIER",
  "ClientId": "xxx",
  "ChallengeResponses": {
    "USERNAME": "0097425b-00a9-424d-8b55-13ea71dd1cf0",
    "PASSWORD_CLAIM_SECRET_BLOCK": "xxx",
    "TIMESTAMP": "Sun Feb 4 11:08:36 UTC 2018",
    "PASSWORD_CLAIM_SIGNATURE": "xxx"
  }
}
```

Which returns:
```json
{
  "AuthenticationResult": {
    "AccessToken": "xxx",
    "ExpiresIn": 3600,
    "IdToken": "xxx",
    "RefreshToken": "xxx",
    "TokenType": "Bearer"
  },
  "ChallengeParameters": {}
}
```

It's a secure and customizable process, described in details on [AWS blog](https://aws.amazon.com/blogs/mobile/customizing-your-user-pool-authentication-flow/).

As a side note, if you use `create-react-app` then there is a way to [pass custom environment variables to the browser](https://github.com/facebook/create-react-app/blob/master/packages/react-scripts/template/README.md#adding-custom-environment-variables). I used it together with [.env files](https://github.com/facebook/create-react-app/blob/master/packages/react-scripts/template/README.md#adding-development-environment-variables-in-env) to keep "secrets" outside of the repo. I put secrets in quotes because there are no real secrets in JS applications where everyone can use Dev Tools. AWS Cognito takes that into the consideration and doesn't require secrets from JS applications, only various IDs. I keep them out of repo anyway, in case I misconfigured something. [Security by obscurity](https://github.com/aws/amazon-cognito-identity-js/issues/312) :) If I ever allow others to use Journal it will no longer make sense.

## AWS Identity and Access Management (IAM)
Official definition from the [IAM page](https://aws.amazon.com/iam/):
>AWS Identity and Access Management (IAM) is a web service that helps you securely control access to AWS resources for your users. You use IAM to control who can use your AWS resources (authentication) and what resources they can use and in what ways (authorization).

[Serverless stack](https://serverless-stack.com/chapters/what-is-iam.html) explains the concepts of users, policies, roles, and groups very well.

Following the tutorial I set up `journalAuth_Role` with access to Cognito for identification and API Gateway for GraphQL access:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cognito-sync:*",
        "cognito-identity:*"
      ],
      "Resource": [
        "*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "execute-api:Invoke"
      ],
      "Resource": [
        "arn:aws:execute-api:eu-central-1:*:xxx/*"
      ]
    }
  ]
}
```

I skipped S3 access for now. There will be time for that later.

## Authorization in Journal
AWS API Gateway requires requests to be signed with [Signature Version 4](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html). When I was trying to do that I realized that Serverless stack tutorial won't help me much. They make signed API requests using [a custom method](https://serverless-stack.com/chapters/connect-to-api-gateway-with-iam-auth.html) which works for them because they don't use the Apollo Client (GraphQL client library).

I found [Sending GraphQL requests through AWS API Gateway using the Apollo client](https://medium.com/merapar/apollo-client-with-aws-api-gw-9fd25ce9f72d) post but it described integration with Apollo v1 while I use v2 which is no longer compatible with this method. In the Apollo docs I found a way to [override fetch method](https://github.com/apollographql/apollo-link/tree/master/packages/apollo-link-http#custom-fetching) in v2. I used that in combination with `sigV4Client.js` from Serverless stack but got a 403 response with `x-amzn-errortype: InvalidSignatureException` header set. I doubt it's a fault of the library, more likely I did something wrong because of its poor documentation.

Finally, I found [aws-sign-web](https://github.com/danieljoos/aws-sign-web/blob/master/aws-sign-web.js) which worked great. It's an npm module so it needed a few customizations to work in a browser but that was straightforward. You can check out the [commit](https://github.com/rogatty/journal/commit/9b272d079fd6ec32c03d79bac9c10234745a5228) if you want to see the exact code.
