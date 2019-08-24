---
Title: Google Summer of Code 2019 at Linkerd
date: 2019-09-25
tags: ["linkerd", "smi", "service-mesh","smi-metrics" , "gsoc", "traffic"]
---

# My GSoC Journey with Linkerd

My GSoC Internship started with a different aim i.e to write a common GraphQL API to query linkerd metrics and my initial proposal was oriented towards it. I worked on the whole GraphQL spec along with the documentaion and relevant rfc's for it. By the end of community bonding period, we had a concrete proposal and I also got a scholarship to attend KubeCon + CloudNativeCon 2019 EU in Barcelona and get to meet my mentor and the whole Linkerd team. We had great fun discussing and clarifying my doubts regarding my internship and the linkerd project as a whole!

But then some companies in the service mesh space like Linkerd, Azure, Kinvolk, solo.io etc announced [Service Mesh Interface](https://smi-spec.io) which aims to provide a common interface for whole service mesh technologies so that common tools can be built on top of it. This seemed like a great platform to build the graphql API. Then I discussed with my mentor, linkerd team and the other SMI folks and decided that it would be best to spend my time on SMI-metrics which aims to solve a similar problem but on a bigger scale i.e for service meshes on whole.

The initial work was to make the smi-metrics API Server support more service meshes like Istio and Consul Connect. As this work can be a bit hectic and required learning a lot of things about Istio and Consul. My mentor suggested me to continue my contributions on Linkerd while also working on SMI-metrics. So, that I don't get struck on one thing. 

So, My GSoC work is divied into my contributions to SMI-metrics and also the Linkerd project.

# SMI-Metrics

Intially the SMI-Metrics, had a very hard dependency on prometheus for metrics as it was still a prototype. This can be a challenge for service mesh projects that don't depend on prometheus for queries or had a drastically different structure of time-series (i.e labels, sum of multiple metrics,etc ). So, I worked on adding a Mesh Interface and de-coupling the prometheus client. This made the repo more extensiable i.e now service meshes can just implement the interface and reply those metrics with or without prometheus.

During my GSoC period, I successfully got the `istio` support for SMI-Metrics working and also created the release automation for the project.

Istio support is available from Smi-Metrics version `v0.2.0`

## Demo of SMI-Metrics with Istio

First, make sure the kubernetes cluster has a working installation of Istio, by following this [doc](https://istio.io/docs/setup/kubernetes/)

Once this is done, let's run a demo app which has the Istio side-cars installed whose metrics we will query.

First, Let's create the `emojivoto` namespace.

```bash
kubectl apply -f https://gist.githubusercontent.com/Pothulapati/c8789307ec24fd73bb5ba5b892154ffd/raw/3386da4bec1472c11b1e0222c8143ca0d8c2a60b/namespace.yaml
```

Let's add the injection label, that makes Istio to inject sidecars automatically.

```bash
kubectl label namespace emojivoto istio-injection=enabled
```

Now, Let's deploy the Emojivoto services on to the same namespace.

```bash
kubectl apply -f https://gist.githubusercontent.com/Pothulapati/abed2e3f638fa0c16947b76191f89353/raw/8a891cf5d228a386bfc59cb7a1ce4c5af2f11db5/emojivoto.yaml
```

Now, If we check the pods in the emojivoto namespacc.

```bash
tarun@tarun-Inspiron-5559 ~/w/smi-metrics> kubectl get pods -n emojivoto
NAME                        READY   STATUS    RESTARTS   AGE
emoji-58c9579849-rp7nc      2/2     Running   0          28h
vote-bot-774764fd7f-blhzx   2/2     Running   0          28h
voting-66d5cdc46d-4fzqh     2/2     Running   0          28h
web-7f8455487f-rw9h5        2/2     Running   0          28h

```

We can see that all pods have two containers running i.e one application container and the Istio proxy (i.e envoy).

Regarding the emojivoto sample, `vote-bot` service keeps sending requests to `web`, which further talks to `emoji` and `voting`.

Now, Let's install SMI-Metrics Extension API Server and see the metrics of these application services.

The `Smi-metrics v0.2.0` helm package can be downloaded from the [link](https://github.com/deislabs/smi-metrics/releases/tag/v0.2.0)

Once it is unpacked. The package can be installed by running.

```bash
tarun@tarun-Inspiron-5559 ~/w/smi-metrics> helm install . --set adapter=istio --name dev
NAME:   dev
LAST DEPLOYED: Sat Aug 24 18:50:53 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME             DATA  AGE
dev-smi-metrics  1     4s

==> v1/Deployment
NAME             READY  UP-TO-DATE  AVAILABLE  AGE
dev-smi-metrics  1/1    1           1          4s

==> v1/Pod(related)
NAME                              READY  STATUS   RESTARTS  AGE
dev-smi-metrics-74df88c5fc-stxbk  1/1    Running  0         4s

==> v1/RoleBinding
NAME             AGE
dev-smi-metrics  4s

==> v1/Secret
NAME             TYPE               DATA  AGE
dev-smi-metrics  kubernetes.io/tls  2     4s

==> v1/Service
NAME             TYPE       CLUSTER-IP   EXTERNAL-IP  PORT(S)  AGE
dev-smi-metrics  ClusterIP  10.0.19.175  <none>       443/TCP  4s

==> v1/ServiceAccount
NAME             SECRETS  AGE
dev-smi-metrics  1        4s

==> v1alpha2/handler
NAME            AGE
smi-prometheus  4s

==> v1alpha2/instance
NAME                AGE
smirequestcount     4s
smirequestduration  4s

==> v1alpha2/rule
NAME     AGE
smiprom  3s

==> v1beta1/APIService
NAME                          AGE
v1alpha1.metrics.smi-spec.io  4s
```

As we can see, all the parts of the SMI installation have been installed successfully. Now, queries on the emojivoto resources can be done by running.

```bash
tarun@tarun-Inspiron-5559 ~/w/smi-metrics> kubectl get --raw /apis/metrics.smi-spec.io/v1alpha1/namespaces/emojivoto/deployments/web | jq
{
  "kind": "TrafficMetrics",
  "apiVersion": "metrics.smi-spec.io/v1alpha1",
  "metadata": {
    "name": "web",
    "namespace": "emojivoto",
    "selfLink": "/apis/metrics.smi-spec.io/v1alpha1/namespaces/emojivoto/deployments/web",
    "creationTimestamp": "2019-08-24T13:22:52Z"
  },
  "timestamp": "2019-08-24T13:22:52Z",
  "window": "30s",
  "resource": {
    "kind": "Deployment",
    "namespace": "emojivoto",
    "name": "web"
  },
  "edge": {
    "direction": "from",
    "resource": null
  },
  "metrics": [
    {
      "name": "p99_response_latency",
      "unit": "ms",
      "value": "9m"
    },
    {
      "name": "p90_response_latency",
      "unit": "ms",
      "value": "7m"
    },
    {
      "name": "p50_response_latency",
      "unit": "ms",
      "value": "3m"
    },
    {
      "name": "success_count",
      "value": "108"
    },
    {
      "name": "failure_count",
      "value": "4"
    }
  ]
}
```
These are the golden metrics of inbound requests that `web` deployment is recieving.

```bash
tarun@tarun-Inspiron-5559 ~/w/smi-metrics> kubectl get --raw /apis/metrics.smi-spec.io/v1alpha1/namespaces/emojivoto/deployments/web/edges | jq
{
  "kind": "TrafficMetricsList",
  "apiVersion": "metrics.smi-spec.io/v1alpha1",
  "metadata": {
    "selfLink": "/apis/metrics.smi-spec.io/v1alpha1/namespaces/emojivoto/deployments/web/edges"
  },
  "resource": {
    "kind": "Deployment",
    "namespace": "emojivoto",
    "name": "web"
  },
  "items": [
    {
      "kind": "TrafficMetrics",
      "apiVersion": "metrics.smi-spec.io/v1alpha1",
      "metadata": {
        "name": "web",
        "namespace": "emojivoto",
        "selfLink": "/apis/metrics.smi-spec.io/v1alpha1/namespaces/emojivoto/deployments/web/edges",
        "creationTimestamp": "2019-08-24T13:22:26Z"
      },
      "timestamp": "2019-08-24T13:22:26Z",
      "window": "30s",
      "resource": {
        "kind": "Deployment",
        "namespace": "emojivoto",
        "name": "web"
      },
      "edge": {
        "direction": "from",
        "resource": {
          "kind": "Deployment",
          "namespace": "emojivoto",
          "name": "vote-bot"
        }
      },
      "metrics": [
        {
          "name": "p99_response_latency",
          "unit": "ms",
          "value": "22m"
        },
        {
          "name": "p90_response_latency",
          "unit": "ms",
          "value": "9m"
        },
        {
          "name": "p50_response_latency",
          "unit": "ms",
          "value": "5m"
        },
        {
          "name": "success_count",
          "value": "116"
        },
        {
          "name": "failure_count",
          "value": "4"
        }
      ]
    },
    {
      "kind": "TrafficMetrics",
      "apiVersion": "metrics.smi-spec.io/v1alpha1",
      "metadata": {
        "name": "web",
        "namespace": "emojivoto",
        "selfLink": "/apis/metrics.smi-spec.io/v1alpha1/namespaces/emojivoto/deployments/web/edges",
        "creationTimestamp": "2019-08-24T13:22:26Z"
      },
      "timestamp": "2019-08-24T13:22:26Z",
      "window": "30s",
      "resource": {
        "kind": "Deployment",
        "namespace": "emojivoto",
        "name": "web"
      },
      "edge": {
        "direction": "to",
        "resource": {
          "kind": "Deployment",
          "namespace": "emojivoto",
          "name": "voting"
        }
      },
      "metrics": [
        {
          "name": "p99_response_latency",
          "unit": "ms",
          "value": "4m"
        },
        {
          "name": "p90_response_latency",
          "unit": "ms",
          "value": "4m"
        },
        {
          "name": "p50_response_latency",
          "unit": "ms",
          "value": "2m"
        },
        {
          "name": "success_count",
          "value": "30"
        },
        {
          "name": "failure_count",
          "value": "0"
        }
      ]
    },
    {
      "kind": "TrafficMetrics",
      "apiVersion": "metrics.smi-spec.io/v1alpha1",
      "metadata": {
        "name": "web",
        "namespace": "emojivoto",
        "selfLink": "/apis/metrics.smi-spec.io/v1alpha1/namespaces/emojivoto/deployments/web/edges",
        "creationTimestamp": "2019-08-24T13:22:26Z"
      },
      "timestamp": "2019-08-24T13:22:26Z",
      "window": "30s",
      "resource": {
        "kind": "Deployment",
        "namespace": "emojivoto",
        "name": "web"
      },
      "edge": {
        "direction": "to",
        "resource": {
          "kind": "Deployment",
          "namespace": "emojivoto",
          "name": "emoji"
        }
      },
      "metrics": [
        {
          "name": "p99_response_latency",
          "unit": "ms",
          "value": "4m"
        },
        {
          "name": "p90_response_latency",
          "unit": "ms",
          "value": "4m"
        },
        {
          "name": "p50_response_latency",
          "unit": "ms",
          "value": "2m"
        },
        {
          "name": "success_count",
          "value": "60"
        },
        {
          "name": "failure_count",
          "value": "0"
        }
      ]
    }
  ]
}
```
The above shows the `edges` from `web`, i.e inbound and outbound deployments and the respective golden metrics of the corresponding edges.

The same queries can be done for different kins of resources i.e pods, namespaces, daemonsets, etc as specified in the [smi-spec](https://github.com/deislabs/smi-spec/blob/master/traffic-metrics.md)

# Linkerd

Throughout the GSoC period, I also kept my contributions up to Linkerd solving issues ranging from small bug fixes to small scale features.

The total Linkerd and SMI-metric contributions can be found [here](https://github.com/Pothulapati/gsoc-meta-linkerd).


## Conclusion

I really had a great time working with the Linkerd team especially Thomas (my mentor). Their struggle for simplicity and ultralight-weight service mesh can be seen in each and every PR. The team definitely deserves a huge win and we are already seeing that happen ;). They really own the project and make sure all the incoming PR's follow standards. I got to learn a lot from them. Thomas is a great mentor and always gives me long sighted suggestions, insights and also fun tech rants(:p). I definitely can't ask for a better mentor. Thanks to Ivan, Alex, Alejandro, Andrew for unblocking me wherever possible and being friendly. <3

Special thanks to William, for the hospitaly at the Team dinner where I felt a bit awkward and alone :p . I will remember that for a long time.

Linkerd team is a dream team, and it would be really sad to not continue my contributions. I will be continuing my contributions to Linkerd and also will be taking up the work to make Tracing a first class component(like prometheus and grafana) in Linkerd. :) 