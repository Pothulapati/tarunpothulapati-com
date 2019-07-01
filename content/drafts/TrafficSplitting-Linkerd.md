
---
Title: Traffic Splitting in Linkerd using SMI
date: 2017-07-26
tags: ["github-pages", "wyam", "static-site"]
---

# Traffic Splitting in Linkerd

Linkerd is a service mesh built for kubernetes and provides

Linkerd 2.4 comes with Traffic Splitting functionality, which allows users to split traffic destined to a particular service to multiple services. Traffic is split based on the weight given for a particular service (in the case of linkerd), percentages can also be used here. Traffic Splitting is useful for performing Canary Deployments, etc. 

## How is it done in Linkerd?

As we know, A service mesh is divided into a Control-Plane and Data Plane(proxies). Whenever the proxies have to perform a request, they get the config from the **destination** component in the control plane.

![](/static/images/control-plane-5c1c76a5-5431-4134-b08c-0f3bef57fcac.png)

The destination component of the control plane makes sure, it watches for [TrafficSplit](https://github.com/deislabs/smi-spec/blob/master/traffic-split.md) and other CR's and pushes the right config for proxies to folllow. Rather than having it's own configuration format, Linkerd follows the [SMI spec](http://www.smi-spec.io), which aims to have a unified, generalised configuration model for service meshes (just like ingress, CRI, etc in k8s). 

## Demo

First download the Linekrd `edge-19.6.4` by running 

```bash
curl https://run.linkerd.io/install-edge | sh
```

Once that is done, Linkerd Control plane can be installed in the kubernetes cluster by running 

```bash
linkerd install | kubectl apply -f -
```

For this demo, let's use [Istio's BookInfo sample application](https://github.com/istio/istio/tree/master/samples/bookinfo), The bookinfo sample can be retrieved [here](https://istio.io/docs/setup/kubernetes/#downloading-the-release),

Next, the linkerd proxies have to be injected into each pod's manifest and this has to be applied to the cluster. This can be done by running 

`linkerd inject ./samples/bookinfo/platform/kube/bookinfo.yaml | kubectl apply -f -` 

You can see the the following pods running in your cluster i.e 3 review page, 1 product page,1 details page pods.

    kubectl get pods
    NAME                              READY   STATUS    RESTARTS   AGE
    details-v1-9f59fd579-ncgkg        2/2     Running   0          11m
    productpage-v1-66df8b7cd5-nrzwv   2/2     Running   0          11m
    ratings-v1-f5d68559-qjh95         2/2     Running   0          11m
    reviews-v1-64c8bc6778-bzlf8       2/2     Running   0          11m
    reviews-v2-756495d66-brfrr        2/2     Running   0          11m
    reviews-v3-7cdb64bdfd-z2m4q       2/2     Running   0          11m

with the following services.

    kubectl get svc
    NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    details       ClusterIP   10.96.115.62    <none>        9080/TCP   13m
    kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP    7h56m
    productpage   ClusterIP   10.109.31.20    <none>        9080/TCP   13m
    ratings       ClusterIP   10.101.57.168   <none>        9080/TCP   13m
    reviews       ClusterIP   10.105.52.139   <none>        9080/TCP   13m

The productpage is the homepage of the application and it can be viewed by running 

    kubectl port-forward svc/productpage 9080:9080

Now, checking `localhost:9080` would show you a product page, with a list of reviews on the right. Those reviews is being loaded from the `reviews` service which is backed by the 3 reviews pods. As requests are being sent to productpage, it sends to reviews service which randomly sends requests to one of the 3 review pods, as they are of different versions, the difference can be seen as,
 v1 (with no stars) 

![](/static/images/v1-972e7ce4-8e3a-43b6-85f7-e0df617b2724.png)

v2 (with orange stars)

![](/static/images/v2-c3bcd0df-5d6e-4e4e-b4ba-f19e2cda762b.png)

v3 (with black stars)

![](/static/images/v3-9b0b281a-a442-43db-bf7e-ab49a0d8862b.png)

Now, Let's have the reviews service, only split traffic to v2 and v3 pods. 

In Linkerd, For Traffic Split, Services are used as the core objects, so we need two new reviews services that correspond to the pods.

two new services `reviews-v2` and `reviews-v3` are created that correspond to the v2 and v3 pods respectively by applying the following YAML.

```yaml

    apiVersion: v1
    kind: Service
    metadata:
      name: reviews-v2
    spec:
      selector:
        app: reviews
        version: v2
      ports:
      - protocol: TCP
        port: 9080
        targetPort: 9080
    
    ---
    
    apiVersion: v1
    kind: Service
    metadata:
      name: reviews-v3
    spec:
      selector:
        app: reviews
        version: v3
      ports:
      - protocol: TCP
        port: 9080
        targetPort: 9080
```

As we can see now, we will have two new services,

```bash

    kubectl get svc
    NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
    details       ClusterIP   10.96.115.62     <none>        9080/TCP   35m
    kubernetes    ClusterIP   10.96.0.1        <none>        443/TCP    8h
    productpage   ClusterIP   10.109.31.20     <none>        9080/TCP   35m
    ratings       ClusterIP   10.101.57.168    <none>        9080/TCP   35m
    reviews       ClusterIP   10.105.52.139    <none>        9080/TCP   35m
    reviews-v2    ClusterIP   10.106.174.219   <none>        9080/TCP   7s
    reviews-v3    ClusterIP   10.96.125.224    <none>        9080/TCP   7s
```

Now, Let's apply the TrafficSplit CRD which makes the requests to `reviews` service split between `reviews-v` and `reviews-v2` 

```yaml

    apiVersion: split.smi-spec.io/v1alpha1
    kind: TrafficSplit
    metadata:
      name: reviews-rollout
    spec:
      service: reviews
      backends:
      - service: reviews-v2
        weight: 500m
      - service: reviews-v3
        weight: 500m
```

This specified the Linkerd Control Plane that whenever there is a request to `reviews` service on a proxy, split them across the `reviews-v2` and `reviews-v3` based on the weights provided.

So, now if the product page is opened, we can only see the reviews with red or black stars equally which means the traffic is split equally (as equal weights are provided)