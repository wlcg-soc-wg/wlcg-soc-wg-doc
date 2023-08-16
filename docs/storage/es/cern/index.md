# Description of Elasticsearch setup at CERN for Computer Security and the SOC

The CERN Computer Security team is using two different clusters managed by the CERN IT Centralised Elasticsearch Service. The goal of this service is to consolidate and centralise Elasticsearch installations within the CERN IT department and the CERN experiments.

## Glossary
* Elasticsearch cluster: an Elasticsearch cluster is a stand-alone Elasticsearch installation. It can have one or more entry points.
* Entry point: A DNS alias pointing to a specific Elasticsearch cluster. 

## Security
The free version of [ReadonlyREST](http://readonlyrest.com/) is used to implement ACLs. There is no support at CERN for any commercial security plugins.

## Default installation
### Access methods
A default installation has the following features:
* Access to the Elasticsearch REST interface using basic authentication and kerberos
* Access to Kibana on port 443 using SSO and 2FA.

### Restrictions
A default installation has the following restrictions:
* The Java API is not supported due to security restrictions

## Available features

The following features are available by default (on port 443):
* egroup based full kibana access (create, update and save dashboards and visualisations)
* kibana read-only access (view dashboards, no rights to change or save them though)
* egroup based read / write Elasticsearch access
* egroup based read only Elasticsearch access

Access to Elasticsearch is granted both using basic authentication or via Kerberos / egroups. Basic authentication is still useful if you need to run clients which do not support Kerberos authentication, like Logstash.

# Entry Point configuration

Dedicated git repositories contain the configuration of the CERN Computer Security Elasticsearch clusters. The settings include: curator, kibana settings, template management, etc.

### Curator configuration

We are using curator as a tool to close and delete old indices. The official documentation of the tool can be found at  <https://www.elastic.co/guide/en/elasticsearch/client/curator/current/index.html>

The current version of curator, curator 4, can interact with elasticsearch 2 and elasticsearch 5. Please, note that the syntax has changed from the previous version of curator. 

The 'curator4.actions' yaml file contains all the actions that curator should do on the cluster. As an example, the next two actions will close any index that is older than 7 days, and delete them when they are older than 90 days 

```
actions: 
  1:
    action: close
    description: Close indices older than 30 days
    options:
      extra_settings:
      timeout_override:
      continue_if_exception: False
      disable_action: False
      ignore_empty_list: True
    filters:
      - filtertype: age
        direction: older
        unit: days
        unit_count: 30
        source: name
        timestring:  '%Y.%m.%d'
      - filtertype: pattern
        kind: prefix
        value: bro_
        exclude:
      - filtertype: closed
        exclude: True
  2:
    action: delete_indices
    description: Delete indices older than 90 days
    options:
      extra_settings:
      timeout_override:
      continue_if_exception: False
      disable_action: False
      ignore_empty_list: True
    filters:
      - filtertype: age
        direction: older
        unit: days
        unit_count: 90
        source: name
        timestring:  '%Y.%m.%d'
      - filtertype: pattern
        kind: prefix
        value: bro_
        exclude:
```

### Elasticsearch templates in git

Elasticsearch accepts the definition of templates that will be applied at the creation of new indices. This is very useful to ensure that the types are correctly specified, and to define aliases. The documentation is available at <https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-templates.html>

The CERN centralised Elasticsearch service offers the possibility to store those templates on a git repository, and apply them to the cluster. The user will send the requests to the git repo, and those templates will be synchronized every half an hour. 
