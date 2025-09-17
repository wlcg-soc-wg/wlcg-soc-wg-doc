# Automation using MISP REST API

This page is a collection of useful links to publish or create MISP events programmatically.

For the full MISP documentation, [check out the following link](https://www.circl.lu/doc/misp/).

## Automation

This tutorial will include a default MISP URL in the code snippets. Change it to your MISP URL.

The placeholder for the MISP URL is:
```
https://<misp_url>
```

You can find the official documentation from MISP for the automation feature at `https://<misp_url>/events/automation`.

## Creating an Automation API Key

Authentication of the automation is performed using a secure API key available in the MISP UI interface.

You can view and manage your API keys under your profile, respectively at `https://<misp_url>/users/view/me`.

Go to `Global Actions` > `My Profile` and expand `Auth keys` section to show the auth keys view. From there you can add new API keys, edit or delete old ones.

You can configure an IPs/subnets whitelist under `Allowed IPs`.
The accepted format is `<ip_address>/<mask>`.
To define a multiple IPs/subnets, write one per line in the accepted.

You can optionally set an expiration time for the key, which is recommended.

Finally, it is also possible to make this key read-only, meaning that it will not be possible to do any changes on this instance using this automation key.

## PyMISP

PyMISP is a convenient Python client library offering access to the MISP REST API.
A guide on how to use PyMISP can be found at [this link](https://www.circl.lu/doc/misp/pymisp/).

## OpenAPI Specs

MISP API allows you to query, create, modify data models, such as Events, Objects, Attributes. This is useful for interconnecting MISP with external tools. The official OpenAPI spec is available at `https://<misp_url>/api/openapi`.
