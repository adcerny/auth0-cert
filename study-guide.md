
## **Introduction**

Preparing for the Auth0 Certified Developer Exam requires more than a surface understanding of identity and access management. It involves developing a clear grasp of how the platform behaves in practice, including its APIs, configuration options, supported protocols, and security mechanisms.  

This guide was created during my own preparation for the exam, with the goal of bringing together key details from the documentation, training materials, and hands-on experience into a single structured resource. It focuses on the areas that are often overlooked but frequently tested, such as exact parameter names, default settings, limitations, and the subtle differences between related features.  

The intention is not to replace the official learning materials but to complement them. It provides a concise, technically focused reference designed to support effective revision and build confidence when approaching the more detailed and scenario-based questions in the exam.  

---

## **Before You Start**

Success in the Auth0 Certified Developer Exam depends on preparation and familiarity with how the platform behaves in real-world use. Below are some practical tips and study advice based on my own experience preparing for the exam.

### **1. Start with the Related Learning**
Complete all of the **Related Learning** modules listed in the official [Auth0 Certified Developer Study Guide](https://certification.okta.com/page/developer-auth0-exam-study-guide), including the codelabs.  
Take notes on any topics that appear in the short tests at the end of each section, as many of these concepts reappear in the real exam.  
For the codelabs, **register a new Auth0 tenant for each lab** to keep your environment clean and avoid configuration conflicts.

### **2. Go Beyond the Study Guide**
Do not expect to pass the exam by completing the study guide alone.  
The real exam goes into fine detail, much of which is not covered in the related learning. Read the official documentation to fill in gaps, but do not feel you need to memorise everything. Focus on taking clear notes on areas where your understanding feels weaker.

### **3. Practice Exams**
- **Standard Practice Exam:** Take the standard exam and review every question you get wrong. Research the topics you struggled with, then retake the test until you understand all 20 questions thoroughly. The real exam is double that length, so aim for full mastery here.  
- **Premium Practice Exam:** If your budget allows, this is worth every penny. It includes the same number of questions as the real exam and many of the topics will appear again. The practical exercises are especially valuable, as they walk you through setting up applications and using the Management API, both of which are essential for the real exam. You can retake the Premium exam up to seven times, so use it to refine your weak areas and build confidence.  

The **practical section** of the exam is worth roughly **half the total score**, and if you follow the instructions carefully you can get close to full marks. So do not panic if you have to make a few educated guesses on the DOMC questions — a strong performance in the practical can easily balance things out.

### **4. DOMC Question Tips**
The exam uses **DOMC (Discrete Option Multiple Choice)** questions. Each option appears one at a time, and you must decide immediately whether it is correct.  
Here are some key tips:
- Do not rush. Think carefully about how each option relates to the question.  
- If you do not see a clear and specific connection to the question, it is safest to assume the option is incorrect.  
- Treat each option independently, as more than one answer may be correct.  
- If an option feels vaguely familiar, pause and think about **why**. Try to recall the exact context in which you’ve seen it before. The exam often tests small but important distinctions between similar concepts, so make sure your answer fits the question precisely rather than relying on general recognition.  
- Read the question stem closely before you start answering; subtle wording often changes what counts as correct.

### **5. Exam Day**
Allow around **40 minutes** to get set up with the proctoring company. Make sure you understand the requirements for ID verification, desk space, and allowed materials before your scheduled time.  
Use **wired peripherals** (mouse, keyboard, speakers) if possible. If you use wireless devices, you will need to show the USB dongle to the proctor, which can be awkward if it is connected tot he bottom of a monitor or anywhere else not easily accessible.  
Ensure your workspace is clear, your internet connection is stable, and you have your ID ready.  

---

## **Part I: Discrete Option Multiple-Choice (DOMC) Exam Subjects**

### **Section 1: Planning and Designing an Auth0 Implementation**

Foundational decisions made during the planning and design phase have cascading effects on an application's security, user experience, and extensibility. Exam questions in this domain test the understanding of Auth0's architectural building blocks and the security implications of fundamental configuration choices.

#### **1.1 Application and Client Types: The Definitive Guide**

The selection of an Application Type within the Auth0 Dashboard is a foundational step that dictates available settings and recommended authentication flows. This choice is immutable for Machine-to-Machine applications and is often locked for other types once certain grant types are selected, underscoring its architectural importance.

The four primary application types are:

* **Machine to Machine (M2M):** Designed for non-interactive processes that execute on a backend, such as command-line tools, daemons, or services that need to call an API without direct user involvement.  
* **Native App:** Intended for applications that run natively on a user's device, such as mobile (iOS, Android) or desktop applications.  
* **Regular Web App (RWA):** Represents traditional web applications where the majority of the logic is executed on the server-side, like those built with Express.js or ASP.NET.  
* **Single Page App (SPA):** For modern web applications where the user interface logic is primarily executed in the user's browser, communicating with backend services via APIs. Examples include applications built with React or Angular.

A core concept from the OAuth 2.0 specification that underpins these types is the distinction between confidential and public clients. This classification is based entirely on an application's ability to securely store its **Client Secret**.

* **Confidential Clients:** These are applications, such as RWAs and M2M apps, that operate on a secure server environment. They can be trusted to maintain the confidentiality of credentials like the Client Secret. Consequently, they can authenticate themselves to the Auth0 token endpoint using methods that require this secret, such as  
  client\_secret\_post or client\_secret\_basic, or by using an asymmetric method like private\_key\_jwt.  
* **Public Clients:** These applications, including SPAs and Native Apps, execute in environments that are not controlled by the application owner (e.g., a user's web browser or mobile device). They are fundamentally incapable of guaranteeing the confidentiality of a Client Secret, as it could be extracted by a malicious actor through decompilation or inspection of the application's code or memory.

This distinction has a critical and direct impact on token security. The Client Secret is used for symmetric signing algorithms like HS256, where the same key is used to both sign and verify a token. For a public client to use HS256, it would need to embed the secret within its code, making it vulnerable. An attacker who extracts this secret could forge valid ID tokens, completely compromising user authentication.

To prevent this, public clients **must** use an asymmetric signing algorithm, **RS256**. In this model, Auth0 signs the token with a private key that is kept secret on Auth0's servers. The public client then verifies the token's signature using a corresponding public key, which it retrieves from a publicly accessible JWKS (JSON Web Key Set) endpoint provided by the Auth0 tenant at `https://{yourDomain}/.well-known/jwks.json`. Because the client only needs the public key for verification, it has no secret to protect. Confidential clients, with their ability to protect a secret, are permitted to use either HS256 (symmetric) or the recommended RS256 (asymmetric) algorithm.

The following table consolidates these critical relationships for memorisation.

| Application Type | Client Type | Can Store Secret? | Recommended Flow |
| :---- | :---- | :---- | :---- |
| Regular Web App | Confidential | Yes | Authorization Code |
| M2M App | Confidential | Yes | Client Credentials |
| Single Page App | Public | No | Authorization Code \+ PKCE |
| Native App | Public | No | Authorization Code \+ PKCE |

#### **1.2 URI Configuration: Syntax and Placeholders**

The "Application URIs" section in the application settings is a critical security boundary. Auth0 enforces that it will only redirect users to URLs that have been explicitly whitelisted in these fields, preventing open redirect vulnerabilities.

**Custom domains**
- Enable a **custom domain** early in development to avoid callback URL churn and to match production origins.

* **Allowed Callback URLs:** A comma-separated list of URLs to which Auth0 is permitted to redirect a user after they have successfully authenticated. While the star symbol (\*) can be used as a wildcard for subdomains (e.g., \*.example.com), this practice is strongly discouraged in production environments due to security risks.  
* **Allowed Logout URLs:** A comma-separated list of URLs to which a user can be redirected after logging out, used in conjunction with the returnTo query parameter. This field has a limit of **100 URLs**. Importantly, query strings and hash fragments in the provided URL are ignored during the validation process.  
* **Allowed Web Origins:** A comma-separated list of URLs that are permitted to make requests using Cross-Origin Authentication. This field also has a limit of **100 URLs**, and validation similarly ignores paths, query strings, and hash fragments.

For multi-tenant B2B applications that use the Auth0 Organisations feature, a more secure and dynamic method for handling customer-specific subdomains is available. Instead of using a risky wildcard, the `{organization\_name}` placeholder can be used in the Allowed Callback URLs, for instance: `https://{organization\_name}.myapp.com/callback`.

Using a wildcard like \*.myapp.com creates a vulnerability because an attacker could potentially register a subdomain (e.g., malicious.myapp.com), initiate a login flow, and trick Auth0 into redirecting an authenticated user—along with their sensitive authorization code—to the attacker-controlled server. The {organization\_name} placeholder mitigates this threat. Auth0 will only substitute the names of organizations that are legitimately registered and configured within the tenant. An attacker cannot forge a redirect to an arbitrary subdomain; it must correspond to a known and trusted entity within the Auth0 configuration.

#### **1.3 Token Strategy and Security**

A robust token strategy is essential for maintaining application security and performance.

* **ID Token Expiration:** By default, the lifetime of an ID token is **36,000 seconds (10 hours)**. This is a common value tested for direct recall.  
* **Refresh Token Rotation:** This is a crucial security feature designed to detect and mitigate the impact of leaked refresh tokens. When an application uses a refresh token to obtain a new access token, Auth0 returns not only a new access token but also a **new refresh token**. The original refresh token that was just used is immediately invalidated. If an attacker were to steal and attempt to reuse the old refresh token, Auth0's reuse detection mechanism would be triggered. This action invalidates the entire chain of refresh tokens derived from the original, effectively logging the user out and forcing a full re-authentication for both the legitimate user and the attacker.  
* **Token and Code Size:** The size of authorization codes and access tokens is variable. Applications must be designed to handle this variability, particularly in environments with storage constraints, such as the size limits of browser cookies.  
* **Token Validation:** A critical security step that must be performed by the recipient of a token.  
  * **Access Token (JWT):** The receiving API must validate the token's signature, issuer (iss), audience (aud), and expiration (exp). The aud claim is particularly important; it must match the unique identifier of the API that the token is intended for.  
  * **ID Token (JWT):** The receiving application must validate the signature, issuer (iss), expiration (exp), and, if used, the nonce. The aud claim must match the application's own client\_id.

#### **1.4 User Migration Strategies**

For applications transitioning to Auth0 from a legacy identity system, two primary migration strategies are available.

* **Automatic Migration (Custom Database Connection):** This strategy, often called "lazy migration," provides the most seamless experience for end-users. A custom database connection is created in Auth0 with scripts, including a "Get User" script. When a user attempts their first login via Auth0, the platform executes this script. The script takes the user's provided credentials and attempts to validate them against the legacy user database. If validation is successful, the script returns the user's profile, and Auth0 automatically creates a corresponding user profile in its own store, effectively migrating the user at the moment of login.  
* **Bulk User Import:** This strategy involves a manual, one-time import of user data. It is suitable when direct access to the legacy database is available. User data must be formatted into a specific JSON file structure. This file is then uploaded to Auth0 via the Management API's POST /api/v2/jobs/users-imports endpoint or through the "Bulk User Import" extension in the dashboard. This method typically requires that the legacy password hashes are in a format compatible with Auth0's import requirements.

### **Section 2: Authentication**

This section delves into the mechanics of user authentication, covering the standard protocols, logout procedures, and specific features that define the login experience.

#### **2.1 Authentication Flows: A Comparative Matrix**

Auth0 provides robust support for standard OAuth 2.0 and OIDC flows, abstracting much of their complexity. The appropriate flow is primarily determined by the application's client type.

| Flow Name | Recommended Application Type | Client Type | Key Security Characteristics | Primary Use Case |
| :---- | :---- | :---- | :---- | :---- |
| **Authorization Code Flow with Proof Key for Code Exchange (PKCE)** | Single-Page App (SPA), Native/Mobile | Public | Does not require a client secret. Uses a dynamic proof key (PKCE) to prevent authorization code interception attacks. Considered the most secure flow for public clients. | User authentication and API access for applications that cannot securely store a secret, such as browser-based SPAs and mobile apps. |
| **Authorization Code Flow** | Regular Web App | Confidential | Requires a client secret, which is exchanged securely on the back-channel. Tokens are never exposed to the user's browser. | User authentication and API access for traditional server-side applications that can securely store a client secret. |
| **Client Credentials Flow** | Machine-to-Machine (M2M) | Confidential | Non-interactive flow. The application authenticates itself directly using its client ID and secret to obtain an access token. No user is involved. | Granting API access to backend services, daemons, CLIs, or other non-interactive clients. |
| **Implicit Flow with Form Post** | (Legacy) Single-Page App | Public | **Discouraged.** Tokens are returned directly in the URL fragment, which can expose them to interception. Lacks support for refresh tokens. | Legacy use case for simple user authentication in SPAs where only an ID Token was needed. Superseded by Authorization Code Flow with PKCE. |
| **Resource Owner Password Flow** | (Legacy) Highly Trusted First-Party Apps | Confidential | **Discouraged.** Requires the application to collect the user's username and password directly, breaking the principle of delegation and increasing security risks. Should only be used when redirect-based flows are impossible. | Legacy systems or specific command-line tools where a browser-based redirect is not feasible. |

## OAuth 2.0 / OIDC Flows — Step by Step

This section describes how the main flows work in Auth0, aligned with their recommended application types.  

### Authorization Code Flow with PKCE (for SPAs and Native Apps)

**Who uses it?**  
Single-Page Applications (SPAs) and Native/Mobile apps — public clients that cannot keep a secret.

**Steps:**
1. **App initiates login**  
   - The SPA/native app redirects the browser (or in-app browser) to Auth0’s `/authorize` endpoint.  
   - It includes a **code_challenge** (a hashed random string) and specifies response type `code`.

2. **User authenticates**  
   - User enters credentials, or signs in with an IdP via Universal Login.  
   - Auth0 creates a session (cookie) and validates user identity.

3. **Authorization Code returned**  
   - Auth0 redirects back to the app’s redirect URI with an **authorization code** (in the URL).  
   - No tokens are exposed at this stage.

4. **Token exchange with PKCE**  
   - The app sends the authorization code + its original **code_verifier** (the secret string matching the challenge) to Auth0’s `/oauth/token` endpoint.  
   - No client secret is needed.  
   - Auth0 checks that the hashed verifier matches the earlier challenge.

5. **Tokens issued**  
   - Auth0 returns an **ID token**, **Access token**, and optionally a **Refresh token**.  
   - Tokens are stored securely in memory (for SPAs) or OS-secure storage (for native apps).

**Security Characteristics:**  
- PKCE ensures an attacker who steals the code cannot exchange it without the code_verifier.  
- No client secret used, so safe for public clients.

### Authorization Code Flow (for Regular Web Apps)

**Who uses it?**  
Server-side rendered web apps (Node.js, Django, ASP.NET) — confidential clients that can keep a secret.

**Steps:**
1. **App initiates login**  
   - Browser is redirected to Auth0’s `/authorize` endpoint with `response_type=code`.

2. **User authenticates**  
   - User logs in at Auth0 (Universal Login).  
   - Auth0 sets its session cookie.

3. **Authorization Code returned**  
   - Auth0 redirects to the app’s redirect URI with the **authorization code**.

4. **Server exchanges code with client secret**  
   - The app’s backend (safe environment) sends the code + **client_id** + **client_secret** to `/oauth/token`.  
   - This happens over a secure back-channel (not exposed to the browser).

5. **Tokens issued**  
   - Auth0 returns an **ID token** and **Access token** (and optionally Refresh token).  
   - The server can set a session cookie for the user, hiding tokens from the browser.

**Security Characteristics:**  
- Tokens never travel in the browser URL beyond the short-lived code.  
- The client secret binds the code exchange to the real app, preventing interception.

### Client Credentials Flow (for Machine-to-Machine apps)

**Who uses it?**  
APIs, backend services, daemons, CLIs — confidential clients with no user present.

**Steps:**
1. **App requests a token**  
   - The service posts directly to `/oauth/token` with its **client_id** and **client_secret**.  
   - It includes the **audience** of the API it wants to call.

2. **Auth0 validates credentials**  
   - Auth0 checks the client_id/secret pair.  
   - Ensures the app is authorised to request tokens for that API.

3. **Access token issued**  
   - Auth0 returns an **Access token** scoped for the requested API.  
   - No ID token is returned because no user exists.

4. **App calls the API**  
   - The app uses the access token in the `Authorization: Bearer` header.  
   - The API validates the token (signature, audience, scope).

**Security Characteristics:**  
- Purely app-to-app. No user context.  
- Tokens represent the app’s identity and permissions.  
- Client secret must be kept safe (vault, config store).

**Memory Tips:**
- **PKCE flow:** “Verifier proves the code is mine.”  
- **Regular Code flow:** “Server hides secret, tokens never leak to browser.”  
- **Client Credentials:** “No user, just machines talking with secrets.”

## What is OIDC (OpenID Connect) and What Does It Apply To?

**OpenID Connect (OIDC)** is an identity layer built on top of the OAuth 2.0 framework.  

- **Purpose:** Defines a standard way for applications to verify a user’s identity and obtain profile information.  
- **How it works:** OIDC extends OAuth 2.0 flows by introducing the **ID Token** (a JWT) that contains claims about the authenticated user (e.g. `sub`, `name`, `email`).  
- **Where it applies:**  
  - Logging users into web apps, SPAs, and native/mobile apps.  
  - Enabling Single Sign-On (SSO) across multiple applications.  
  - Any scenario where you need to know *who the user is*, not just whether they can access a resource.  

**Key distinction:** OAuth 2.0 alone is about *authorisation* (can you access this resource?).  
OIDC adds *authentication* (who are you?).

---

### Memory Aid: OAuth 2.0 vs OIDC

| Aspect                | OAuth 2.0 (alone)                           | OIDC (OAuth 2.0 + identity layer)            |
|-----------------------|----------------------------------------------|----------------------------------------------|
| **Primary Purpose**   | Authorisation (granting access to resources) | Authentication + authorisation               |
| **Main Token**        | Access Token                                | Access Token **+ ID Token**                  |
| **ID Token Format**   | N/A                                          | JWT containing identity claims               |
| **Typical Use Case**  | A service calling an API on behalf of a user | A client app logging a user in and knowing who they are |
| **SSO Support**       | Not specified                               | Built-in, using ID Token and standard claims |

**Memory tip:** *OAuth tells you what you can do. OIDC tells you who you are.*

#### **2.2 Logout Scenarios and Session Layers**

Properly implementing logout requires an understanding of the different session layers that can exist during an authenticated user's journey.

There are three distinct session layers to consider:

1. **Application Session Layer:** This is the session managed directly by the application itself, often implemented with a session cookie. The application is solely responsible for creating and clearing this session.  
2. **Auth0 Session Layer:** Auth0 maintains its own session for the user via a cookie set on the authorization server's domain. This session enables Single Sign-On (SSO). Logging out of this layer clears the SSO state.  
3. **Identity Provider (IdP) Session Layer:** If a user authenticates via a federated IdP like Google or a SAML provider, they will also have an active session with that provider.

**RP-Initiated Logout** is the standard OIDC mechanism for logging a user out. The application (the Relying Party, or RP) initiates the process by redirecting the user to Auth0's /oidc/logout endpoint. Key parameters for this endpoint include:

* id\_token\_hint: (Recommended) The ID token issued to the user. This securely identifies the specific session that needs to be terminated.  
* post\_logout\_redirect\_uri: The URL where the user should be redirected after the logout is complete. This URL **must** be registered in the application's or tenant's "Allowed Logout URLs" list.  
* federated: A parameter that, when included, instructs Auth0 to also attempt to log the user out from their upstream federated IdP.

The distinction between application-level and tenant-level logout URLs is a matter of context and specificity. If the logout request includes a client\_id, Auth0 has the context of a specific application and will only permit redirects to URLs listed in that application's "Allowed Logout URLs" setting. If no client\_id is provided, Auth0 falls back to the global list of "Allowed Logout URLs" configured at the tenant level, providing a more generic but less specific security boundary.

**Back-Channel Logout** is a server-to-server mechanism where Auth0 sends a signed logout\_token to a pre-registered endpoint on each application's backend, notifying them to terminate the user's session. This is effective for traditional web apps with server-side sessions.

**Universal Logout** enhances this by also revoking any active refresh tokens for the user, providing a more thorough logout for SPAs and native apps that rely on them.

##### 2.2.1 Backchannel and Frontchannel Logout Details

**Front-channel logout**  
- Browser-based, often through hidden iframes or redirects.  
- Clears application sessions by loading each registered logout URL in the browser.  
- Tied to the user's browser context and cookies.

**Back-channel logout**  
- Server-to-server. Auth0 sends a signed **logout token** to each registered back-channel logout endpoint.  
- Uses the `sid` claim to identify the session to terminate.  
- Suitable for server-side sessions where a browser signal is unreliable.

**OIDC RP-Initiated Logout and global logout**  
- Use `/oidc/logout` with `id_token_hint` and `post_logout_redirect_uri`.  
- Add `federated` when you also need to log out of the upstream IdP, for example `/oidc/logout?federated`.

#### **2.3 Advanced Authentication Patterns**

Auth0 supports several advanced patterns to customize the login experience.

* **Identifier First Authentication:** This pattern splits the login process into two steps. First, the user provides their identifier (e.g., email address). Based on this identifier, Auth0 can then present the appropriate second step, such as a password field or a redirect to an enterprise IdP.  
* **Home Realm Discovery (HRD):** Used with Identifier First, HRD allows Auth0 to route users to the correct identity provider automatically. When a user enters an email address with a specific domain (e.g., user@acme.com), Auth0 checks if the acme.com domain is registered with an enterprise connection. If a match is found, the user is seamlessly redirected to that enterprise's login page. A single enterprise connection can have up to **1000 domains** associated with it for HRD purposes.  
* **Flexible Identifiers:** This feature allows a database connection to be configured to accept an email, username, or phone number as the primary identifier for a user. To use phone number verification during signup, Identifier First authentication must be enabled.

#### **2.4 JWT Structure and OIDC Claims**

A JSON Web Token (JWT) is a compact, self-contained standard for transmitting information as a JSON object. It is composed of three parts, separated by dots: Header.Payload.Signature.

* **Header:** Contains metadata about the token, such as its type (typ) and the signing algorithm (alg).  
* **Payload:** Contains the claims, which are statements about the user and the token's context.  
* **Signature:** A cryptographic signature used to verify that the token was not tampered with and was issued by a trusted party.

OpenID Connect (OIDC) defines a set of standard scopes that an application can request during authentication. These scopes correspond to specific sets of user attributes, or claims, that will be included in the ID token.

| Requested Scope(s) | Key Claims Returned in ID Token |
| :---- | :---- |
| openid | sub (user ID), iss (issuer), aud (audience), exp (expiration), iat (issued at) |
| openid profile | All of the above, plus: name, nickname, picture, given\_name, family\_name |
| openid email | All of the openid claims, plus: email, email\_verified |
| openid profile email | All of the claims from all three scopes |

The openid scope is mandatory for any OIDC-compliant authentication request and signals the intent to authenticate the user.

#### **2.5 Embedded Login and Cross-Origin Authentication**

**When to use**  
Embedded login renders the login experience inside your app, not on the hosted Universal Login domain. It is generally discouraged for new builds, but you may see exam questions on how to enable it safely.

**Key requirements**  
- Enable **Allow Cross-Origin Authentication** for the application.  
- Add your app origin to **Allowed Web Origins** and **Allowed Origins (CORS)**.  
- Add your redirect URI to **Allowed Callback URLs**.  
- For embedded login, **Universal Login is not used** for the primary UI. You are responsible for handling redirects and token storage correctly.

**Notes for SPAs**  
- Prefer Authorization Code with PKCE.  
- Ensure storage is memory-first and avoid long-lived tokens in localStorage.

**Passwordless endpoints to remember**  
- `POST /passwordless/start` to send the email or SMS.  
- `POST /passwordless/verify` to complete the login.

### **Section 3: User Provisioning and Management**

This domain covers the lifecycle of a user account within Auth0, from creation and data storage to access control and multi-tenancy.

#### **3.1 User Profiles: Metadata and Linking**

Auth0 provides a flexible system for storing user data, centered around a normalised user profile and two types of metadata.

* **user\_metadata vs. app\_metadata:** This distinction is a fundamental concept for data storage and access control.  
  * **user\_metadata:** This is a JSON object for storing non-critical user attributes and preferences that the user themselves can typically view and edit, such as a preferred language or profile settings. This data should not impact what the user can or cannot access.  
  * **app\_metadata:** This is a JSON object for storing information that should not be modifiable by the user. It is intended for data that influences authorization, such as assigned roles, subscription levels, or permissions. This data is typically managed by administrators or through automated processes.  
* **Normalized User Profile:** Auth0 ingests user profiles from various sources (e.g., social providers like Google, enterprise directories via SAML, or its own database) and maps them to a consistent, normalised schema. This provides the application with a predictable structure for user data, regardless of how the user authenticated.  
* **Account Linking:** This feature allows multiple identities that belong to the same physical person (e.g., a user who signed up with a username/password and later logged in with Google) to be linked together under a single primary user profile. This provides a unified view of the user and prevents duplicate accounts. Linking can be performed programmatically via the Management API or by using the Account Link extension.

#### **3.2 Role-Based Access Control (RBAC)**

Auth0's core authorization feature set provides a robust implementation of RBAC for securing APIs.

* **Configuration:** RBAC is enabled on a per-API basis within the Auth0 Dashboard. The "Enable RBAC" setting must be toggled on within the API's settings page.  
* **Permissions and Scopes:** Within the context of an Auth0-secured API, permissions are defined as scopes. These are strings representing specific actions that can be performed (e.g., read:financial\_reports, approve:invoices). These permissions are defined in the "Permissions" (or "Scopes") tab of the API's settings.  
* **Permissions in Tokens:** For RBAC to be effective, the permissions granted to a user must be included in the access token. This is achieved by enabling the "Add Permissions in the Access Token" toggle in the API's RBAC settings. When enabled, a successful authentication flow for that API will result in an access token containing a permissions claim. This claim is an array of strings listing all the permissions the user has for that specific API. The API's backend code is then responsible for inspecting this array to enforce authorization decisions.

#### **3.3 Auth0 Organizations for B2B/Multi-Tenancy**

Auth0 Organizations is a feature set designed specifically for B2B and SaaS applications that need to manage distinct groups of users, such as different customer companies.

* **Core Use Case:** Organizations allow for the management of separate user memberships, federated connections (e.g., each customer can bring their own SAML IdP), and light branding for each distinct business customer, all within a single Auth0 tenant.  
* **Key Limitations:**  
- **Enable connections per organisation:** after creating an Organisation, go to the Organisation settings and enable the required connections. Users will not be able to authenticate until the connection is enabled for that organisation.

  * Organizations are only compatible with the **New Universal Login** experience; Classic Login is not supported.  
  * It does not support providing a unique custom domain for each organisation. If customer-a.com and customer-b.com are required as login domains, separate Auth0 tenants are necessary.  
  * The feature is incompatible with certain flows, including the Resource Owner Password Grant (ROPG), Device Authorization Flow, and WS-Federation.  
* **Member Management:**  
  * **Invitation Flow:** To add a user who may not yet have an account, an administrator can send an invitation. The user receives an email with a unique link containing an invitation ticket ID and an organisation ID. The application's login route must be configured to parse these query parameters and pass them to Auth0 during the authentication flow to correctly associate the new user with the organization.  
  * **Direct Assignment:** For users who already exist in the Auth0 tenant, they can be added to an organisation directly via the Management API. This is done by making a POST request to the `/api/v2/organizations/{id}/members` endpoint. The request body must contain a JSON object with a members key, whose value is an array of user ID strings (e.g., `{"members": \["auth0|user1", "google-oauth2|user2"\]}`). A maximum of **10 members** can be added in a single API call.  
* **Tokens and Organizations:** When a user authenticates through an organization-specific login flow, the resulting ID and access tokens will include an org\_id claim containing the unique identifier of that organization. If enabled in the tenant settings, an org\_name claim can also be included. Both the client application (consuming the ID token) and the API (consuming the access token) should validate this claim to ensure that actions are being performed within the correct tenant context.

#### **3.4 Progressive Profiling and Profile Enrichment**

**Progressive Profiling**  
Collect additional attributes after a successful login, for example first-time capture of `phone_number` or `address`.  
- Use a **Post-Login Action** to detect a missing attribute and call `api.redirect.sendUserTo()` to route the user to a first-run profile page.  
- On return, resume the flow and persist attributes with `api.user.setUserMetadata()` or by calling the Management API.

**Profile Enrichment**  
Augment the profile by calling a third-party API after login, for example a CRM or marketing platform.  
- Use a **Post-Login Action** and `fetch` to call the remote API with a tenant secret.  
- Store reference data in `app_metadata` when it affects authorisation, keep preferences in `user_metadata`.  
- If user consent is required, redirect to a consent screen first, then resume.

**Example: redirect for missing phone number**  
```js
exports.onExecutePostLogin = async (event, api) => {
  const needsPhone = !event.user.user_metadata?.phone_number;
  if (needsPhone && event.transaction?.protocol === 'oidc-basic-profile') {
    api.redirect.sendUserTo('https://app.example.com/first-run', {
      query: { email: event.user.email }
    });
  }
};
```

**Custom DB script size limit**  
When using a Custom Database, the combined script size limit is **100 KB**. Keep code lean and move heavy logic to APIs.

### **Section 4: Customization**

Auth0's extensibility features allow developers to inject custom logic and branding into the identity flows. Actions are the modern and recommended approach, superseding the legacy Rules and Hooks systems.

#### **4.1 The Extensibility Hierarchy: Actions vs. Rules vs. Hooks**

Understanding the evolution and current state of Auth0's extensibility is critical.

* **Current Recommendation:** **Actions** are the sole recommended feature for customizing Auth0 flows. **Rules and Hooks are legacy features** with a declared End of Life (EOL) date of **November 18, 2026**. As of October 16, 2023, they are no longer available to be created in new tenants. While the exam will primarily focus on Actions, comparative knowledge is essential.  
* **Execution Order:** In a tenant where all three legacy and modern features might coexist, they execute in a specific, fixed order during the authentication pipeline: **Rules first, then Hooks, and finally Actions**. This is a crucial detail to memorize.  
* **The Evolution to Actions:** The transition from Rules and Hooks to Actions was driven by the need for a more unified, powerful, and developer-friendly extensibility model. Rules were limited to the authentication pipeline, while Hooks handled other extensibility points like pre-user registration or password changes, creating a disjointed experience. Actions consolidate these capabilities into a single framework with a superior developer environment that includes an integrated online editor with code completion, robust versioning and rollback capabilities, secure secrets management, and broad support for public npm packages.

#### **4.2 Anatomy of an Auth0 Action**

An Action is a serverless Node.js function that executes at a specific point (a "trigger") in an Auth0 flow.

* **Key Triggers:**  
  * **Post-Login:** Executes after a user successfully authenticates but before tokens are minted. This is a **blocking (synchronous)** trigger, meaning the authentication flow pauses until the Action completes. It is the primary trigger for use cases like denying access based on custom criteria, adding custom claims to tokens, or redirecting a user for progressive profiling.  
  * **Pre-User Registration:** Executes before a new user profile is created for a database or passwordless connection. This is also a **blocking (synchronous)** trigger. It is used to validate incoming user data or to deny a registration attempt based on business logic.  
  * **Post-User Registration:** Executes after a new user has been successfully created. This is a **non-blocking (asynchronous)** trigger. The registration flow completes without waiting for the Action to finish. It is ideal for side effects like sending a welcome email, creating a user record in a CRM, or other downstream tasks.  
* **The event Object:** This object is passed to every Action and contains read-only contextual information about the current transaction. Key properties include:  
  * event.user: The profile of the user undergoing the transaction, including user\_metadata and app\_metadata.  
  * event.client: The application initiating the transaction.  
  * event.connection: The identity provider connection used.  
  * event.organization: Details of the organization, if applicable.  
  * event.request: Information about the inbound HTTP request, such as ip address and geoip data.  
  * event.secrets: An object used to access securely stored secrets, such as API keys, without exposing them in the code.  
* **The api Object:** This object provides a set of methods that allow the Action's code to modify the behavior of the flow. Key methods include:  
  * api.access.deny(reason, userMessage): Immediately halts the flow and denies the user access. This is available in synchronous triggers like Post-Login and Pre-User Registration.  
  * api.idToken.setCustomClaim(name, value): Adds a namespaced custom claim to the ID token. Only available in the Post-Login trigger.  
  * api.accessToken.setCustomClaim(name, value): Adds a namespaced custom claim to the access token. Only available in the Post-Login trigger.  
  * api.user.setUserMetadata(name, value) and api.user.setAppMetadata(name, value): Programmatically update the user's metadata profile.  
  * api.redirect.sendUserTo(url, `{ query: {...} }`): Redirects the user to an external URL, effectively pausing the authentication flow. This is used for progressive profiling or custom consent steps and is only available in the Post-Login trigger.  
  * api.multifactor.enable(...) and api.authentication.challengeWith(...): Programmatically enforce an MFA challenge, overriding tenant-level policies. Only available in the Post-Login trigger.

#### **4.3 Email Templates and SMTP Configuration**

**Templates**  
Auth0 emails are HTML with **Liquid** syntax. You can format names and conditional content.  
- Example: `{{ user.given_name | upcase }}.{{ user.family_name | upcase }}`  
- Use conditionals to branch by connection or app. Keep logic simple for readability.

**Custom SMTP**  
Configuring a custom SMTP provider is a separate step.  
- Set up under **Dashboard → Emails → Providers**.  
- Templates are under **Dashboard → Emails → Templates**.  
- After enabling a custom provider, emails are sent through your SMTP. Monitor deliverability, from address, and domain settings.

### **Section 5: Security and Attack Protection**

Auth0 provides a layered suite of security features designed to protect against common identity-related threats.

#### **5.1 Multi-Factor Authentication (MFA)**

MFA adds a critical layer of security by requiring users to provide more than one form of verification.

* **Available MFA Factors:** A comprehensive list of supported factors includes 59:  
  * **Push notifications:** Via the Auth0 Guardian mobile app or a custom app using the Guardian SDK.  
  * **One-time passwords (OTP):** Using authenticator apps like Google Authenticator.  
  * **SMS and Voice notifications:** Delivering a one-time code via text message or phone call.  
  * **WebAuthn:** Supporting both roaming security keys (like YubiKey) and platform biometrics (like Windows Hello, Touch ID, Face ID).  
  * **Email notifications:** Sending a one-time code via email.  
  * **Cisco Duo:** An integration with the Duo Security service.  
  * **Recovery codes:** Single-use codes for users who lose access to their primary factors.  
* **MFA Policies:** These global settings, found under Security \> Multi-factor Auth, define the default MFA behavior for the tenant.  
  * **Never:** MFA is off by default.  
  * **Always:** MFA is enforced for every user on every login.  
  * **Use Adaptive MFA:** (Requires Enterprise plan) Auth0 dynamically assesses the risk of each login attempt and only prompts for MFA when the risk is deemed high.  
* **Adaptive MFA Risk Assessors:** The risk score for Adaptive MFA is calculated based on three primary signals 62:  
  1. **NewDevice:** Flags a login from a device that has not been seen for that user in the last 30 days.  
  2. **ImpossibleTravel:** Flags a login from a geographic location that would be impossible to reach in the time elapsed since the user's last login.  
  3. **UntrustedIP:** Flags a login from an IP address that is present on threat intelligence lists for suspicious activity.

A crucial point for the exam is the order of precedence for security policies. MFA behavior defined within a **Post-Login Action** using methods like api.multifactor.enable() will **always override** the global tenant-level MFA policy setting. For example, if the tenant policy is set to "Never," but an Action requires MFA for users with an "admin" role, those admin users will be prompted for MFA.

#### **Understanding the Three Factors of Authentication**

Multi-Factor Authentication (MFA) is grounded in the principle of combining distinct categories of authentication factors to enhance security posture. These factors fall into three core domains:

- **Something You Know** *(Knowledge Factor)*  
  Information the user memorizes and provides during authentication. Examples include passwords, PINs, and answers to security questions. These are susceptible to phishing, brute-force attacks, and credential stuffing.

- **Something You Have** *(Possession Factor)*  
  A physical or digital item the user controls, such as a mobile device, hardware token (e.g., YubiKey), or authenticator app. Possession factors are effective against remote attacks but can be compromised through theft or loss.

- **Something You Are** *(Inherence Factor)*  
  Biometric characteristics unique to the user, including fingerprints, facial recognition, retina scans, or voice patterns. These are difficult to replicate but introduce privacy and data protection considerations.

Auth0’s MFA implementation supports a wide array of factors across these domains. For example:
- OTP via Google Authenticator (Possession)
- WebAuthn with biometrics (Inherence)
- Passwords (Knowledge)

The strength of MFA lies in requiring **at least two factors from different categories**, significantly increasing resistance to account compromise. For instance, even if a password is leaked (Knowledge), access is denied without the second factor (e.g., OTP or biometric).

This triad forms the conceptual backbone of MFA and is essential for understanding both the **security rationale** and **implementation strategy** behind Auth0’s MFA features.

#### **5.2 Attack Protection Suite**

Auth0's Attack Protection features are designed to automatically detect and mitigate various automated and targeted attacks.

* **Brute-Force Protection:** This feature protects individual user accounts from repeated, targeted login attempts. It blocks a user after a configurable number of consecutive failed login attempts (default is 10; configurable between 1 and 100). The block can be configured to be specific to the user and IP address (count\_per\_identifier\_and\_ip) or to lock out the user account regardless of the source IP (count\_per\_identifier).  
* **Suspicious IP Throttling:** This feature is designed to counter large-scale, distributed attacks like credential stuffing, where an attacker uses a single IP to try credentials against many different user accounts. It blocks or throttles traffic from any IP address that generates an unusually high rate of login or signup attempts in a short period.  
* **Bot Detection:** This feature uses risk assessment models to identify traffic that is likely automated. When a risky login is detected, it can challenge the user with a CAPTCHA. The policy can be set to Never, Always, or When Risky. The "When Risky" setting allows for tuning the sensitivity with Low, Medium, or High detection levels. Auth0 provides its own "Auth Challenge" (the default, requires JavaScript) and "Simple CAPTCHA" (no JS required, but has accessibility limitations), and also supports integration with third-party providers.  
* **Breached Password Detection:** This feature checks passwords used during signup, login, or password reset against a continuously updated database of credentials known to have been exposed in third-party data breaches. If a match is found, Auth0 can block the action, preventing the use of compromised passwords.

#### **5.3 Mitigating Core Web Threats**

Auth0 provides built-in mechanisms to defend against fundamental web application attacks.

* **Cross-Site Request Forgery (CSRF):** In an OAuth 2.0 context, a CSRF attack could involve an attacker tricking an already authenticated user into authorizing the attacker's account, linking the victim's data to the attacker. The **state** parameter is the primary defense. The client application must generate a unique, unpredictable value for the state parameter and store it locally (e.g., in the user's session) before redirecting to Auth0. Auth0 will include this exact value in the redirect back to the application. The application must then verify that the returned state parameter matches the value it originally stored. A mismatch indicates a potential CSRF attack, and the authentication response must be rejected.  
* **Replay Attacks:** A replay attack occurs when an attacker captures a valid token and reuses it to impersonate the user. For ID tokens, the **nonce** parameter is the defense. Similar to state, the application generates a unique value for nonce and sends it in the authentication request. Auth0 then includes this same nonce value as a claim inside the signed ID token. The application must verify that the nonce claim in the received ID token matches the value it sent for that specific request. This ensures the token was freshly generated for the current transaction and is not a replayed token from a previous one. The use of the nonce parameter is **required** when using the Implicit Flow.

#### **5.4 Step-Up Authentication and Adaptive MFA**

**Step-Up Authentication**  
- **What it is:** Prompt for MFA when a **sensitive resource** is accessed. Trigger is **resource sensitivity**, not risk.  
- **When it happens:** **After login**, at the time of accessing a protected action or page.  
- **Implementation patterns:**  
  - **Web apps:** Check the ID token or session for `amr` containing `"mfa"`. If missing and the user is accessing a high-value page, require MFA.  
  - **APIs:** Model high-value actions as **restricted scopes** such as `view:balance` or `transfer:funds`. Only issue these when the user has done MFA.  
  - **Actions:** In a **Post-Login Action**, detect the requested resource or scopes and call `api.multifactor.enable()` to force MFA when needed.

**Adaptive MFA**  
- **What it is:** MFA prompts based on **risk**. Triggered only when a login looks risky.  
- **Signals used in confidence scoring:**  
  1. **Device** - new or unrecognised device.  
  2. **IP reputation** - untrusted or risky IP.  
  3. **Impossible Travel** - location jump too fast to be legitimate.  
- **Event object tip:** `event.authentication.riskAssessment.assessments.ImpossibleTravel` can be inspected in Actions when available.  
- **Note:** Adaptive MFA only triggers **during login**, not on resource access.

**Step-Up vs Adaptive MFA - quick view**

| Feature | Step-Up | Adaptive MFA |
|---|---|---|
| **Trigger** | Accessing a sensitive resource | Detected risk at login |
| **Timing** | Post-login | During login |
| **Use case** | API or high-value page access | Smart MFA prompts |
| **Key signal** | Resource sensitivity | Risk score (device, IP, travel) |

**Dashboard quick steps**  
- Configure global MFA policies under **Security → Multi-factor Auth → Define Policies**.  
- Reset a user’s MFA from **User Management → Users → Security**, or **Security → Multi-factor Auth**, or via the Management API by **deleting MFA enrolments**.

**Action snippet for Step-Up by scope**  
```js
exports.onExecutePostLogin = async (event, api) => {
  const needsHighValue = (event.authorization?.requested_permissions || []).some(p =>
    ['view:balance', 'transfer:funds'].includes(p)
  );
  const hasMfa = (event.authentication?.methods || []).some(m => m.name === 'mfa') 
                 || (event.user.multifactor && event.user.multifactor.length > 0);

  if (needsHighValue && !hasMfa) {
    api.multifactor.enable('any');
    // Optionally short-circuit issuing the high-value scopes until MFA is completed
    api.accessToken.setCustomClaim('https://example.com/high_value_pending', true);
  }
};
```

#### **5.5 Assurance Levels and Phishing Resistance (Passkeys)**

**Authentication Assurance Levels (AAL)**

| Level | Description | Example factors |
|---|---|---|
| **AAL1** | Basic single factor | Password only |
| **AAL2** | Two factors, not necessarily phishing resistant | Password + OTP |
| **AAL3** | Requires phishing-resistant factor | WebAuthn, FIDO security key |

**Passkeys and FIDO2/WebAuthn**  
- Passkeys are WebAuthn credentials bound to the **origin**. They resist phishing because a fake domain cannot satisfy the cryptographic origin check.  
- WebAuthn platform authenticators and roaming keys are accepted as **phishing-resistant** and satisfy **AAL3** when policy requires it.

**Passkey policies**  
- **Progressive enrolment:** invite users to add a passkey after a successful sign-in.  
- **Challenge policy:** prompt users to use a passkey when device support is detected.  
- **Local enrolment:** allow adding a passkey directly from the device during login.

### 5.6 WebAuthn (FIDO2)

| Concept | Explanation |
|----------|--------------|
| **Definition** | **WebAuthn** (Web Authentication API) is a W3C standard that allows users to authenticate using **public key cryptography** rather than passwords. It forms part of the **FIDO2** standard (a collaboration between the FIDO Alliance and W3C). |
| **Purpose** | Provides strong, phishing-resistant authentication using hardware or built-in authenticators such as **YubiKey**, **Windows Hello**, **Touch ID**, or **Face ID**. It removes dependence on shared secrets and enables both **passwordless** and **MFA** experiences. |
| **How It Works** | 1. **Registration:** The browser and authenticator generate a **key pair**. The **public key** is sent to Auth0 and stored in the user’s profile; the **private key** never leaves the device.<br>2. **Authentication:** Auth0 sends a challenge to the browser, the authenticator signs it with the private key, and Auth0 verifies the signature using the stored public key. |
| **Security Model** | Each credential is bound to the combination of **user + device + origin**, preventing phishing and replay attacks. Because the private key never leaves the authenticator, it cannot be stolen or reused elsewhere. |
| **Supported Authenticators** | - **Platform authenticators:** built into the device (e.g. Windows Hello, Touch ID).<br>- **Roaming authenticators:** external devices (e.g. YubiKey, Titan Key). |
| **Auth0 Integration** | - **Authentication → MFA → WebAuthn (FIDO2):** used as a second factor after password login.<br>- **Authentication → Passwordless → WebAuthn:** used as a primary passwordless method.<br>- Can also be triggered through **Actions** or **Step-Up MFA** policies for sensitive resources. |
| **Relation to AALs** | WebAuthn authenticators are considered **phishing-resistant** and can satisfy **AAL3** when used as a factor. |

#### 5.6.1 Behaviour on First Login and New Devices

- **First Login:**  
  WebAuthn credentials must be registered before use. On a first login, the user authenticates using an existing method (e.g. password or social IdP). Auth0 then offers to **enrol WebAuthn** by creating and storing a key pair.  
  Future logins can then use that credential for passwordless or MFA authentication.

- **New Devices:**  
  WebAuthn credentials are **device-bound**. When logging in from a new device, the user must **register a new WebAuthn credential** after authenticating through another method (password, recovery, or backup factor). Each device has its own unique key pair.

**Tip:** Think of WebAuthn credentials as *keys tied to devices, not accounts*. Each trusted device must enrol its own key.

#### 5.6.2 Summary Table

| Mode | Prompt Timing | Purpose | Example |
|------|----------------|----------|----------|
| **Passwordless WebAuthn** | Prompted **first**, replaces password | Passwordless login | “Touch your security key to sign in.” |
| **MFA WebAuthn** | Prompted **after password** | Second factor (AAL2/AAL3) | “Enter password → Touch your key.” |
| **New Device / First Use** | Not auto-prompted before authentication | Must enrol a new credential after identity is verified | Register device key after first login. |

**Key takeaways**
- WebAuthn is part of **FIDO2** and uses public-key cryptography for phishing-resistant logins.  
- It can operate as **passwordless** or **MFA** depending on configuration.  
- Each credential is unique to a device; new devices require new enrolment.  
- WebAuthn authenticators meet the highest **Authentication Assurance Level (AAL3)**.

### **Section 6: Monitoring and Logging**

#### **6.1 Interpreting Auth0 Logs**

Auth0 provides detailed logs for all tenant activity, which are essential for monitoring, troubleshooting, and security analysis.

* **Key Log Event Type Codes:** Familiarity with common log codes is vital for quickly diagnosing issues.  
  * s: Success Login  
  * f: Failed Login  
  * fu: Failed Login (invalid username/email)  
  * fp: Failed Login (incorrect password)  
  * limit\_wc: Account blocked by Brute-Force Protection  
  * limit\_mu: IP address blocked by Suspicious IP Throttling  
  * api\_limit: A rate limit on an Auth0 API has been exceeded  
  * pla: Pre-login assessment performed by Bot Detection.  
* **Log Streams:** While the Auth0 Dashboard provides a view of recent logs, their retention period is limited by the subscription plan. For long-term storage, compliance, and advanced analysis, **Log Streams** should be configured. This feature allows log events to be streamed in near real-time to external systems like SIEMs (Security Information and Event Management), data warehouses, or monitoring tools via pre-built integrations or a custom webhook.

### **Section 7: Auth0 Management API**

#### **7.1 Automating with the Management API**

The Management API (v2) is a RESTful API that enables programmatic control over nearly all aspects of an Auth0 tenant.

* **Purpose:** It allows for the automation of administrative tasks, such as creating applications, managing users, configuring connections, and deploying Actions, which would otherwise be performed manually in the Dashboard.  
* **Authentication:** Access to the Management API is protected and requires a bearer token. This access token must be obtained by an authorized application, typically a confidential M2M application, using the Client Credentials flow. The M2M application must be granted specific permissions (scopes) to call the desired API endpoints.  
* **Pagination:** For endpoints that can return large lists of items, the Management API supports two pagination methods 77:  
  * **Offset-based Pagination:** Uses page (the page number, starting from 0\) and per\_page (the number of items per page) query parameters. It is simple to implement but can have performance implications on very large datasets. It is best suited for collections expected to have fewer than 1,000 items.  
  * **Checkpoint-based Pagination:** A more performant method for large datasets. The API response includes a next checkpoint ID. To get the next page of results, the client sends this ID in the from query parameter, along with a take parameter for the page size. This method is forward-only, and each checkpoint ID is valid for **24 hours**.  
* **Rate Limiting:** The Management API is subject to strict rate limits to ensure platform stability. Production applications must be built to handle 429 Too Many Requests responses gracefully, typically by implementing an exponential backoff and retry strategy.

**Retry guidance**
- Use exponential backoff with jitter.
- Honour `X-RateLimit-Reset` headers when present.
- For checkpoint pagination, retain the next checkpoint id for up to 24 hours.

#### 7.2 Naming convention

- Multi-word resource names use **hyphens**, not camelCase.  
  Examples: `jobs/users-exports`, `tickets/email-verification`, `user-blocks`.  
- Many-to-many relationships exist **on both sides**,  
  e.g. `/users/{id}/roles` and `/roles/{id}/users`.  
- Actions (like “generate ticket” or “start job”) are modelled as **nouns** that you **POST** to.

#### 7.3 High-yield areas and supported verbs
*(Paths shown without the required `/api/v2` prefix.)*

| Area | Path fragment | Supported verbs | Notes |
|------|----------------|-----------------|-------|
| Users, collection | `/users` | GET, POST | List or search users. Create requires `connection`. |
| User, single | `/users/{id}` | GET, PATCH, DELETE | Read, update, or delete a user by ID. |
| User roles | `/users/{id}/roles` | GET, POST, DELETE | Manage roles for a user. |
| User permissions | `/users/{id}/permissions` | GET, POST, DELETE | Manage direct permissions for a user. |
| Roles, collection | `/roles` | GET, POST | List or create roles. |
| Role relationships | `/roles/{id}/permissions` | GET, POST, DELETE | List, add, or remove permissions on a role. |
| Role reverse lookup | `/roles/{id}/users` | GET | List users who have a given role. |
| Organisations, members | `/organizations/{id}/members` | GET, POST, DELETE | List, add, or remove members. |
| Org member roles | `/organizations/{id}/members/{user_id}/roles` | GET, POST, DELETE | Manage roles scoped to an organisation. |
| Jobs, bulk export | `/jobs/users-exports` | POST | Start a bulk user export job. |
| Jobs, bulk import | `/jobs/users-imports` | POST | Start a bulk user import job. |
| Jobs, status & errors | `/jobs/{id}` · `/jobs/{id}/errors` | GET | Poll job completion or fetch errors. |
| Tickets | `/tickets/email-verification` · `/tickets/password-change` | POST | Generate verification or password-reset URLs. |
| Attack protection | `/user-blocks/{id}` | GET, DELETE | Read or clear brute-force blocks for a user ID. |

#### 7.4 Quick heuristics

- **Relationship** → `/{collection}/{id}/{subcollection}`  
  Example: `/users/{id}/roles`
- **Bulk tasks** → under `/jobs/` with hyphenated names, `POST` to start.  
- **One-off links** → under `/tickets/`, `POST` to generate.  
- **Multi-word names** → hyphenated, never camelCase.  
- **All start with** `/api/v2` — authentication API endpoints like `/authorize` and `/oidc/logout` are *not* part of this set.

---

## **Part II: Practical Application and Hands-On Task Preparation**

This section provides practical, step-by-step guides for the hands-on use cases that are assessed in Part II of the Okta Certified Developer \- Auth0 exam. Each guide is designed to be a concise lab, walking through the necessary configurations in the Auth0 Dashboard and explaining the core concepts involved.

### **Task 1: Authentication**

This set of tasks focuses on configuring various methods for user authentication.

#### **1.1 Set up Single Sign-On (SSO)**

# Configure a SAML SSO Integration (Auth0 as IdP)
Follow the codelab here:  
[Configure Database and Passwordless Connections in Auth0](https://learning.okta.com/path/discover-the-differences-among-auth0-application-types/lab-configure-single-page-applications-with-auth0/2104232)

#### **1.2 Implement Passwordless Authentication**

This lab demonstrates how to set up **passwordless login** using an email magic link.

Follow the codelab here:  
[Configure Database and Passwordless Connections in Auth0](https://learning.okta.com/path/authenticate-with-auth0-database-and-passwordless-connections/lab-configure-database-and-passwordless-connections-in-auth0/2107572)

**What you’ll practice:**
- Enabling passwordless authentication in Auth0.  
- Configuring email magic links as the login mechanism.  
- Testing the flow end-to-end.

#### **1.3 Create Custom Database**

This lab shows how to connect Auth0 to an external, legacy user database.

Detailed guide: [Create Custom Database Connections](https://auth0.com/docs/authenticate/database-connections/custom-db/create-db-connection)

## Steps in the Auth0 Dashboard

1. **Create a new database connection**
   - Go to **Dashboard → Authentication → Database**.
   - Click **Create DB Connection**.
   - Enter a **Name** (alphanumeric + dashes, ≤ 35 chars).
   - Optional settings:
     - *Requires Username* → users must register with both username + email.
     - *Username Length* → set min/max lengths.
     - *Disable Signups* → block self-service signup; only admins can create users.
   - Click **Create**.

2. **Enable custom database**
   - Open the new connection.
   - Go to the **Custom Database** tab.
   - Toggle **Use my own database** → *ON*.

3. **Create action scripts**
   - Required: **Login** script.  
     - Authenticates the user against your DB.  
     - Must return a user object with unique `id` (prefix with connection name to avoid collisions).  
   - Optional:  
     - **Create** (signup new users).  
     - **Verify** (post-email verification).  
     - **Change Password**.  
     - **Get User** (retrieve profile by email).  
     - **Delete** (remove user).
   - Use **Templates** dropdown (MySQL, PostgreSQL, MongoDB, etc.) to scaffold code.
   - Modify with your DB details (host, username, password, schema).
   - Example (MySQL login):

     ```js
     function login(email, password, callback) {
       var bcrypt = require('bcrypt');
       var mysql = require('mysql');
       var connection = mysql.createConnection({
         host: 'localhost',
         user: 'me',
         password: 'secret',
         database: 'mydb'
       });
       connection.connect();
       connection.query(
         "SELECT id, nickname, email, password FROM users WHERE email = ?",
         [email],
         function (err, results) {
           if (err || results.length === 0) 
             return callback(new WrongUsernameOrPasswordError(email));
           var user = results[0];
           bcrypt.compare(password, user.password, function (err, isValid) {
             if (err || !isValid) return callback(new WrongUsernameOrPasswordError(email));
             return callback(null, {
               id: 'MyConnection|' + user.id.toString(),
               nickname: user.nickname,
               email: user.email
             });
           });
         }
       );
     }
     ```

   - Click **Save and Try** to test the script.

4. **Attach connection to apps**
   - Go to the **Applications** tab of the connection.
   - Enable it for your target app(s).

✅ At this point, Auth0 will call your scripts whenever users authenticate, sign up, reset password, or are managed in your legacy database.

#### **1.4 Set up Social Connections**

This lab demonstrates how to configure **social logins** (e.g., Google, Facebook, GitHub) in Auth0.

Follow the codelab here:  
[Configure Connections in Auth0](https://learning.okta.com/path/authenticate-with-auth0-social-and-enterprise-connections/lab-configure-connections-in-auth0/2107573)

**What you’ll practice:**
- Enabling a social connection in the Auth0 Dashboard.  
- Providing client ID and secret from the social provider.  
- Attaching the connection to your application.  
- Testing login through the social provider.

### **Task 2: Managing Users**

These tasks focus on user data management, including migration and authorization.

#### **2.1 Implement Bulk User Migration**

This section explains how to bulk migrate users into Auth0. You can do this in the Dashboard or via the Management API.

Docs: [Bulk User Imports](https://auth0.com/docs/manage-users/user-migration/bulk-user-imports)

### Prerequisites
- Admin/editor access to your Auth0 tenant.  
- A target **Database Connection** to hold the migrated users.  
- A JSON file of users that follows the **Bulk Import schema**.

### Option A — Dashboard: Bulk import via UI
1. **Open Users:** In the Auth0 Dashboard, go to **User Management → Users**.  
2. **Start Import:**  
   - If your tenant has no users yet, click **Import Users**.  
   - If users exist, choose **Import/Export Users → Import Users**.  
3. **Upload file:** Select your **users JSON** file, making sure it follows the import schema.  
4. **Choose destination connection:** Pick the **Database Connection** to import into.  
5. **Run the import:** Submit the job and wait for completion.  
6. **Verify:** Once complete, spot-check a few imported users and perform a test login.

> Notes for the UI path  
> - The **users JSON** must match the documented schema.  
> - The Dashboard importer supports files up to a small size, so for large migrations use the API method below and split files as needed.

### Option B — Management API: Bulk import job
1. **Prepare JSON:** Build a users JSON file that follows the **Bulk Import schema**.  
2. **Pick connection:** Identify the **connection_id** of your destination database connection.  
3. **Create job:** Call **POST** `/api/v2/jobs/users-imports` with multipart form data:  
   - `users` = your JSON file  
   - `connection_id` = target database connection  
   - Optional flags like `upsert` and `send_completion_email`  
4. **Poll job status:** Use the returned `job_id` to check progress until it completes.  
5. **Verify and test:** Spot-check imported users and perform a test login.

### After import
- **Attach the connection** to any Applications that need it.  
- **Test authentication** end-to-end with a few migrated accounts.  
- **Roll out** to production once validated.

### Tips and gotchas
- **Schema is strict:** Field names and structure must match the documented **Bulk Import schema**.  
- **Passwords:** If your legacy passwords are stored as supported hashes, include the documented hash fields so users can keep their passwords without resets.  
- **Large migrations:** Split input into multiple files and queue more jobs after prior ones finish. Tenants are limited to a small number of concurrent import jobs.  
- **Dry run first:** Test with 1–5 users to validate schema and attributes before running the full migration.

#### **2.2 Set up Organizations and Access Control/Roles**

This lab demonstrates how to configure **Organizations** in Auth0 to manage B2B users and apps, including setting up **roles** and **access control**.

Follow the codelab here:  
[Manage B2B Users and Apps with Auth0 Organizations](https://learning.okta.com/path/manage-b2b-users-and-apps-with-auth0-organizations/lab-manage-b2b-users-and-apps-with-auth0-organizations/2107578)

#### **2.3 Enrich Tokens (Custom Claims)**

This lab demonstrates adding a custom claim to a token using an Action.

1. **Add Metadata to a User:** Navigate to User Management \> Users and select a test user. In the "Metadata" section, under app\_metadata, add the following JSON: `{"plan": "premium"}`. Save the changes.  
2. **Create a Post-Login Action:** Navigate to Actions \> Library and create a new "Login / Post Login" action from scratch. Name it "Add Plan Claim".  
3. **Write the Action Code:** In the editor, add the following code to read the metadata and add it as a custom claim to both the ID and Access Tokens. The claim must be namespaced.  
    ```javaScript  
   exports.onExecutePostLogin \= async (event, api) \=\> {  
     const namespace \= 'https://myapp.example.com/';  
     if (event.user.app\_metadata.plan) {  
       api.idToken.setCustomClaim(\`${namespace}plan\`, event.user.app\_metadata.plan);  
       api.accessToken.setCustomClaim(\`${namespace}plan\`, event.user.app\_metadata.plan);  
     }  
   };
   ```

4. **Deploy and Attach:** Deploy the Action and drag it into the Login flow. Click Apply.  
5. **Verify:** Log in as the test user and obtain an ID or Access Token. Decode it and verify that it now contains the claim https://myapp.example.com/plan: "premium".

### **Task 3: Customization**

These tasks cover customizing the appearance and behavior of the Auth0 platform.

#### **3.1 Set up Branding**

This lab demonstrates how to customize the **Universal Login** page with your own branding.

Follow the codelab here:  
[Customize Auth0 Universal Login](https://learning.okta.com/lab-customize-auth0-universal-login/2104220)

#### **3.2 Create Custom Actions**

This lab shows how to extend Auth0 with **Actions**, adding custom logic to the authentication pipeline.

Follow the codelab here:  
[Working with Actions](https://learning.okta.com/path/extend-auth0-with-actions/lab-working-with-actions/2107574)

### **Task 4: Security**

This task focuses on enhancing security by enabling Multi-Factor Authentication.

#### **4.1 Set up Multi-Factor Authentication**

This lab covers enabling and configuring **MFA** to secure user logins.

Follow the codelab here:  
[Secure Auth0 Applications with MFA](https://learning.okta.com/path/secure-auth0-applications-with-mfa/lab-secure-auth0-applications-with-mfa/2100997)

## **Part III: Useful Documentation**

### Applications and App Types
- [Application Credentials](https://auth0.com/docs/secure/application-credentials)
- [Application Grant Types](https://auth0.com/docs/get-started/applications/application-grant-types)
- [Application Settings](https://auth0.com/docs/get-started/applications/application-settings)
- [Applications in Auth0](https://auth0.com/docs/get-started/applications)
- [Confidential and Public Applications](https://auth0.com/docs/get-started/applications/confidential-and-public-applications)
- [Discover the Differences Among Auth0 Application Types](https://learning.okta.com/path/discover-the-differences-among-auth0-application-types)
- [Register Regular Web Applications](https://auth0.com/docs/get-started/auth0-overview/create-applications/regular-web-apps)

### Authentication, Protocols, and Flows
- [Add Login Using the Authorization Code Flow](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow/add-login-auth-code-flow)
- [Authenticate](https://auth0.com/docs/authenticate)
- [Authentication and Authorization Flows](https://auth0.com/docs/get-started/authentication-and-authorization-flow)
- [Client Credentials Flow](https://auth0.com/docs/get-started/authentication-and-authorization-flow/client-credentials-flow)
- [Protocols](https://auth0.com/docs/authenticate/protocols)
- [What is OAuth 2.0 and what does it do for you?](https://auth0.com/intro-to-iam/what-is-oauth-2)
- [Which OAuth 2.0 Flow Should I Use?](https://auth0.com/docs/get-started/authentication-and-authorization-flow/which-oauth-2-0-flow-should-i-use)

### Tokens and JWT
- [JSON Web Key Sets](https://auth0.com/docs/secure/tokens/json-web-tokens/json-web-key-sets)
- [JSON Web Token Introduction](https://jwt.io/introduction)
- [OpenID Connect Scopes](https://auth0.com/docs/get-started/apis/scopes/openid-connect-scopes)
- [Validate Access Tokens](https://auth0.com/docs/secure/tokens/access-tokens/validate-access-tokens)
- [Validate ID Tokens](https://auth0.com/docs/secure/tokens/id-tokens/validate-id-tokens)
- [Validate JSON Web Tokens](https://auth0.com/docs/secure/tokens/json-web-tokens/validate-json-web-tokens)
- [What Are Refresh Tokens and How to Use Them Securely](https://auth0.com/blog/refresh-tokens-what-are-they-and-when-to-use-them/)

### Logout and Single Sign-on
- [Log Users Out of Auth0 with OIDC Endpoint](https://auth0.com/docs/authenticate/login/logout/log-users-out-of-auth0)
- [Logout](https://auth0.com/docs/authenticate/login/logout)
- [Single Sign-On](https://auth0.com/docs/authenticate/single-sign-on)
- [Universal Logout](https://auth0.com/docs/authenticate/login/logout/universal-logout)

### Universal Login and Identifiers
- [Configure Identifier First Authentication](https://auth0.com/docs/authenticate/login/auth0-universal-login/identifier-first)
- [Flexible Identifiers and Attributes](https://auth0.com/docs/authenticate/database-connections/flexible-identifiers-and-attributes)

### Organizations and Multi-tenant
- [Add members to an organization | Management API v2](https://auth0.com/docs/api/management/v2/organizations/post-members)
- [Assign Members to an Organization](https://auth0.com/docs/manage-users/organizations/configure-organizations/assign-members)
- [Auth0 Organizations](https://auth0.com/docs/manage-users/organizations)
- [Invite Organization Members](https://auth0.com/docs/manage-users/organizations/configure-organizations/invite-members)
- [Realizing multi-tenant authentication using Auth0 Organizations function](https://www.macnica.co.jp/en/business/security/manufacturers/okta/auth0_organizations.html)
- [Understand How Auth0 Organizations Work](https://auth0.com/docs/manage-users/organizations/organizations-overview)
- [Work with Tokens and Organizations](https://auth0.com/docs/manage-users/organizations/using-tokens)

### Actions, Rules, Hooks, and Triggers
- [Actions Triggers: post-challenge — Event Object](https://auth0.com/docs/customize/actions/explore-triggers/password-reset-triggers/post-challenge-trigger/post-challenge-event-object)
- [Actions Triggers: post-login — API Object](https://auth0.com/docs/customize/actions/explore-triggers/signup-and-login-triggers/login-trigger/post-login-api-object)
- [Actions Triggers: post-login — Event Object](https://auth0.com/docs/customize/actions/explore-triggers/signup-and-login-triggers/login-trigger/post-login-event-object)
- [Actions Triggers: pre-user-registration — API Object](https://auth0.com/docs/customize/actions/explore-triggers/signup-and-login-triggers/pre-user-registration-trigger/pre-user-registration-api-object)
- [Actions Triggers: pre-user-registration — Event Object](https://auth0.com/docs/customize/actions/explore-triggers/signup-and-login-triggers/pre-user-registration-trigger/pre-user-registration-event-object)
- [Auth0 Hooks](https://auth0.com/docs/customize/hooks)
- [Auth0 Rules](https://auth0.com/docs/customize/rules)
- [Introducing Auth0 Actions](https://auth0.com/blog/introducing-auth0-actions/)
- [Migrating Auth0 Hooks to Auth0 Actions](https://auth0.com/blog/migrating-auth0-hooks-to-auth0-actions/)
- [Redirect with Actions](https://auth0.com/docs/customize/actions/explore-triggers/signup-and-login-triggers/login-trigger/redirect-with-actions)
- [Rules Best Practices](https://auth0.com/docs/rules-best-practices)
- [Sample Use Cases: Actions with Authorization](https://auth0.com/docs/manage-users/access-control/sample-use-cases-actions-with-authorization)
- [Using Actions to Customize Your MFA Factors](https://auth0.com/blog/using-actions-to-customize-your-mfa-factors/)

### Multi-factor Authentication
- [Adaptive MFA](https://auth0.com/docs/secure/multi-factor-authentication/adaptive-mfa)
- [Customize Multi-Factor Authentication Pages](https://auth0.com/docs/secure/multi-factor-authentication/customize-mfa)
- [Enable Adaptive MFA](https://auth0.com/docs/secure/multi-factor-authentication/adaptive-mfa/enable-adaptive-mfa)
- [Multi-Factor Authentication (MFA)](https://auth0.com/docs/secure/multi-factor-authentication)
- [Multi-Factor Authentication Factors](https://auth0.com/docs/secure/multi-factor-authentication/multi-factor-authentication-factors)
- [Multifactor Authentication (MFA) — Learn](https://auth0.com/learn/get-started-with-mfa)

### Attack Protection and Security
- [Attack Protection](https://auth0.com/docs/secure/attack-protection)
- [Bot Detection](https://auth0.com/docs/secure/attack-protection/bot-detection)
- [Brute-Force Protection](https://auth0.com/docs/secure/attack-protection/brute-force-protection)
- [Prevent Attacks and Redirect Users with OAuth 2.0 State Parameters](https://auth0.com/docs/secure/attack-protection/state-parameters)
- [Prevent Common Cybersecurity Threats](https://auth0.com/docs/secure/security-guidance/prevent-threats)
- [Suspicious IP Throttling](https://auth0.com/docs/secure/attack-protection/suspicious-ip-throttling)
- [View Attack Protection Log Events](https://auth0.com/docs/secure/attack-protection/view-attack-protection-events)
- [auth0_attack_protection | Terraform Registry](https://registry.terraform.io/providers/auth0/auth0/latest/docs/resources/attack_protection)

### Monitoring and Logs
- [How to Monitor Bot Protection Usage — Community](https://community.auth0.com/t/how-to-monitor-bot-protection-usage/174821)
- [Log Type Codes](https://auth0.com/docs/deploy-monitor/logs/log-event-type-codes)

### APIs and Management API
- [Auth0 APIs](https://auth0.com/docs/api)
- [Create Machine-to-Machine Applications for Testing](https://auth0.com/docs/get-started/apis/create-m2m-app-test)
- [Introduction | Auth0 Management API v2](https://auth0.com/docs/api/management/v2)

### Tutorials and Overviews
- [Token Based Authentication Made Easy](https://auth0.com/learn/token-based-authentication-made-easy)

### Exam Resources
- [Auth0 Certified Developer Exam Study Guide](https://certification.okta.com/page/developer-auth0-exam-study-guide)

### Third-party explainers and blogs
- [Auth0: Key Features, Technical Overview, and Alternatives](https://frontegg.com/guides/auth0)
- [Authorization Code Flow (Refresh Token) in Auth0](https://www.macnica.co.jp/en/business/security/manufacturers/okta/tech_auth0_acfre.html)
- [OAuth Application Types. Confidential vs Public Applications](https://medium.com/identity-beyond-borders/oauth-application-types-729764af73d5)
- [What is the difference between public and confidential clients](https://blog.logto.io/reveal-public-and-confidential-clients)
