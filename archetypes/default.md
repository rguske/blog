---
author: "Robert Guske"
authorLink: "/about/"
lightgallery: true
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
featuredImage: /img/projectnautilus_cover.jpg
categories: ["cat1", "cat2"]
tags:
- tag2
- tag2
---
## Introduction





{{< image src="/img/posts//" caption="Figure I: " src-s="/img/posts//" >}}


{{< admonition info "" true >}}

{{< /admonition >}}



| | | |
|:---: |:---: |:---: | :---: |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |


{{< mermaid >}}
graph LR;
    subgraph HAProxy Appliance
    A[ Management ]
    B[ Workload ]
    C[ Frontend / VIPs ]
    end
    subgraph Traffic from
    G( Client / Service )
    end
    subgraph Load Balancer VIPs
    E[ Supervisor Cluster VMs ]
    F[ Guest Cluster VMs ]
    end
    A & B & C --- E
    B & C --- F
    G -.- C -.-> F & E
{{< /mermaid >}}