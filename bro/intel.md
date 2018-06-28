# Threat Intelligence

## Base setup

For a basic setup you can use something like this:

```
# Intel framework
@load frameworks/intel/seen
@load frameworks/intel/do_expire

redef Intel::item_expiration = 20min;

const feed_directory = "/data/bro/feeds";

redef Intel::read_files += {
# MISP feeds
        feed_directory + "/domain.txt",
        feed_directory + "/e-mail.txt",
        feed_directory + "/filehash.txt",
        feed_directory + "/filename.txt",
        feed_directory + "/ip.txt",
        feed_directory + "/url.txt",
};
```
We expire intel items after 20 minutes. Assuming that you have your MISP to Bro export configured to run every 15 minutes that will ensure that whatever IoCs get removed from MISP (because you've added a proposal to remove the for IDS flag for example) or expire (because the event has been published or re-publised for more than the amount of time defined in the MISP export script) will also get removed from Bro, with a 5 minute delay. We keep this 5 minutes buffer to account for possible delays in triggering the export. Of course you are more than welcome to adjust all these frequency of the export and intel item expiration to suit your needs.

Of course you are free to adapt the type of IoCs that you want to use for detection.

## Base setup

For a more advanced setup you can use something like this:

```
# Intel framework
@load frameworks/intel/seen
@load frameworks/intel/do_expire
@load notice-extensions

redef Intel::item_expiration = 20min;

const feed_directory = "/data/bro/feeds";

redef Intel::read_files += {
# MISP feeds
        feed_directory + "/domain.txt",
        feed_directory + "/e-mail.txt",
        feed_directory + "/filehash.txt",
        feed_directory + "/filename.txt",
        feed_directory + "/ip.txt",
        feed_directory + "/url.txt",
};

# Adapt the notice framework
redef Notice::emailed_types += {Intel::Notice};
redef Notice::type_suppression_intervals += {
        [Intel::Notice] = 12hr,
};
```

Under `/opt/bro/share/bro/site/` create a directory called `notice-extensions`. There place 2 files:

1. One file named `__load__.bro` and containing:

    ```
    @load ./do_notice.bro
    ```

2. Another file called `do_notice.bro` and containing:

    ```
    # Extends the orginal script to add an identifier to the notices.
    # Jan Grashoefer (jan.grashofer@cern.ch) and Liviu Valsan (liviu.valsan@cern.ch)
    # Original script is part of Bro.

    @load base/frameworks/intel
    @load base/frameworks/notice

    module Intel;

    export {
            redef enum Notice::Type += {
                    ## Intel::Notice is a notice that happens when an intelligence
                    ## indicator is denoted to be notice-worthy.
                    Intel::Notice
            };

            redef record Intel::MetaData += {
                    ## A boolean value to allow the data itself to represent
                    ## if the indicator that this metadata is attached to
                    ## is notice worthy.
                    do_notice: bool &default=F;

                    ## Restrictions on when notices are created to only create
                    ## them if the *do_notice* field is T and the notice was
                    ## seen in the indicated location.
                    if_in: Intel::Where &optional;
            };
    }

    event Intel::match(s: Seen, items: set[Item])
        {
        for ( item in items )
            {
            if ( item$meta$do_notice &&
                (! item$meta?$if_in || s$where == item$meta$if_in) )
                {
                local n = Notice::Info($note=Intel::Notice,
                $msg = fmt("Intel hit on %s at %s", s$indicator, s$where),
                $sub = cat("Indicator = ", s$indicator));

                if ( s?$conn )
                    {
                    n$conn = s$conn;

                    # Add identifier composed of indicator, originator's and responder's IP,
                    # without considering the direction of the flow.
                    local intel_id = s$indicator;
                    if( s$conn?$id )
                        {
                        if( s$conn$id$orig_h < s$conn$id$resp_h)
                            intel_id = cat(intel_id, s$conn$id$orig_h, s$conn$id$resp_h);
                        else
                            intel_id = cat(intel_id, s$conn$id$resp_h, s$conn$id$orig_h);
                        }
                    n$identifier = intel_id;
                    }

                # Add additional information to the generated mail
                local srv_str = "";
                for ( srv in s$conn$service )
                    srv_str = cat(srv_str, srv, " ");
                local mail_ext = vector(
                    fmt("Service: %s", srv_str),
                    fmt("Description: %s", item$meta$desc),
                    fmt("URL: %s", item$meta$url),
                    fmt("Intel source: %s", item$meta$source));
                n$email_body_sections = mail_ext;

                NOTICE(n);
                }
            }
        }
    ```

This improves the notifications sent by Bro to include context from MISP and also to suppress further alerts for the same IoC and the same device for the next 12 hours.