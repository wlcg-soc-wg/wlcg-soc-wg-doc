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

## Networks

Inside the `networks.cfg` file which should be located at `/opt/bro/etc/networks.cfg` define your local subnets. List of local networks should follow the CIDR notation, optionally followed by a descriptive tag. For example, "10.0.0.0/8" or "fe80::/64" are valid prefixes.

## Node configuration

Inside the `node.cfg` file which should be located at `/opt/bro/etc/node.cfg` define the Bro process configuration of the node. Please rename `<hostname>` with the hostname of your Bro node and `<interface_name>` with the name of the interface to which you have your traffic mirrored. You will need to adapt the number and IDs of the CPUs to match the layout of your system. It's advisable to start by using the physical cores only at the beginning and only if it turns out that just by using the physical cores Bro can't cope with the traffic and you have enough spare memory, you can move to using the hyperthreads.

```
[manager]
type=manager
host=<hostname>

[proxy]
type=proxy
host=<hostname>

[<hostname>-<interface_name>]
type=worker
host=<hostname>
interface=<interface_name>
lb_method=pf_ring
lb_procs=16
pin_cpus=0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15
```
