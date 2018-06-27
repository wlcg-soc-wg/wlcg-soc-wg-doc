# NIC tweaks

Normally, Bro's event engine will discard packets which don't have valid checksums. This can be a problem if one wants to analyze locally generated/captured traffic on a system that offloads checksumming to the network adapter. In that case, all transmitted/captured packets will have bad checksums because they haven't yet been calculated by the NIC, thus such packets will not undergo analysis defined in Bro policy scripts as they normally would. Bad checksums in traces may also be a result of some packet alteration tools.

A solution to that is to disable checksum offloading for your network adapter, but this is not always possible or desirable. Disable checksum offloading on the NIC using `ethtool --offload <int> rx off tx off` so the correct checksums are generated to begin with. Replacing `<int>` with the name of your interface.

Other tweaks to be made at the NIC level is to disable WOL and to increase the RX and TX ring parameters to higher values (for example 4096), both for RX and TX:
```
ethtool -s <int> wol d
ethtool -G <int> rx 4096 tx 4096
```
