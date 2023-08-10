---

title: "Design analysis of methods for distributing the computing metric"
abbrev: "Analysis of metric distribution"
category: info

docname: draft-shi-cats-analysis-of-metric-distribution-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Routing"
workgroup: "Computing-Aware Traffic Steering"
keyword:

author:
 -
    ins: H. Shi
    fullname: Hang Shi
    organization: Huawei Technologies
    email: shihang9@huawei.com
    country: China
 -
    ins: T. Yang
    fullname: Tianle Yang
    organization: China Broadcast Mobile Network Company
    email: yangtianle@10099.com.cn
    country: China

normative:

informative:


--- abstract

This document analyses different methods for distributing the computing metrics from service instances to the ingress router.


--- middle

# Introduction

Many modern computing services are deployed in a distributed way. Multiple service instances deployed in multiple sites provide equivalent function to the end user. As described in {{?I-D.yao-cats-ps-usecases}}, traffic steering that takes computing resource metrics into account would improve the quality of service. Such computing metrics are defined in {{?I-D.du-cats-computing-modeling-description}}. This document analysis different methods for  distributing these metrics.



# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document uses terms defined in {{?I-D.ldbc-cats-framework}}. We list them below for clarification.

- Computing-Aware Traffic Steering (CATS): An architecture that takes into account the dynamic nature of computing resources and network state to steer service traffic to a service instance. This dynamicity is expressed by means of relevant metrics.

- CATS Service Metric Agent (C-SMA):Responsible for collecting service capabilities and status, and reporting them to a CATS Path Selector (C-PS).

- CATS Path Selector (C-PS): An entity that determines the path toward the appropriate service location and service instances to meet a service demand given the service status and network status information.

# Requirement of distributing computing metric
The CATS functional components are defined in {{?I-D.ldbc-cats-framework}}(see {{fig-cats-fw}}, the figure is replicated here for better understanding). C-SMA is responsible for collecting the computing metrics of the service instance and distributing the metrics to the C-PSes. A C-PS then selects a path based on the computing metrics and network metrics.

~~~
      +-----+              +------+            +------+
    +------+|            +------+ |          +------+ |
    |client|+            |client|-+          |client|-+
    +------+             +------+            +------+
        |                    |                   |
        |   +-------------+  |            +-------------+
        +---|    C-TC     |---+     +------|    C-TC     |
            |-------------|         |      |-------------|
            |     | C-PS  |     +------+   |CATS-Router 4|
    ........|     +-------|.....| C-PS |...|             |...
    :       |CATS-Router 2|     |      |   |             |  .
    :       +-------------+     +------+   +-------------+  :
    :                                                       :
    :                                            +-------+  :
    :                         Underlay           | C-NMA |  :
    :                      Infrastructure        +-------+  :
    :                                                       :
    :                                                       :
    :   +-------------+                 +-------------+     :
    :   |CATS-Router 1|  +-------+      |CATS-Router 3|     :
    :...|             |..| C-SMA |.... .|             |.....:
        +-------------+  +-------+      +-------------+
                |         |             |    C-SMA    |
                |         |             +-------------+
                |         |                     |
                |         |                     |
              +------------+               +------------+
            +------------+ |             +------------+ |
            |  service   | |             |  service   | |
            |  instance  |-+             |  instance  |-+
            +------------+               +------------+

              edge site 1                   edge site 2
~~~
{: #fig-cats-fw title="CATS Functional Components"}



# Option 1: Centralized C-SMA + Centralized C-PS

The computing metrics can be collected internally with a hosting infrastructure by a centralized monitor of the hosting infrastructure. Various tools such as Prometheus for this purpose. The monitor can pass the metrics to a network controller, which behaves as a C-PS. Then, the network controller calculates the optimal path and distribute the paths to CATS ingress routers. When a service request arrives at the CATS ingress router, it just steers the request to the path. The network controller distributed the metric update to the C-PS using south-bound protocol.

# Option 2: Centralized C-SMA + Distributed C-PS

Similar to option 1, the network controller does not calculate the path. It just passes the computing metrics received from the cloud monitor to the C-PS embedded in a CATS ingress router. The C-PS at each CATS ingress router will proceed with path computation locally.


# Option 3: Distributed C-SMA + Centralized C-PS

The C-SMA can be deployed in a distributed way. For example, C-SMA running at each site collects the computing metrics of the service instances running in a site. Then, it reports the metrics to a network controller, which behaved as a C-PS. The network controller calculates the best path for a service and distribute the path to a CATS ingress router.

# Option 4: Distributed C-SMA + Distributed C-PS

Similar to option 3, each C-SMA collects the computing metrics of each site. Then it needs to distribute the metric to C-PS at each ingress router. It can do so directly or through a network controller.

# Comparaison

|  | Option 1 | Option 2 | Option 3 | Option 4 |
|------+-------+------+-------+----|
| Protocol | None | Southbound | Southbound | Southbound or Eastbound |
| CATS router requirement | Low | High | Low | High |
| Network controller requirement | High | Low | High | Low |
{: #ref-to-tab title="Comparison between different option"}

# Security Considerations

TBD

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

The author would like to thank Xia Chen, Guofeng Qian, Haibo Wang for their help.
