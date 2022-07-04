---
author: "Robert Guske"
authorLink: "/about/"
lightgallery: true
title: "Transforming Harbor Webhook Notification Event Schema in Cloudevents using VMware Event Broker Appliance"
date: 2022-07-03T20:07:54+02:00
draft: true
featuredImage: /img/harbor-webhook-function-cover.png
description: ""
categories: ["VMware", "Cloud Native", "FaaS", "Serverless", "Event-Driven"]
tags:
- VEBA
- Knative
- Serverless
- Kubernetes
- Cloud Native
- Harbor
---
## Web🪝s, Web🪝s, Web🪝s

Many solutions today are supporting the configuration of a webhook endpoint, a remote webserver, to send information to it over `HTTP, HTTPS POST/PUT`. Information as such can vary but a popular use case for example is, to send out notifications in case of the occurrence of a specific event. The information sent are often provided in a `JSON` structured format. The open source container registry [Harbor](https://goharbor.io/) for instance, which I already tackled a couple of time on my blog ([#harbor](https://rguske.github.io/tags/harbor/)), supports the configuration of a webhook endpoint in order to send specific notification events via `http` or `https` (`POST`) as well. Those events are related to a created [Harbor project](https://goharbor.io/docs/latest/working-with-projects/create-projects/) specifically.

{{< admonition info "Harbor project" true >}}
A project in Harbor contains all repositories of an application. Images cannot be pushed to Harbor before a project is created. Role-Based Access Control (RBAC) is applied to projects, so that only users with the appropriate roles can perform certain operations.

*Source: [Harbor Documentation](https://goharbor.io/docs/latest/working-with-projects/create-projects/)*
{{< /admonition >}}

When now a certain project related interaction happened, like e.g. the `push` of a new container image (`PUSH_ARTIFACT`) or an image got pulled (`PULL_ARTIFACT`) or deleted (`DELETE_ARTIFACT`) to/from the project, the following payload will be send to the configured webhook endpoint:

```json
// Harbor specific JSON payload

{
  "type": "DELETE_ARTIFACT",
  "occur_at": 1655887039,
  "operator": "admin",
  "event_data": {
    "resources": [
      {
        "digest": "sha256:3b465cbcadf7d437fc70c3b6aa2c93603a7eef0a3f5f1e861d91f303e4aabdee",
        "tag": "v2.2.2",
        "resource_url": "harbor-app.jarvis.tanzu/veba-test/veba-image:v1.0"
      }
    ],
    "repository": {
      "name": "veba-image",
      "namespace": "veba-test",
      "repo_full_name": "veba-test/veba-image",
      "repo_type": "public"
    }
  }
}
```

*Source: [Harbor Documentation](https://goharbor.io/docs/latest/working-with-projects/project-configuration/configure-webhooks/)*

The information sent are very useful and are serving the receiver with details like e.g. `which`, `where`, `who`, `when` and more. Now, since those information are very, again useful and relevant, we want to process such in the most effective way. This is where I came up with the idea to use our [VMware Event Broker Appliance - VEBA](https://vmweventbroker.io) in order to ensure effectiveness.

## VEBA Generic Web🪝 and Web🪝 Functions

VEBA supports a generic webhook endpoint, configureable on `https://<VEBA-FQDN>/webhook`, since version [v0.7.0](https://vmweventbroker.io/evolution). This is absolutely great in terms of supporting even more event producers (like Harbor) which can create a custom payload. [William Lam](https://twitter.com/lamw) introduced a webhook function already in his article ["Custom webhook function to publish events into VMware Event Broker Appliance (VEBA)"](https://williamlam.com/2021/09/custom-webhook-function-to-publish-events-into-vmware-event-broker-appliance-veba.html). Now the challenge is, and William directly covered it at the very beginning of his post, that not every solution supports the [CloudEvents](https://cloudevents.io/) specification/schema. The idea behind CloudEvents is simple and powerful. If there's no default description/definition for event data, event handling logics have to be developed again and again!

I hope this will change in the future soon, because...

<center> {{< tweet user="embano1" id="1527298365528190976" >}} </center>

> Session from the tweet: **"Thinking Cloud Native, CloudEvents Future - Scott Nichols, Chainguard"** at KubeCon 2022 Europe :point_down: [Resources](#resources).

In my specific use case with Harbor, I checked the open project issue's on Github first to see if someone already raised the idea of making the webhook notification events CloudEvent conform and yes, <i class='fab fa-github fa-fw'></i> [#10146](https://github.com/goharbor/harbor/issues/10146) is bringing this idea up (give it a :thumbs_up:, :wink:).

However, in order to react on the sent events from Harbor, I briefly brainstormed with [Michael](https://twitter.com/embano1), got him hooked with the overall idea, and he quickly wrote a complete new function which transforms the incoming Harbor event into a CloudEvent :boom:.

<center> {{< tweet user="embano1" id="1541387083515846658" >}} </center>

## Low hanging Fruits

So, the overall idea is to send the non-CloudEvent payload to the new `kn-go-harbor-webhook` function, get it transformed into a CloudEvent payload and have the new [kn-ps-harbor-slack](https://github.com/vmware-samples/vcenter-event-broker-appliance/tree/development/examples/knative/powershell/kn-ps-harbor-slack) function triggered on the delivered and `subscribed` event (e.g. `PUSH_ARTIFACT`). This is how the flow will look like when you've deployed it:

{{< mermaid >}}
graph TD;
    subgraph Harbor Registry
    A
    end
    subgraph VEBA
    B
    C
    D
    end
    subgraph Slack
    E
    end
    A[Webhook Notification] -->|Harbor Event Payload| B[kn-go-harbor-webhook Function]
    B -->|CloudEvent Transformation - New Payload| C[Knative Magic]
    D[kn-ps-harbor-slack Function] --> C
    D --> E[Slack Channel Webhook]
{{< /mermaid >}}

{{< admonition info "Low Hanging Fruits" true >}}
:cherries: = [VEBA Function Examples](https://vmweventbroker.io/examples)

I simply took the existing [Slack-Function Example](https://github.com/vmware-samples/vcenter-event-broker-appliance/tree/master/examples/knative/powershell/kn-ps-slack), adjusted the necessary `fields` whithin the `handler.ps1` accordingly in order to ultimately pick the wanted data from the incoming event. Voilà, a Harbor specific function is ready to serve.
{{< /admonition >}}

## Deploying the new Harbor Functions

Let me guide you through the necessary steps in order to:

1. have a properly configured environment (DNS, VEBA, Harbor)
2. have the kn-go-harbor-webhook function running
3. have the kn-ps-harbor-slack function running

### DNS Wildcard Configuration for VEBA

The first required step is to configure your DNS server with a wildcard entry for VEBA. This is necessary because when you're going to deploy functions to VEBA, they'll ultimately run as a Pod on Kubernetes, or in Knative terminology a [Knative Service](https://knative.dev/docs/serving/services/) (`serving.knative.dev/service`). Those are providing (dynamic) endpoints for which we have to make sure that they are reachable over the wire. It is presented as follows: `https://[function-name].[function-namespace].[veba-fqdn]`. Since we are talking about Kubernetes Custom Resources, we can interact with such using `kubectl`. In order to check your functions, run `kubectl -n vmware-functions get ksvc`.

I'm using the Knative CLI for it because it'll add an additional column to the output which is `CONDITIONS`. It's always good to check those.

```shell
$ kn service list -n vmware-functions

NAME                   URL                                                                  LATEST                       AGE    CONDITIONS   READY   REASON
kn-go-harbor-webhook   http://kn-go-harbor-webhook.vmware-functions.veba-dev.jarvis.tanzu   kn-go-harbor-webhook-00001   9d     3 OK / 3     True
kn-pcli-nsx-tag-sync   http://kn-pcli-nsx-tag-sync.vmware-functions.veba-dev.jarvis.tanzu   kn-pcli-nsx-tag-sync-00001   13d    3 OK / 3     True
kn-pcli-tag            http://kn-pcli-tag.vmware-functions.veba-dev.jarvis.tanzu            kn-pcli-tag-00001            133d   3 OK / 3     True
kn-ps-harbor-slack     http://kn-ps-harbor-slack.vmware-functions.veba-dev.jarvis.tanzu     kn-ps-harbor-slack-00001     9d     3 OK / 3     True
```

Configuring a DNS wildcard entry on a Windows Server is pretty easy. Open the **DNS Manager**, right-click on your **Forward Lookup Zone** and select **New Host (A or AAA)**. For **Name** enter `*.<VEBA-Hostname>`. This should automatically create a FQDN like `*.veba-dev.jarvis.tanzu` in my example. Last is the **IP address** of your VEBA. That's it for the first step.

*Figure I* shows you how it looks like for me.

{{< image src="/img/posts/202207_harbor_webhhok_post/rguske-post-harbor-webhook-function-1.png" caption="Figure I: Windows Server Wildcard DNS Configuration" src-s="/img/posts/202207_harbor_webhhok_post/rguske-post-harbor-webhook-function-1.png" >}}

William explains the necessary configurations when using Unbound on Ubuntu - [LINK](https://williamlam.com/2021/09/custom-webhook-function-to-publish-events-into-vmware-event-broker-appliance-veba.html).

### Deploy the kn-go-harbor-webhook Function

It applies for most of the provided function examples, deploying a function to VEBA is simple by following the guidance provided for each specific function. Starting with Step 3, the creation of a Kubernetes Secret in order to enforce basic authentication on the HTTP endpoint.

1. kn-go-harbor-webhook secret

```shell
$ kubectl create secret generic webhook-auth \
--type=kubernetes.io/basic-auth \
--from-literal=username='webhookuser' \
--from-literal=password='replaceme' \
--namespace vmware-functions
```

Checking if the secret contains the passed values.

```shell
$ kubectl -n vmware-functions get secret webhook-auth -o yaml
apiVersion: v1
data:
  password: cmVwbGFjZW1l
  username: d2ViaG9va3VzZXI=
kind: Secret
metadata:
  creationTimestamp: "2022-07-04T14:05:49Z"
  name: webhook-auth
  namespace: vmware-functions
  resourceVersion: "291669370"
  uid: c0fc6ecc-a6cb-4532-813b-daea79dc0604
type: kubernetes.io/basic-auth
```

Let's quickly decrypt the value for `password`:

```shell
$ echo cmVwbGFjZW1l | base64 -d

$ replaceme
```

2. kn-go-harbor-webhook `SinkBinding`

A Knative `SinkBinding` makes the decoupling of an `event source` and an `event sink` possible. Therefore, it's possible to automatically inject the VEBA default broker into the function.

Create the `SinkBindung`:

```yaml
kubectl -n vmware-functions create -f - <<EOF
apiVersion: sources.knative.dev/v1
kind: SinkBinding
metadata:
  name: kn-go-harbor-webhook-binding
spec:
  subject:
    apiVersion: serving.knative.dev/v1
    kind: Service
    name: kn-go-harbor-webhook
  sink:
    ref:
      apiVersion: eventing.knative.dev/v1
      kind: Broker
      name: default
EOF
```

When checking the deployed object, you'll probably notice the info `REASON: SubjectMissing`. This is normal since it is waiting for the actual function deployment.

3. kn-go-harbor-webhook function

The final step now is the deployment of the function itself.

```yaml
kubectl -n vmware-functions create -f - <<EOF
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: kn-go-harbor-webhook
  labels:
    app: veba-ui
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/maxScale: "1"
        autoscaling.knative.dev/minScale: "1"
    spec:
      containers:
        - image: us.gcr.io/daisy-284300/veba/kn-go-harbor-webhook:1.0
          imagePullPolicy: IfNotPresent
          env:
            - name: ADDRESS
              value: "0.0.0.0"
            - name: WEBHOOK_PATH
              value: "/webhook" # default
            - name: DEBUG
              value: "true"
            - name: WEBHOOK_SECRET_PATH # remove this env to disable basic auth
              value: "/var/bindings/webhook"
          volumeMounts: # remove this when not using basic auth
            - name: webhook-auth
              mountPath: "/var/bindings/webhook"
              readOnly: true
      volumes: # remove this when not using basic auth
        - name: webhook-auth
          secret:
            secretName: webhook-auth
EOF
```

The Kubernetes Deployment or Knative Service is up and running is ready to receive and transform Harbor notification events.

```shell
$ k -n vmware-functions logs kn-go-harbor-webhook-00001-deployment-7d76fbb5fc-qclfv user-container

2022-07-04T14:32:14.599Z        INFO    harbor-webhook  kn-go-harbor-webhook/server.go:95       starting http server    {"commit": "5472048a", "tag": "1.0", "address": "0.0.0.0:8080", "path": "/webhook", "sink": "http://default-broker-ingress.vmware-functions.svc.cluster.local", "basic_auth": true}
```

### Configure the Webhook Feature in Harbor

In Harbor, open your project and go to **Webhooks**. Click on **+ NEW WEBHOOK** and configure it as you wish (e.g. in terms of events). Important is the **Endpoint URL** as well as the **Auth Header**.

For **Endpoint URL**, enter the complete URL ending with `/webhook` of the recently created Knative Service. For me it's: `https://kn-go-harbor-webhook.vmware-functions.veba-dev.jarvis.tanzu/webhook`

In **Auth Header** enter `Basic` followed by a whitespace and the base64-encoded value of the `<username>:<password>` string combination defined in the `webhook-auth` secret step. Use a Basic Auth Header Generator like e.g. [DebugBear](https://www.debugbear.com/basic-auth-header-generator).

{{< image src="/img/posts/202207_harbor_webhhok_post/rguske-post-harbor-webhook-function-2.png" caption="Figure II: Webhook Configuration in Harbor" src-s="/img/posts/202207_harbor_webhhok_post/rguske-post-harbor-webhook-function-2.png" >}}

The Webhook config should show up as shown in *Firgue III* below:

{{< image src="/img/posts/202207_harbor_webhhok_post/rguske-post-harbor-webhook-function-3.png" caption="Figure III: Configured Webhook in Harbor" src-s="/img/posts/202207_harbor_webhhok_post/rguske-post-harbor-webhook-function-3.png" >}}

### Verifying the Configuration

Verifying if everything works as expected can be done by e.g. `push`ing an image to your project (repo) or e.g. `delete`ing an existing one. For the sake of my example, I deleted an existing image (`artifact`) and checked the logs of the container called `user-conatiner`.

```shell
$ k -n vmware-functions logs kn-go-harbor-webhook-00001-deployment-7d76fbb5fc-qclfv user-container -f

2022-07-04T14:32:14.599Z        INFO    harbor-webhook  kn-go-harbor-webhook/server.go:95       starting http server    {"commit": "5472048a", "tag": "1.0", "address": "0.0.0.0:8080", "path": "/webhook", "sink": "http://default-broker-ingress.vmware-functions.svc.cluster.local", "basic_auth": true}
2022-07-04T15:01:02.406Z        DEBUG   harbor-webhook  kn-go-harbor-webhook/server.go:140      received request        {"commit": "5472048a", "tag": "1.0", "eventID": "f5c27ced-2b6b-45c3-a795-59efc33947b0", "request": "{\"type\":\"DELETE_ARTIFACT\",\"occur_at\":1656946862,\"operator\":\"admin\",\"event_data\":{\"resources\":[{\"digest\":\"sha256:d3890814cc5a7cfc02403435281cdf51adfb6b67e223934d9d6137a4ad364286\",\"tag\":\"1.21.6-debian-10-r117\",\"resource_url\":\"harbor.jarvis.tanzu/veba-webhook/bitnami-nginx:1.21.6-debian-10-r117\"}],\"repository\":{\"name\":\"bitnami-nginx\",\"namespace\":\"veba-webhook\",\"repo_full_name\":\"veba-webhook/bitnami-nginx\",\"repo_type\":\"public\"}}}"}
2022-07-04T15:01:02.412Z        DEBUG   harbor-webhook  kn-go-harbor-webhook/server.go:173      successfully sent cloudevent    {"commit": "5472048a", "tag": "1.0", "eventID": "f5c27ced-2b6b-45c3-a795-59efc33947b0", "event": "Context Attributes,\n  specversion: 1.0\n  type: com.vmware.harbor.delete_artifact.v0\n  source: /kn-go-harbor-webhook\n  subject: admin\n  id: f5c27ced-2b6b-45c3-a795-59efc33947b0\n  time: 2022-07-04T15:01:02Z\n  datacontenttype: application/json\nData,\n  {\n    \"type\": \"DELETE_ARTIFACT\",\n    \"occur_at\": 1656946862,\n    \"operator\": \"admin\",\n    \"event_data\": {\n      \"resources\": [\n        {\n          \"digest\": \"sha256:d3890814cc5a7cfc02403435281cdf51adfb6b67e223934d9d6137a4ad364286\",\n          \"tag\": \"1.21.6-debian-10-r117\",\n          \"resource_url\": \"harbor.jarvis.tanzu/veba-webhook/bitnami-nginx:1.21.6-debian-10-r117\"\n        }\n      ],\n      \"repository\": {\n        \"name\": \"bitnami-nginx\",\n        \"namespace\": \"veba-webhook\",\n        \"repo_full_name\": \"veba-webhook/bitnami-nginx\",\n        \"repo_type\": \"public\"\n      }\n    }\n  }\n"}
```

Even better looking is the output in VEBA's provided event viewer under `https://<veba-fqdn>/events`.

{{< image src="/img/posts/202207_harbor_webhhok_post/rguske-post-harbor-webhook-function-4.png" caption="Figure IV: Harbor CloudEvent in Sockeye" src-s="/img/posts/202207_harbor_webhhok_post/rguske-post-harbor-webhook-function-4.png" >}}

Awesome! :rocket:

### Deploy the kn-ps-harbor-slack Function

With the available CloudEvent payload, we can now configure other functions for being subscribed to incoming events and to ultimately get triggered by those. As aforementioned (low hanging fruits), I wanted to get notified by actions happened related to my Harbor project. The [kn-ps-harbor-slack](https://github.com/vmware-samples/vcenter-event-broker-appliance/tree/development/examples/knative/powershell/kn-ps-harbor-slack) function will send a beautiful looking message to your Slack channel webhook (there we go again... web:hook:s everywhere :smile:).

1. Add your Slack channel webhook URL to the [slack_secret.json](https://github.com/vmware-samples/vcenter-event-broker-appliance/blob/development/examples/knative/powershell/kn-ps-harbor-slack/slack_secret.json) file
2. Create the Kubernetes secret

```shell
$ kubectl -n vmware-functions create secret generic harbor-slack-secret --from-file=SLACK_SECRET=slack_secret.json

$ kubectl -n vmware-functions get secrets

NAME                    TYPE                                  DATA   AGE
harbor-slack-secret     Opaque                                1      1m
webhook-auth            kubernetes.io/basic-auth              2      9m
```

3. Deploy the `function.yaml`

By default, the function deployment will filter on the `com.vmware.harbor.push_artifact.v0` Harbor Event. If you wish to change this, update the `type` field within `function.yaml` to the desired event type. A list of supported notification events is available on the official Harbor documentation under [Configure Webhook Notifications](https://goharbor.io/docs/latest/working-with-projects/project-configuration/configure-webhooks/). As mentioned, use the VEBA event viewer endpoint (`https://<veba-fqdn>/events`) to display all incoming events.

```yaml
kubectl -n vmware-functions create -f - <<EOF
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: kn-ps-harbor-slack
  labels:
    app: veba-ui
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/maxScale: "1"
        autoscaling.knative.dev/minScale: "1"
    spec:
      containers:
        - image: us.gcr.io/daisy-284300/veba/kn-ps-harbor-slack:1.0
          envFrom:
            - secretRef:
                name: harbor-slack-secret
          env:
            - name: FUNCTION_DEBUG
              value: "false"
---
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: kn-ps-harbor-slack-trigger
  labels:
    app: veba-ui
spec:
  broker: default
  filter:
    attributes:
      type: com.vmware.harbor.push_artifact.v0
      subject: admin
  subscriber:
    ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: kn-ps-harbor-slack
EOF
```

When you now repeat one of the described steps from above, like e.g. `push` or `delete` an artifact, a new notification should show up in your Slack channel, like shown in *Firgure V*.

{{< image src="/img/posts/202207_harbor_webhhok_post/rguske-post-harbor-webhook-function-5.png" caption="Figure V: Slack Notification delivered by kn-ps-harbor-slack Function" src-s="/img/posts/202207_harbor_webhhok_post/rguske-post-harbor-webhook-function-5.png" >}}

## Resources

[^1]: [Thinking Cloud Native, CloudEvents Future - Scott Nichols, Chainguard](https://www.youtube.com/watch?v=Y6D0AY5aK-4)

{{< youtube Y6D0AY5aK-4 >}}