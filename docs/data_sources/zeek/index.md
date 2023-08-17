# Introduction to Zeek NSM
[Zeek](https://docs.zeek.org/en/master/about.html), formerly Bro, is an open source, passive network traffic analysis tool.  Zeek is the most commonly used Network Security Monitoring (NSM) tool in the security community.  Zeek also supports a wide range of traffic analysis tasks beyond the security domain, including performance measurement and troubleshooting.

Zeek may be deployed as a stand alone tool in a SOC environment and is available under an open source license free of cost.  Zeek is also commonly packaged in security appliances and software suites, under both commercial and open use licensing, some common packaged deployments and appliance which utlise Zeek in a security environment are [Corelight](https://corelight.com/) hardware and software appliances  and [Security Onion](https://securityonionsolutions.com/).

In the WLCG SOC Reference model Zeek fills the role of the primary Data Source for network traffic capture.

## Zeek vs Suricata vs PCAP

Useful paragraph from the Zeek documentation about its suitability for NSM vs signature matching:

```
In brief, Zeek is optimized for interpreting network traffic and generating logs based on that traffic. It is not optimized for byte matching, and users seeking signature detection approaches would be better served by trying intrusion detection systems such as Suricata. Zeek is also not a protocol analyzer in the sense of Wireshark, seeking to depict every element of network traffic at the frame level, or a system for storing traffic in packet capture (PCAP) form. Rather, Zeek sits at the “happy medium” representing compact yet high fidelity network logs, generating better understanding of network traffic and usage.
```

- [Hardware requirements](hardware_requirements.md)
- [Recommended operating system](recommended_os.md)
- [Software repository](software_repository.md)
- [Capture Network Interface Card tweaks](nic_tweaks.md)
- [Deploying PF_RING](deployment_pfring.md)
- [Deploying Zeek](deployment_zeek.md)
- [Configuration](configuration.md)
- [Alerting](intel.md)
- [JSON Logs](json.md)
