# Enabling OpenID Connect Authentication in MISP

## Introduction

### What is OIDC?

OIDC, or [OpenID Connect](https://auth0.com/docs/authenticate/protocols/openid-connect-protocol), is a protocol built on top of OAuth 2.0 that allows a client (e.g. MISP) to access a user's identity information from an OIDC provider, which fall under the more general category of identity providers (IdP). The OIDC provider assures they have the correct privileges to access a service. In effect, it allows a user to sign into MISP using another single-sign-on provider.

Examples of OIDC providers include Keycloak, Okta, and Indigo IAM, amongst others.

### Why use OIDC?

Using an OIDC provider for authentication means users' level of access can be managed in the IdP, so there is no need to manage users within MISP - one can simply assign the `misp-access` role to a user in the IdP, and they will be able to log into MISP as a standard user. Similarly one could assign a user the `misp-admin-access` role and they would be able to log in as an admin. This makes it easy to do proper role-based access control where all roles are managed in a central place.

Of course it is important to make sure the IdP itself is properly secured and has locked-down permissions as to who can edit and assign roles.

When users navigate to a MISP instance that uses an OIDC provider, they will be met with the OIDC provider's login page. Once they authenticate with the IdP, they will be redirected to MISP, logged in with the level of access according to their role.

## Configuring MISP to use OIDC

This guide uses [Keycloak](https://www.keycloak.org/) as the OIDC provider. Some testing was also carried out with [Indigo IAM](https://github.com/indigo-iam/iam), but more work needs to be done to get it communicating properly with MISP. It also assumes the use of the [JISC CTI MISP deployment](https://github.com/JiscCTI/misp-docker).

### Setting up Keycloak

Notes: You must be a realm admin in Keycloak in order to follow this guide. This guide is based on the Keycloak [admin console released in version 19.0.0](https://www.keycloak.org/docs/latest/release_notes/index.html#new-admin-console-graduation).

In your Keycloak realm, create a new client with the following configuration (replace `{MISP_BASEURL}` with the base URL of your MISP instance):

 * Client ID: `misp`, though this can be different.
 * Under Access settings:
   * Root URL and Home URL: `{MISP_BASEURL}` 
   * Valid redirect URIs: `{MISP_BASEURL}/users/login`
 * Under Capability config, ensure 'Client authentication' and 'Authorization' are on, and that 'Standard flow' and 'Direct access grants' are checked under 'Authentication flow'
 * In the 'Credentials' tab:
   * Set 'Client Authenticator' to 'Signed Jwt with Client Secret', as this is a more secure method of authentication that MISP supports.
   * Copy the client secret generated on this page.

Next, navigate to 'Realm Roles' and create, at the very least, a role called `misp-access`. Navigate to the 'Users' page, click on a user and assign this role under the 'Role mapping' tab.

### Configuring MISP

The settings that control OIDC can be placed in `app/config.php`. If using the JISC CTI deployment, this can be edited in the persistent data directory, at `./persistent/<misp-instance>/data/config/config.php`. Editing this file directly should update the MISP configuration live. If you encounter problems, try restarting the `misp-web` container.

Add the following to `config.php`, replacing the values in brackets with your own:

```
  'Security' =>
  array (
...
    'auth' => array('OidcAuth.Oidc'),
...
),

...

  'OidcAuth' =>
  array (
    'provider_url' => 'https://{KEYCLOAK_URL}/auth/realms/{REALM_NAME}/.well-known/openid-configuration',
    'client_id' => 'misp',
    'client_secret' => '{KEYCLOAK_CLIENT_SECRET}',
    'authentication_method' => 'client_secret_jwt',
    'code_challenge_method' => 'S256',
    'redirect_uri' => '{MISP_BASEURL}/users/login',
    'role_mapper' => [
        'misp-admin-access' => 1, // Admin
        'misp-org-admin-access' => 2, // Org Admin
        'misp-sync-access' => 5, // Sync user
        'misp-publisher-access' => 4, // Publisher
        'misp-api-access' => 'User with API access',
        'misp-access' => 3, // User
    ],
    'default_org' => 'Testing org',
  ),
```

Note that the role mapper can be edited to match roles created in Keycloak. Each role corresponds to a different level of access in MISP, all configured in this array. MISP will automatically give a user with multiple roles the highest privileges it can - i.e. `misp-admin-access` takes priority over `misp-access`.

Navigating to the MISP base URL should now present the user with a login page from the SSO. Provided they have a valid role, they will be redirected to MISP upon authenticating with Keycloak.

### Known limitations

For some reason, the MISP logout button seems to keep the user logged in. In order to log out, a user will need to delete cookies for the current page (Hint: clicking the padlock next to the URL in most browser will allow you to quickly manage cookies for that site).
