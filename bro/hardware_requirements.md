# Hardware requirements

## General system requirements

- System type: Preferable to have a physical box dedicated to it, a VM is unlikely to be able to cope with the amount of traffic. Still, if you don’t have a physical box available (we recommend a dual socket server), Bro can be deployed on a VM for testing.
- Number of cores: The recommendation is to have at least 12 physical cores. Rule of thumb is that one core is able to handle 800 Mbps - 1 Gbps of network traffic.
- Memory: At least 64 GB. Each Bro worker requires on average 6 GB of RAM.
- Disk space: As much as you can afford. The amount of disk space you have will determine the amount of log history you can store on disk. This is less important if you are shipping your logs to Elasticsearch of HDFS. Disk space doesn’t necessarily need to be local, you can also use a network filesystem to write to.
- Storage type: HDD or SSD. While SSD will obviously perform better, Bro is not I/O bound as writes are batched together.

## Network Interface Cards

- One NIC to be used by the operating system. This interface is used for the initial OS Installation and for connecting to the box. No special requirements on this one, can be a standard 1 Gbps NIC.
- One NIC dedicated for network traffic mirroring. The speed of this NIC needs to be equal or larger than the network throughput that you are looking to analyse. Also the interface of the NIC needs to be compatible with the interface that you have on the network device from where you’re doing the mirroring. A standard NIC will do here, no need to customised NIC with hardware based offloading.
- One management (IPMI) interface. This could be shared with the NIC used by the OS.

The above recommendations are assuming 10 Gbps of network traffic being monitored. In case the network throughput being monitored is less, the resources needed are obviously reduced. Resources are highly dependent on the number of connections, more than on the network throughput (many small connections are more expensive to monitor than a smaller number of large transfers).
