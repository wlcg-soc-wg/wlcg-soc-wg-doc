# Mapping of fields

By default, elasticsearch will guess the fields of the documents based on the first document that arrives on a particular index. If the type of the fields is known beforehand, it is often better to upload an index template, which will create the mappings at index creation. See
* <https://www.elastic.co/guide/en/elasticsearch/guide/current/index-templates.html> about how to write template files
* <https://esdocs.web.cern.ch/esdocs/tools/elastic_templates.html> about we support them
