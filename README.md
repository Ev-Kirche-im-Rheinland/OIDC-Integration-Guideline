# openid connect integration guideline 

## scope
This document only covers the OIDC integration guideline. SAML integration remains unchanged to the previous integration guideline, but the given metadata.xml will change to metadata.xml. A future goal is to combine SAML and OIDC integration guideline into one document that covers both requirements.

## about openid connect 
OpenID Connect is a protocol that enables authentication and authorization between an identity provider (IdP) and a client application (business application, or relaying party in the context of OpenID Connect). It builds on the OAuth 2.0 framework and provides a standard way for clients to authenticate users and obtain information about their identity.

## identity provider (IdP)
The ekir portal provides a central identity provider (IdP) for authenticating portal users.

## requirements & terminology
To make sure the integration is successful, the vendor is responsible to ensure all of these:

-	Whenever this document refers to a "business application" in the context of OpenID Connect, it is understood to mean a relaying parties (RP) as defined in the OpenID Connect specifications.

-	For brevity, throughout this document, the term "business application" will be abbreviated as "BA".
-	All identities are provided from a central Single Sign-On (SSO) identity provider (hereafter abbreviated to "IdP") managed by EKiR through the EKiR portal.

-	It should be emphasized that in the context of OpenID Connect, the IdP does not implement authorization. Instead, it is the responsibility of the business application to assign permissions (roles, rights) for a user. To achieve this, a separate API interface can be used which provides additional functions and information about the user. Vendors can obtain specialized machine-to-machine (M2M) access tokens to access this interface, allowing for seamless integration with the business application. Further details can be found by following this link to the user information service.

-	All content of the business application must be served using https.
-	CORS must be allowed for portal.ekir.de for a successfull integration into the portal.
-	OpenID Connect Single Logout (SLO) must be implemented. More information here.
-	The user ID (uidNumber) serves as the unique identifier for each user. It is imperative to note that no other attribute, such as username or email, MUST BE used for identification as these attributes may change over time. A list of provided user attributes can be found here.
-	A BA must not allow any configuration of user information managed by the IdP, such as email or name. Additionally, the BA must not allow to set a user password to login independent from the IdP.

## configuration parameters
Additionally to the above, the vendor is required to provide the following information:
-	Redirect URI: In order to login users, we must know where to redirect the user after a successful login attempt.
-	Login URI: An endpoint in the BA that automatically initiates the login flow and redirects the user to the IdP. This is important for the user experience (UX), i.e. the user is immediately logged in after opening the application in the portal. In constrast to SAML, OpenID Connect does not specify a SSO-initiated login flow, so we require an endpoint to automatically start an BA-initiated login flow.
-	Scopes: This is always set to openid, unless the BA specifically requires alternative scopes to be send. However, no additional claims will be sent as a result of the login attempt. Additional user attributes may be acquired from a separate API interface, if permitted.
-	Claims: Same as scopes, we only provide basic information about users from the IdP. Specialized information can be acquired from a separate API interface. See user attributes for more details on the provided information.
-	Login Flow: OpenID specifies multiple authentication flows. Please provide the one that is required for the BA. More information about supported flows can be found here.
-	Front-Channel Logout URI: A URI to send logout requests against in case the user issues a logoff command. For more information, see here.
-	Audience (optional): Some clients might require a specific audience. If not, this will be the same as the client id.

## we can rename user properties
For both scopes and claims, we can rename the existing user properties to other names. For example, if the firstName attribute is required to be called givenName for the vendors specific BA, we can do so. Please provide a mapping if our existing mapping does not fit the BAs needs.

## steps to complete the integration
After the vendor has specified the required configuration parameters, the following data is provided to the vendor in order to complete the integration:
- ClientID and Secret: The clientId and clientSecret that allows the BA to authenticate users against the IdP.
- JWKS: An URI that specifies Authorizeation Endpoint, Token Endpoint, User Info Endpoint and several other properties about the integration. Use this if the BA allows to configure JWKS instead of the properties below.
- Authorization Endpoint (optional, if JWKS provided): The endpoint that the vendor application needs to redirect to in order to request authorization from the IdP.
- Token Endpoint (optional, if JWKS provided): This is the endpoint that the vendor's application will use to request an access token from the IdP.
- User Info Endpoint (optional, if JWKS provided): This is the endpoint that the vendor's application will use to request information about the user from the IdP.

## single logout (SLO)
Each application must guarantee that the SLO is working properly. If the SLO is not working properly, the SLO chain will be broken and the logout will not be completed. Currently "Front-Channel" is used. During a front-channel logout event initiated by the end-user, the IdP will send additional logout requests to all BAs that the user has logged in to. Each BA must guarantee that after a successfull SLO request, the user's current session is terminated: Cookies are marked as expired, sessions in backend are removed/terminated.

## openid connect flows
In OpenID Connect, authentication is performed using one of several flows, which determine the steps involved in the authentication process. The most commonly used OpenID Connect flows are:
1. Authorization Code Flow: This is the most commonly used flow in OpenID Connect. In this flow, the client sends the user to the authorization server to obtain an authorization code. The user authenticates with the authorization server and, if successful, the authorization server returns an authorization code to the client. The client then exchanges the authorization code for an ID token and access token.
2. Implicit Flow: The implicit flow is similar to the authorization code flow, but the ID token and access token are returned directly to the client, without the need for an additional token exchange step. This flow is used for clients that are unable to securely store a client secret, such as JavaScript or mobile applications.
3. Hybrid Flow: The hybrid flow is a combination of the authorization code flow and the implicit flow. It provides the security of the authorization code flow and the convenience of the implicit flow.
4. Client Credentials Flow: The client credentials flow is used for client authentication, rather than user authentication. In this flow, the client presents its credentials directly to the authorization server, which returns an access token. This flow is typically used for server-to-server communication.
The choice of which flow to use will depend on the requirements and type of the vendors BA. For example, if the BA is a JavaScript or mobile application, the implicit flow may be the best choice. On the other hand, if the BA is a server-side application, the authorization code flow is recommended.
For SPAs, we suggest the Implicit Flow. For all other applications, we require the authorization code flow.
We will never support the Client Credentials Flow.

## user information service
The identity provider uses JWT tokens to transmit a limited amount of user information to the connected BA. This is a unique sequential number, the current e-mail address of the respective user and the "username", which corresponds to the e-mail prefix before the "@" character. More information here.
A separate information service is provided for the exchange of additional information between the portal and the BA. The information service for providing additional user-related information is implemented as a REST API and secured via HTTPS. Each BA is given dedicated machine-to-machine (M2M) access, and the additional information that can be retrieved is defined for each BA separately.

## api specification 
API specification require a login.
Note that the existence of a user information service does not obligate vendors to use it. The decision to integrate with this service is made on a case-by-case basis, taking into consideration whether it makes sense to automate permission management for the relevant business application.

## user attributes
The following user attributes are provided to the BA by default:
•	lastName
•	firstName
•	email
•	username
•	uidNumber

! Don't confuse username and uidNumber. The uidNumber is the unique number of the user, whereas username is the current username (the part before the @ in the email address). As users change their name, these will change too, so MUST NOT be used as identifier for users. Only uidNumber MUST BE used as an identfier.
For OpenID Connect, the sub claim is set to the same value as uidNumber for all BAs.
