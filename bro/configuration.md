# Configuration

## Broctl

Inside the `broctl.cfg` file which should be located at `/opt/bro/etc/broctl.cfg` set the following configuration options:
- `MailTo`: Destination address for non-alarm mails. This is the email address that will receive the detection alerts.
- `MailAlarmsTo`: Destination address for alarm summary mails. Default is to use the same address as MailTo. This overrides the Bro script variable Notice::mail_dest_pretty_printed.
- `MailSubjectPrefix`: General Subject prefix for mails. This can be used for filtering Bro emails.
- `LogDir`: Directory for archived log files. You may want to have these on a separate volume with a high storage capacity.
- `LogExpireInterval`: Time interval that archived log files are kept (a value of 0 means log files never expire). The time interval is expressed as an integer followed by one of the following time units: day, hr, min.
- `SpoolDir`: Location of the spool directory where files and data that are currently being written are stored. It's recommended to have this located on the same type of high capacity volume as the logs.

Please note that `broctl deploy` will need to be ran everytime there is a change to the above file.
