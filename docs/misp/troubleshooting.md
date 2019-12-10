# Troubleshooting MISP

In case you are are having errors logs can be found in to different locations:
- Standard Apache logs, usual location `/var/log/httpd/`
- MISP specific logs, found under `/var/www/MISP/app/tmp/logs/`

Of the MISP specific logs the most interesting log files are `error.log` and `debug.log`. The logs starting with `resque-*` are logs for background workers.
