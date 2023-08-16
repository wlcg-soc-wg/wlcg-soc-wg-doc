# Sync with WLCG MISP instance

There are two options for using the MISP event data stored on the WLCG MISP instance; synching a local MISP instance and direct API access. Both methods start by gaining access to the WLCG instance. The advantage of using direct API access is that no further setup is required as you do not need to use a web instance. Advantages of using a web instance include a GUI interface for investigating events as well as the ability to give other users access to the data. For example, a web instance deployed at a given institution would allow access to be given to a local security team (following appropriate discussion of trusted sharing of information).

## WLCG MISP user accounts

Access to the WLCG MISP is gained via CERN SSO. To gain access, follow the following steps:

1. For a given site, one person needs to be selected as the contact point - this should be a real person, as opposed to a mailing list. The choice of who to use in this context is up to the institution, but should be a security contact that would normally be expected to work with threat intelligence.
2. The preferred route is via federated access using an identity provider in eduGAIN, with [SIRTFI](https://refeds.org/sirtfi) asserted. A list of the providers available can be found on the [login page](https://login.cern.ch) under *Sign in with your organization or institution account*.
    - If your institution is not listed, it is possible that they do not yet have SIRTFI asserted; you can use [this page](http://sirtfi.cern.ch) to find your provider and send them an email to request it.
    - One option, if you already have a grid certificate, would be to use the IGTF Certificate Proxy.
3. An alternative, if you have a CERN account, would be to use this instead; note that this should not typically be used in the first resort.
4. Once you have logged in successfully, make a note of your username and contact misp-admins AT cern.ch to discuss access.

If you have questions about access, please contact misp-admins AT cern.ch.

## Synching a local MISP instance

1. Log into in the WLCG MISP instance at <https://misp.cern.ch>. Click on your username at the top right of the screen and make a note of your auth key. **NOTE** this key will give access to all events stored on the MISP instance, so keep it safe. If you need to regenerate your key you can also do that from this page.
2. Follow the instructions given in the main MISP documentation for [adding a server](https://www.circl.lu/doc/misp/sharing/#adding-a-server). In particular, at the point of setting the remote organisation, use the following details:

    - Remote Sync Organisation Type: New external organisation
    - Remote Organisation's Name: WLCG
    - Remote Organisation's Uuid: 5a0ef475-e594-45c6-bdb0-7e25bcb85306

## Direct API access

To access the WLCG MISP instance directly via API, you can do so from the command line using your auth key as noted above. One example would be to use the [instructions](../integrations/misp_zeek.md) to pull event data from a MISP instance for use in Zeek. In this case, replace `your.misp.instance` with `misp.cern.ch` rather than your own instance.

