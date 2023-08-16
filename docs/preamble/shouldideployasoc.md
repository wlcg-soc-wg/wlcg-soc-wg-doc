# Should I deploy a SOC?

A properly configured Security Operations Centre (SOC) is a powerful tool for monitoring and alerting threats to your environment.  However, an improperly deployed or configured SOC can be a tremendous drain on precious resources with very little reward in terms of impact and outcomes.  Therefore, before building a SOC, it is essential to examine the existing security environment to ensure that building a SOC will augment the existing capabilities of the security team.

The reference model for the WLCG SOC architecture allows for several starting places for organisations of different maturity levels to consider implementing to allow for participation in the sharing of threat intelligence between research computing and education facilities.

At all levels a site considering SOC deployment should have, at a minimum:

- Partial or full staffing for dedicated cybersecurity responsibilities
- Internal documentation governing incident response and staff assigned to the task
- A cybersecurity policy defining roles, responsibilities, and data classifications for the organisation
- Basic inventory of critical hardware and software assets

Sites with the above programmatic requirements in place which have the effort and resources available to improve their existing security operations may consider the introduction of one of the SOC deployment models described in the reference architecture.  Organisations lacking any of the above would most likely be better served investing time and effort adopting a cybersecurity framework, core policies, and adopting a full set of controls appropriate for their environment.  

An appropriately scoped and sized SOC deployment should augment the existing activities of a security program through automation and monitoring, allowing for better detection time, integration of threat intelligence from other sources, and more effective and timely incident response. When considering deploying a SOC, there are two paths that an organisation may consider in the WLCG SOC model, a low cost, low complexity approach: the pDNSSOC model, and a more complex, essential SOC model which may be built upon over time to further enrich and enhance data monitoring and threat intel sharing, the Essential SOC.

Least Effort (pDNSSOC) 

- Partially dedicated security staff are nee- ded to process alerts and respond to incidents generated by pDNSSOC
- An incident response policy and/or playbook is needed
- Some sort of (loosely) defined security program to define roles, enable data classification, and authorise staff to make security decisions, influence IT operations
- inventory?

Essential SOC 

- All of the above *AND*
- Dedicated and committed financial resources for the building and sustainment of an operational SOC
- Sufficient staff expertise for effective hardware and software deployment and configuration
- Sufficient staff expertise to apply configuration and alerting baselines created by WLCG SOC and tailor them to fit the organisations needs.
- Sufficient available time to triage/threat hunt on generated alerts and feedback alert tuning
- Organisational policies which allow for the sharing and consumption of threat intel data from the WLCG or other MISP instances

Maturing SOC

- All of the above *AND*
- A continuous improvement model to enhance the existing SOC capabilities with new data sources and other processes as necessary
- Provide threat intelligence to the wider community as able

To help decide which path is most suitable for an organisation should consider what their goals in SOC participation are and what level of long term resources they are able to devote to the project.  The pDNSSOC model is a low cost and very rapid way to increase the effectiveness of their security operation but is limited in scope and is not designed to be built upon over time.  The Essential SOC aims to be the entry point for a multi-component SOC that allows for monitoring and alerting on network packet data which can integrate multiple sources of threat intelligence and data enrichment.  It requires substantially more investment in both time, resources, and technical expertise, the upside of which is that it is highly configurable and adaptable to many sizes and types of institutions and allows for a variety of outcomes of alerting, threat intelligence sharing, and scalability.