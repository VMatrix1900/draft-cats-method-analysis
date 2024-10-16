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
    ins: Z. Du
    fullname: Zongpeng Du
    organization: China Mobile
    email: duzongpeng@foxmail.com
 -
    ins: X. Yi
    fullname: Xinxin Yi
    organization: China Unicom
    email: yixx3@chinaunicom.cn
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

# Overhead of Metric Distribution
In the context of metric distribution for CATS, whether in a distributed or centralized architecture, one of the key considerations is the overhead involved in transferring metrics between the producers (C-SMA) and consumers (C-PS). This overhead can be defined by the following equation:

Metric Distribution Overhead = Number of Producers × Number of Consumers × Distribution Frequency × Metric Size

# Choice 1: Centralized versus Dencentralized

## Option 1: Centralized C-SMA + Centralized C-PS

The computing metrics can be collected internally with a hosting infrastructure by a centralized monitor of the hosting infrastructure. Various tools such as Prometheus can serve this purpose. The monitor can pass the metrics to a network controller, which behaves as a C-PS. Then, the network controller calculates the optimal path and distribute the paths to CATS ingress routers. When a service request arrives at the CATS ingress router, it just steers the request to the path. The network controller distributed the metric update to the C-PS using south-bound protocol.

## Option 2: Centralized C-SMA + Distributed C-PS

Similar to option 1, the network controller does not calculate the path. It just passes the computing metrics received from the cloud monitor to the C-PS embedded in a CATS ingress router. The C-PS at each CATS ingress router will proceed with path computation locally.


## Option 3: Distributed C-SMA + Centralized C-PS

The C-SMA can be deployed in a distributed way. For example, C-SMA running at each site collects the computing metrics of the service instances running in a site. Then, it reports the metrics to a network controller, which behaved as a C-PS. The network controller calculates the best path for a service and distribute the path to a CATS ingress router.

## Option 4: Distributed C-SMA + Distributed C-PS

Similar to option 3, each C-SMA collects the computing metrics of each site. Then it needs to distribute the metric to C-PS at each ingress router. It can do so directly or through a network controller.

## Comparaison

|  | Option 1 | Option 2 | Option 3 | Option 4 |
|------+-------+------+-------+----|
| Protocol | None | Southbound | Southbound | Southbound or Eastbound |
| CATS router requirement | Low | High | Low | High |
| Network controller requirement | High | Low | High | Low |
{: #ref-to-tab title="Comparison between different option"}

# Choice 2: Push versus Pull
There are two primary modes of the metric distribution: push and pull modes. The push mode operates on the principle of immediate dissemination of computing metrics as soon as they are refreshed. This approach boasts the advantage of timeliness, ensuring that the latest metrics are always available at the cost of frequent updates. The frequency of these updates directly correlates with the rate at which the computing metrics are refreshed.

Conversely, the pull mode adopts a more reactive strategy, where the latest computing metrics are fetched only upon receiving a specific request for them. This means that the distribution frequency of computing metrics hinges on the demand of such data, determined by the frequency of incoming service request from each ingress.

Irrespective of the chosen mode, various optimization techniques can be employed to regulate the frequency of metric distribution effectively. For instance, in the push mode, setting thresholds can mitigate the rate of updates by preventing the dispatch of new computing metrics unless there is a significant change in the metrics. This approach reduces unnecessary network traffic and computational overhead but at the potential cost of not always having the most up-to-date information.

In the pull mode, caching the returned computating metric for a predetermined duration offers a similar optimization. This method allows for the reuse of previously fetched data, delaying the need for subsequent requests until the cache expires. While this reduces the load, it introduces a delay in acquiring the latest computing metrics, possibly affecting decision-making processes that rely on the most current data.

Both push and pull models, despite their inherent differences, share a common challenge: striking a balance between the accuracy of the distributed computating metrics and the overhead associated with their distribution. Optimizing the distribution frequency through techniques such as threshold setting or caching can help mitigate these challenges. However, it's important to acknowledge that these optimizations may compromise the precision of scheduling tasks based on these metrics, as the very latest information may not always be available. This trade-off necessitates a careful consideration of the specific requirements and constraints of the computational environment to determine the most suitable approach.

# Choice 3: Aggregation of metric update messages

Another crucial aspect to consider in the distribution of computing metrics is the potential for aggregating updates. Specifically, in distributed C-SMA  scenarios, where an Egress point connects to multiple sites, it's feasible to consolidate updates from these sites into a single message. This aggregation strategy significantly reduces the number of individual update messages required, streamlining the process of disseminating computing metric.

Aggregation can be particularly beneficial in reducing network congestion and optimizing the efficiency of information distribution. By bundling updates, we not only minimize the frequency of messages but also the associated overheads, such as header information and protocol handling costs. This approach is not limited to distributed environments but is equally applicable in centralized C-SMA scenarios.

In centralized C-SMA scenarios, a controller responsible for managing computing metric updates to ingress nodes can employ a similar aggregation technique. By consolidating updates for multiple sites into a single message, the system can significantly decrease the overhead associated with update protocol messages.

While aggregating computing metrics offers substantial benefits in terms of reducing network traffic and optimizing message efficiency, it's important to address a specific challenge associated with this approach: the potential delay in message timeliness due to the waiting period required for aggregation. In scenarios where computing metrics from multiple nodes are consolidated into a single update message, the updates from individual nodes might not arrive simultaneously. This discrepancy can lead to situations where updates must wait for one another before they can be aggregated and sent out.

This waiting period introduces a delay in the dissemination of computing metrics, which, while beneficial for reducing the volume of update messages and network overhead, can inadvertently affect the system's responsiveness. The delay in updates might not align well with the dynamic needs of computing resource management, where timely information is crucial for making informed decisions about resource allocation and load balancing.

Therefore, while the aggregation of updates is an effective strategy for enhancing the efficiency of computing metrics distribution, it necessitates a careful consideration of its impact on the system's ability to respond to changes in computing needs promptly. Balancing the benefits of reduced message frequency and overhead with the potential delays introduced by aggregation requires a nuanced approach. This might involve implementing mechanisms to minimize waiting times, such as setting maximum wait times for aggregation or dynamically adjusting aggregation strategies based on the current load and the arrival patterns of updates.

# Security Considerations

TBD

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

The author would like to thank Xia Chen, Guofeng Qian, Haibo Wang for their help.
