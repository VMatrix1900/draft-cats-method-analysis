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
area: Routing Area
workgroup: CATS
keyword:

author:
 -
    ins: H. Shi
    fullname: Hang Shi
    organization: Huawei Technologies
    email: shihang9@huawei.com
    country: China

normative:

informative:


--- abstract

This document analyses different methods for distributing the computing metric from the service instance to the ingress router.


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
The function components of the CATS is defined in {{?I-D.ldbc-cats-framework}}(see {{fig-cats-fw}}, the figure is replicated here for better understanding). C-SMA is responsible for collecting the computing metric of the service instance and distributing the metrics to the C-PS. C-PS then select the path based on the computing metric and network metric.

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

The computing metric can be collected internally by a centralized cloud monitor of the cloud platform. Various tools such as Prometheus for this purpose. The cloud monitor can pass the metric to the network controller. Then the network controller calculate the optimal path and distribute the path to the CATS ingress router. The ingress router just need to steer the flow to the path. The network controller distributed the metric update to the C-PS using south-bound protocol.

# Option 2: Centralized C-SMA + Distributed C-PS

Similar to option 1, the network controller does not calculate the path. It just passes the computing metric received from the cloud monitor to the C-PS in the CATS ingress router. The C-PS at each CATS ingress router will calculate the best path independently.


# Option 3: Distributed C-SMA + Centralized C-PS

The C-SMA can be deployed in a distributed way. For example, C-SMA running at each site collects the computing metric of the service instance running in the site. Then it reports the metric to the network controller. The network controller calculates the best path and distribute the path to the CATS ingress router.

# Option 4: Distributed C-SMA + Distributed C-PS

Similar to option 3, each C-SMA collect the computing metric of each site. Then it needs to distribute the metric to C-PS at each ingress router. It can do so directly or through a network controller.

# Comparison

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
