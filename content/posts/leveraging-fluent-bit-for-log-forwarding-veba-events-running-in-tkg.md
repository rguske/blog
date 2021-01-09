---
author: "Robert Guske"
authorLink: "/about/"
lightgallery: true
title: "Leveraging Fluent Bit for Log Forwarding VEBA Events Running in a Tanzu Kubernetes Grid Cluster"
date: 2021-01-07T23:30:23+01:00
draft: true
featuredImage: /img/veba_fluentbit_vrli_cover.jpg
categories: ["VEBA", "Tanzu", "vRealize Log Insight", "Kubernetes", "Logging"]
tags:
- FluentBit
- vRLI
- Kubernetes
- VEBA
- Tanzu
---
## Introduction
With the recent v0.5 release of the [VMware Event Broker Appliance (VEBA)](https://vmweventbroker.io) project, the ability of deploying the core component, which is the [VMware Event Router](https://github.com/vmware-samples/vcenter-event-broker-appliance/tree/development/vmware-event-router), via a [Helm](https://helm.sh) chart to an existing Kubernetes cluster was introduced. [William Lam](https://twitter.com/lamw) mentioned this and all the other great new features and enhancements in his corresponding [blog post](https://www.virtuallyghetto.com/2020/12/vcenter-event-broker-appliance-veba-v0-5-0.html). This earned very positive feedback from the community and opens even more doors in terms of flexibility as well as extensibility.

Speaking of which, Extensibility...

During the v0.5 Pre-Release team meeting, William came up with the idea to integrate VEBA with our log analysis solutions VMware [vRealize Log Insight](https://www.vmware.com/products/vrealize-log-insight.html) and/ or [vRealize Log Insight Cloud](https://cloud.vmware.com/log-insight-cloud) and he asked me if I would take over this. Of course there's no real reason to say no to his idea because it fits perfect to my post [Monitoring the VMware Event Broker Appliance with vRealize Operations Manager](https://rguske.github.io/post/monitoring-the-vmware-event-broker-appliance-with-vrealize-operations-manager/), in which I'm describing how I used [cAdvisor](https://github.com/google/cadvisor) to get resource usage and performance data from the various VEBA components and to monitor them.

## The Hummingbird (Attention: no technical relevance)
The power of symbols thrills me. Just think about the Kubernetes steering wheel for a second. If you feel home in this cloud native space and you see a steering wheel in real life nowadays, you will probably think of Kubernetes. This is the tremendous power of [symbols](https://en.wikipedia.org/wiki/Symbol). That is why there are fields of study for Symbology and movies with well known actors who are playing characters like Robert Langdon :wink:.

The Open Source projects [Fluentd](https://www.fluentd.org/) and [Fluent Bit](https://fluentbit.io/) immediately came into my mind as I started thinking about which solution I'm going to leverage for log processing and forwarding. My requirements on the solution are basically easy. It should be lightweight and fast. Comparing both mentioned projects brought me multiple times back to Fluent Bit (at least for my use case).

From the official Fluent Bit homepage:

> Fluent Bit is designed with performance in mind: high throughput with low CPU and Memory usage.

Now, coming back to the symbols/ logos of both projects, I think it's really a good match and pick from the maintainers to choose these birds. Fluentd for example has over **1000+** plugins available which can be connect to data sources and outputs! The logo of the project is a [Carrier Pigeon](https://en.wikipedia.org/wiki/Homing_pigeon) which has performance (e.g. in terms of distance covered) as one if it's major properties.

The [Hummingbird](https://en.wikipedia.org/wiki/Hummingbird) adorns the Fluent Bit logo. Those birds are are lightweight and PRETTY fast. They have wing-flapping rates between 12 and 80 beats per secons(!) depending on their actual size. This matches perfect the above quote.

## A short comparision

Doing my comparision of both projects, I found a lot of valuable articles out there. Just search for it on the web. Nevertheless, I wanted to put the official (short) comparison in here as well.

| | **Fluentd** | **Fluent Bit** |
|:---: |:---: |:---: |
| Scope | Containers / Servers | Embedded Linux / Containers / Servers |
| Language | C & Ruby | C |
| Memory | ~40MB | ~650KB |
| Performance | High Performance | High Performance |
| Dependencies | Built as a Ruby Gem, it requires a certain number of gems. | Zero dependencies, unless some special plugin requires them |
| Plugins | More than 1000 plugins available | Around 70 plugins available |
| License | Apache License v2.0 | Apache License v2.0 |

> By default, container engines such as Docker capture the standard output or error and leverage the JSON-file driver on each host to write messages to files. Docker maintains a separate log file for each container and stores it in the /var/log/containers directory of the Docker host. Annotation for each log entry consists of the following:

Log message
Message origin - stdout or stderr
Timestamp

Source: https://tanzu.vmware.com/content/blog/the-big-easy-visualizing-logging-data-by-integrating-fluentd-and-vrealize-log-insight-with-vmware-pks


https://github.com/pivotal-cf/fluent-bit-out-syslog#how-to-configure-fluent-bit-conf


## Comparison

https://docs.fluentbit.io/manual/about/fluentd-and-fluent-bit

https://github.com/fluent/fluent-bit/

## Docker Images/ Versions

https://docs.fluentbit.io/manual/installation/docker
Official: https://hub.docker.com/r/fluent/fluent-bit
Bitnami: https://hub.docker.com/r/bitnami/fluent-bit

## TKG

[Implementing Log Forwarding with Fluent Bit](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.2/vmware-tanzu-kubernetes-grid-12/GUID-extensions-logging-fluentbit.html#http)
Download the [VMware Tanzu Kubernetes Grid Extensions Manifest 1.2.0](https://my.vmware.com/en/web/vmware/downloads/info/slug/infrastructure_operations_management/vmware_tanzu_kubernetes_grid/1_x)


Create a namespace for the Fluent Bit service on the cluster.

kubectl apply -f logging/fluent-bit/namespace-role.yaml




## Service Section

https://docs.fluentbit.io/manual/administration/configuring-fluent-bit/configuration-file


## Input Section

If the cluster uses a CRI runtime, like containerd or CRI-O, change the Parser described in input-kubernetes.conf from docker to cri.

Input Plugins https://docs.fluentbit.io/manual/pipeline/inputs

Fluent Bit Input Plugins: https://github.com/fluent/fluent-bit/#input-plugins

## Output Section

https://github.com/pivotal-cf/fluent-bit-out-syslog/blob/master/config/parsers.conf

Syslog Output (Fluent Bit v.1.5.3) https://docs.fluentbit.io/manual/pipeline/outputs/syslog

The following properties are allowed: mode, syslog_format, syslog_maxsize, syslog_severity_key, syslog_facility_key, syslog_hostname_key, syslog_appname_key, syslog_procid_key, syslog_msgid_key, syslog_sd_key, and syslog_me

vRLI Port (Firewall Rec.) https://docs.vmware.com/en/vRealize-Log-Insight/8.2/com.vmware.log-insight.administration.doc/GUID-14DBC90A-379A-4316-9D76-4850E08437A8.html

Supported RFC's LogInsight: https://docs.vmware.com/en/vRealize-Log-Insight/8.2/log-insight-administration-guide.pdf



https://logz.io/blog/fluentd-vs-fluent-bit/ 




https://github.com/fluent/fluent-bit/issues/758 

    [INPUT]
        Name             tail
        Path             /var/log/containers/*.log
        Parser           docker_no_time
        Tag              kube.<namespace_name>.<pod_name>.<container_name>
        Tag_Regex        (?<pod_name>[a-z0-9]([-a-z0-9]*[a-z0-9])?(\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*)_(?<namespace_name>[^_]+)_(?<container_name>.+)-
        Refresh_Interval 5
        Mem_Buf_Limit    5MB
        Skip_Long_Lines  On


<output 1>
Match         kube.*myapp*.*
...
<output 2>
Match_Regex    ^kube\.(?~myapp)\.*$
...

#       DB                /var/log/flb_kube.db

Log Level: https://docs.fluentbit.io/manual/administration/configuring-fluent-bit/configuration-file


Fluentd LogInsight Plugin
fluent-plugin-vmware-loginsight
https://github.com/vmware/fluent-plugin-vmware-loginsight

CRI - https://rubular.com/r/tjUt3Awgg4

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: fluent-bit
  annotations:
    kapp.k14s.io/delete-strategy: orphan
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: fluent-bit
  namespace: fluent-bit
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: fluent-bit-read
rules:
- apiGroups:
  - ""
  resources:
  - namespaces
  - pods
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: fluent-bit-read
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: fluent-bit-read
subjects:
- kind: ServiceAccount
  name: fluent-bit
  namespace: fluent-bit
```



```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
  namespace: fluent-bit
  labels:
    k8s-app: fluent-bit
data:
  fluent-bit.conf: |
    [SERVICE]
        Flush         1
        Log_Level     info
        Daemon        off
        Parsers_File  parsers.conf
        HTTP_Server   On
        HTTP_Listen   0.0.0.0
        HTTP_Port     2020

    @INCLUDE input-kubernetes.conf
    @INCLUDE filter-kubernetes.conf
    @INCLUDE filter-record.conf
    @INCLUDE output-syslog.conf

  input-kubernetes.conf: |
    [INPUT]
        Name                tail
        Tag                 kube.*
        Path                /var/log/containers/*.log
        Parser              cri
        DB                  /var/log/flb_kube.db
        Mem_Buf_Limit       5MB
        Skip_Long_Lines     On
        Refresh_Interval    10

    [INPUT]
        Name                systemd
        Tag                 kube_systemd.*
        Path                /var/log/journal
        DB                  /var/log/flb_kube_systemd.db
        Systemd_Filter      _SYSTEMD_UNIT=kubelet.service
        Systemd_Filter      _SYSTEMD_UNIT=containerd.service
        Read_From_Tail      On
        Strip_Underscores   On

  filter-kubernetes.conf: |
    [FILTER]
        Name                kubernetes
        Match               kube.*
        Kube_URL            https://kubernetes.default.svc:443
        Kube_CA_File        /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        Kube_Token_File     /var/run/secrets/kubernetes.io/serviceaccount/token
        Kube_Tag_Prefix     kube.var.log.containers.
        Merge_Log           On
        Merge_Log_Key       log_processed
        K8S-Logging.Parser  On
        K8S-Logging.Exclude Off

    [FILTER]
        Name                modify
        Match               kube.*
        Copy                kubernetes k8s

    [FILTER]
        Name                nest
        Match               kube.*
        Operation           lift
        Nested_Under        kubernetes

  filter-record.conf: |
    [FILTER]
        Name                record_modifier
        Match               *
        Record tkg_cluster tkg-veba
        Record tkg_instance tkg-veba


    [FILTER]
        Name                nest
        Match               kube.*
        Operation           nest
        Wildcard            tkg_instance*
        Nest_Under          tkg

  output-syslog.conf: |
    [OUTPUT]
        Name                 syslog
        Match                *
        Host                 10.10.13.5
        Port                 514
        Mode                 tcp
        Syslog_Format        rfc5424
        Syslog_Hostname_key  tkg_cluster
        Syslog_Appname_key   pod_name
        Syslog_Procid_key    container_name
        Syslog_Message_key   message
        Syslog_SD_key        k8s
        Syslog_SD_key        labels
        Syslog_SD_key        annotations
        Syslog_SD_key        tkg

  parsers.conf: |
    [PARSER]
        Name                 json
        Format               json
        Time_Key             time
        Time_Format          %d/%b/%Y:%H:%M:%S %z

    [PARSER]
        Name                 docker
        Format               json
        Time_Key             time
        Time_Format          %Y-%m-%dT%H:%M:%S.%L
        Time_Keep            On

    [PARSER]
        Name                 docker-daemon
        Format               regex
        Regex                time="(?<time>[^ ]*)" level=(?<level>[^ ]*) msg="(?<msg>[^ ].*)"
        Time_Key             time
        Time_Format          %Y-%m-%dT%H:%M:%S.%L
        Time_Keep            On

    [PARSER]
        Name                 cri
        Format               regex
        Regex                ^(?<time>[^ ]+) (?<stream>stdout|stderr) (?<logtag>[^ ]*) (?<message>.*)$
        Time_Key             time
        Time_Format          %Y-%m-%dT%H:%M:%S.%L%z

    [PARSER]
        Name                 syslog-rfc5424
        Format               regex
        Regex                ^\<(?<pri>[0-9]{1,5})\>1 (?<time>[^ ]+) (?<host>[^ ]+) (?<ident>[^ ]+) (?<pid>[-0-9]+) (?<msgid>[^ ]+) (?<extradata>(\[(.*)\]|-)) (?<message>.+)$
        Time_Key             time
        Time_Format          %Y-%m-%dT%H:%M:%S.%L
        Time_Keep            On

    [PARSER]
        Name                 kube-custom
        Format               regex
        Regex                (?<tag>[^.]+)?\.?(?<pod_name>[a-z0-9](?:[-a-z0-9]*[a-z0-9])?(?:\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*)_(?<namespace_name>[^_]+)_(?<container_name>.+)-(?<docker_id>[a-z0-9]{64})\.log$
```

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-bit
  namespace: fluent-bit
  labels:
    k8s-app: fluent-bit-logging
    version: v1
    kubernetes.io/cluster-service: "true"
spec:
  updateStrategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        k8s-app: fluent-bit-logging
        version: v1
        kubernetes.io/cluster-service: "true"
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "2020"
        prometheus.io/path: /api/v1/metrics/prometheus
    spec:
      containers:
      - name: fluent-bit
        image: registry.tkg.vmware.run/fluent-bit:v1.5.3_vmware.1
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 2020
        readinessProbe:
          httpGet:
            path: /api/v1/metrics/prometheus
            port: 2020
        livenessProbe:
          httpGet:
            path: /
            port: 2020
        resources:
          requests:
            cpu: 5m
            memory: 10Mi
          limits:
            cpu: 50m
            memory: 60Mi
        volumeMounts:
        - name: var-log
          mountPath: /var/log
        - name: var-lib-docker-containers
          mountPath: /var/lib/docker/containers
          readOnly: true
        - name: fluent-bit-config
          mountPath: /fluent-bit/etc/
        - name: systemd-log
          mountPath: /run/log
      terminationGracePeriodSeconds: 10
      volumes:
      - name: var-log
        hostPath:
          path: /var/log
      - name: var-lib-docker-containers
        hostPath:
          path: /var/lib/docker/containers
      - name: systemd-log
        hostPath:
          path: /run/log
      - name: fluent-bit-config
        configMap:
          name: fluent-bit-config
      serviceAccountName: fluent-bit
      tolerations:
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      - operator: Exists
        effect: NoExecute
      - operator: Exists
        effect: NoSchedule
  selector:
    matchLabels:
      k8s-app: fluent-bit-logging
      version: v1
      kubernetes.io/cluster-service: "true"
```

```
Fluent Bit v1.5.3
* Copyright (C) 2019-2020 The Fluent Bit Authors
* Copyright (C) 2015-2018 Treasure Data
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io

[2021/01/07 20:18:40] [ info] [engine] started (pid=1)
[2021/01/07 20:18:40] [ info] [storage] version=1.0.5, initializing...
[2021/01/07 20:18:40] [ info] [storage] in-memory
[2021/01/07 20:18:40] [ info] [storage] normal synchronization mode, checksum disabled, max_chunks_up=128
[2021/01/07 20:18:40] [ info] [input:systemd:systemd.1] seek_cursor=s=50cd04862f114a64b3e0be3ff1904769;i=101... OK
[2021/01/07 20:18:40] [ info] [filter:kubernetes:kubernetes.0] https=1 host=kubernetes.default.svc port=443
[2021/01/07 20:18:40] [ info] [filter:kubernetes:kubernetes.0] local POD info OK
[2021/01/07 20:18:40] [ info] [filter:kubernetes:kubernetes.0] testing connectivity with API server...
[2021/01/07 20:18:41] [ info] [filter:kubernetes:kubernetes.0] API server connectivity OK
[2021/01/07 20:18:41] [ info] [output:syslog:syslog.0] setup done for 10.10.13.5:514
[2021/01/07 20:18:41] [ info] [http_server] listen iface=0.0.0.0 tcp_port=2020
[2021/01/07 20:18:41] [ info] [sp] stream processor started
```

image: registry.tkg.vmware.run/fluent-bit:v1.5.3_vmware.1



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