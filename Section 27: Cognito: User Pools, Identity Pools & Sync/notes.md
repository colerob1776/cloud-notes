# Amazon Cognito

- Give users an identity to interact with our web or mobile apps
- **User Pools**:
    - Sign in func for app users
    - Integrate with API Gateway & ALB

- **Coginto Identity Pools**:
    - Provide AWS creds to users to they can access AWS resources directly
    - Integrate with User Pools as an Identity Provider

- **Cognito vs IAM**: "hundreds of users", "mobile users", "authenticate with SAML"

# Cognito User Pools (CUP)

- **Create a serverless database of user for your web & mobile apps**

- simple login: Username / password
- Password reset
- Email & Phone Number verification
- MFA
- Federated Identities: users from Facebook, Google, SAML...
- Feature: block users if their creds are compromised elsewhere
- Login sends back a JWT

### CUP - Integrations

- integrates with **API Gateway** and **ALB**

# CUP - Others

### CUP - Lambda Triggers

- CUP can invoke a Lambda synchronously on these triggers:
    - pre-auth
    - post-auth
    - pre token generation
    - pre-signup
    - post confirmation
    - migrate user
    - custom messages
    - pre token generation

### CUP - Hosted Auth UI

- Cognito has a **hosted auth UI** you can use to handle sign-up/sign-in workflows
- Using the hosted UI, you have a foundation for integration with social logins, OIDC, or SAML
- Can customize with **custom CSS and Logo**

### CUP - Hosted UI Custom Domain

- For custom domains, you must create an ACM certificate in **us-east-1** (no other option)
- The custom domain must be defined in the "App Integration" section

### CUP - Adaptive Auth

- **Block sign-ins or require MFA if the login appears suspicious**
- Cognito examines each sign-in attempt and generates a risk score (low, med, high) for how likely the sign-in request is to be malicious
- Users are prompted for a second MFA only when the risk is detected
- Risk score is based on different factors (device, location, IP)
- Checks for compromised creds, account takeover protection, and phone and email verification
- Integration with CloudWatch Logs (sign-in attempts, risk score, failed challenges...)

### Decoding a ID Token; JWT

- CUP issues JWT tokens (Base64 encoded):
    - **Header**
    - **Payload**
    - **Signature**
- **The signature must be verified to ensure the JWT can be trusted**
- Libs can help you verify the validity of JWTs issued by Cognito User Pools
- The Payload will contain the user info (sub UUID, given_name, email, phone_number, attrs...)

# CUP - ALB User Auth

- You ALB can securely auth users
    - Offload the work of authenticating users to your load balancer
    - You apps can focus on their business logic
- Auth users through:
    - Identity Proficer: OpenIDConnect (OIDC) compliant
    - Cognito User Pools:
        - social IdPs (Amazon, Facebook, Google)
        - Corporate identities using SAML, LDAP, or Microsoft AD
- **Must use an HTTPS listener to set authenticate-oidc & authenticate-cognito rules**
- **OnUnauthenticatedRequest** - authenticate (default), deny, allow

# Cognito Identity Pools (Federated Identities)

- **Get identities for "users" so they obtain temp AWS creds**
- Your identity pool (eg identity source) can include:
    - Public Providers
    - users in Cognito User Pool
    - OpenID Connect Providers & SAML IdPs
    - Develop Authenticated identities (custom login server)
    - Cognito Identity Pools allow for ***unauthenticated (guest) access**
- **users can then access AWS services directly or through API Gateway**
    - The IAM policies applied to the creds are defined in Cognito
    - They can be customized based on the user_id for fine-grained control

### Cognito Identity Pools - IAM Roles

- Default IAM roles for auth and guest users
- Define rules to choose the role for each user based on the user's ID
- You can partition your users' access using **policy variables**

- IAM credentials are obtained by Cognito Identity Pools through STS
- The roles must have a "trust" policy of Cognito Identity Pools

# Cognito User Pools vs Identity Pools

- **Cognito User Pools (authentication = identity verification)**:
    - DB of users for your web and mobile apps
    - Allows to federate logins through Public Social, OIDC, SAML..
    - Can customize hosted UI for auth
    - Has triggers w/ AWS Lambda during auth flow
    - Adapt the sign-in UX to different risk levels (MFA, adaptive auth, etc...)
- **Cognito Identiy Pools (authorization = access control)**:
    - Obtain AWS creds for your users
    - Allows to federate logins through Public Social, OIDC, SAML & Cognito User Pools
    - Users can be unauthenticated (guests)
    - Users are mapped to IAM roles & policies, can leverage policy variables

- **CUP + CIP = authentication + authorization**

