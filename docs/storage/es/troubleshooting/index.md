# Troubleshooting

## Kibana crashes when using the Google Chrome web browser

Please check if you have any plugins installed which may interfere with Kibana. Issues have been reported when using 'Metamask Chrome'.

## Kibana freezes or takes a very long time to return

Please check if the data ranges are as you expect them to be. For example, if you try to create a histogram with small bin size, and your data contains some unexpected values which are far outside the expected range, then way more bins will have to be rendered than you were expecting which can result in freezing of the browser. This can be avoided by adding additional filters on the expected range of the returned data.

