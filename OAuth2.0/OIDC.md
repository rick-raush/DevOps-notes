&nbsp;

## 🔑 What is OpenID Connect (OIDC)?

- **OIDC = OpenID Connect**
    
- It is an **identity layer built on top of OAuth 2.0**.
    
- While OAuth2 is about **authorization** (“can this app access this data?”), OIDC is about **authentication** (“who is this user?”).
    

👉 In short:

- **OAuth2** → Delegated authorization (grant access to an API).
    
- **OIDC** → Authentication (log the user in, prove their identity).
    

* * *

## 🏗️ How OIDC Works

OIDC extends OAuth2 flows by introducing the **ID Token** (a JWT) alongside the **Access Token**.

1.  **User logs in** through the Authorization Server (Identity Provider, e.g. Google, Okta, Keycloak).
    
2.  The app (client) receives:
    
    - **Access Token** → to call APIs.
        
    - **ID Token** → contains user identity info (who they are, email, etc.).
        
    - (optional) **Refresh Token** → to renew tokens.
        

* * *

## 🔑 Key Components

- **Identity Provider (IdP)**  
    Issues tokens (Google, Microsoft, Okta, Keycloak).
    
- **Relying Party (RP) / Client**  
    The app that needs to authenticate users.
    
- **ID Token**
    
    - A JWT signed by the IdP.
        
    - Contains claims like `sub` (user ID), `email`, `name`.
        
    - Example payload:
        
        ```json
        {
          "iss": "https://accounts.google.com",
          "sub": "1234567890",
          "name": "Alice",
          "email": "alice@example.com"
        }
        ```
        
- **UserInfo Endpoint**  
    An API to fetch more profile info if needed.
    

* * *

## 📌 Why OIDC is important in DevOps & Cloud

- Standard for **SSO (Single Sign-On)** → login once, access multiple apps.
    
- Works with **federated identity** (Google, GitHub login).
    
- Used in **Kubernetes (Dex, Keycloak, OIDC providers)** for authenticating cluster access.
    
- Works across **SPAs, mobile apps, APIs** with secure token flows.
    

* * *

## ⚖️ OIDC vs OAuth2

| Feature | OAuth2 | OIDC (OpenID Connect) |
| --- | --- | --- |
| Purpose | Authorization (access APIs) | Authentication (who is user) |
| Tokens | Access Token | Access Token + **ID Token** |
| User identity info | Not provided | Provided via ID Token/UserInfo |
| Example use case | Allow app to access Google Drive | “Login with Google” button |

* * *

&nbsp;