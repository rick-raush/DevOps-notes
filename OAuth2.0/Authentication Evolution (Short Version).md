## Authentication Evolution (Short Version)

### 1\. Username + Password

Every application managed its own users.

```text
User -> App
```

Problems:

- Password reuse
    
- Multiple logins
    
- Every app stores credentials
    

* * *

### 2\. LDAP / Active Directory

Centralized user database.

```text
User -> App -> LDAP/AD
```

One identity for many internal applications.

* * *

### 3\. SSO (Single Sign-On)

Goal:

```text
Login once -> Access many apps
```

SSO is **not a protocol**, it's the user experience.

Example:

```text
Login once
↓
Grafana
Jenkins
Kibana
ArgoCD
```

* * *

### 4\. SAML

First major enterprise SSO protocol.

```text
Identity Provider -> SAML Assertion -> Application
```

Characteristics:

- XML based
    
- Enterprise focused
    
- Older but still widely used
    

* * *

### 5\. OAuth 2.0

Created to solve **delegation**, not login.

Before:

```text
App asks for your Gmail password
```

After:

```text
Google issues token to app
```

OAuth answers:

```text
What can this application access?
```

Examples:

- Google Drive access
    
- GitHub API access
    
- Slack API access
    

OAuth = **Authorization**

* * *

### 6\. OpenID Connect (OIDC)

Built on top of OAuth.

OAuth:

```text
What can you access?
```

OIDC:

```text
Who are you?
```

Adds an **ID Token** containing user identity.

Most modern "Login with Google" flows use OIDC.

* * *

# Key Concepts

| Term | What It Is |
| --- | --- |
| Authentication | Verify who the user is |
| Authorization | Verify what the user can access |
| SSO | Login once, access many apps |
| SAML | Older enterprise SSO protocol |
| OAuth 2.0 | Authorization protocol |
| OIDC | Authentication layer on OAuth |
| Keycloak | Identity Provider software implementing OIDC/OAuth/SAML |

* * *

# Keycloak's Role

Keycloak is software that provides:

- User management
    
- SSO
    
- OAuth 2.0
    
- OIDC
    
- SAML
    
- MFA
    

Example:

```text
            Keycloak
                |
      +---------+---------+
      |         |         |
   Grafana   Kibana   Jenkins
```

User logs in once to Keycloak and gets access to all integrated applications.

* * *

# Modern Setup (Most Common Today)

```text
User
  |
  v
Keycloak
  |
  +--> OIDC --> Grafana
  +--> OIDC --> Kibana
  +--> OIDC --> ArgoCD
  +--> OIDC --> Jenkins
```

### One-line Memory Aid

```text
SSO      = Login once everywhere
OAuth2   = Authorization
OIDC     = Authentication on top of OAuth2
SAML     = Older enterprise SSO protocol
Keycloak = Identity provider implementing all of the above
```