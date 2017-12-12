# Exporting IoCs from MISP to Bro

Bro comes with its own intelligence framework and MISP is able to do an export into the native format used by the Bro intelligence framework. The following script can be used for that:

```
#!/bin/bash

INTEL_DIR="/path/to/intel_feeds"
INTEL_DIR_TMP="/path/to/intel_feeds.tmp"
FEED_URL="https://your.misp.instance/attributes/bro/download/"
AUTH_KEY="Authorization: <MISP_API_KEY>"
JSON="application/json"
LAST="30d"
EXCLUSIONS="\"eventId\":[\"!999998\"],"
WHITELIST="\"eventId\":[\"999999\"],"

# Prepare
if [ ! -d $INTEL_DIR ]; then
    mkdir -p $INTEL_DIR
fi

if [ ! -d $INTEL_DIR_TMP ]; then
    mkdir -p $INTEL_DIR_TMP
fi

# Fetch feeds
for type in ip domain url email filename filehash certhash software; do
    curl -s --header "$AUTH_KEY" --header "Accept: $JSON" --header "Content-type: $JSON" -X POST --data "{\"request\": {${EXCLUSIONS} \"type\": \"${type}\", \"last\": \"${LAST}\"}}" $FEED_URL > "${INTEL_DIR_TMP}/${type}.txt"
done

mv -f $INTEL_DIR_TMP/* $INTEL_DIR/
```
