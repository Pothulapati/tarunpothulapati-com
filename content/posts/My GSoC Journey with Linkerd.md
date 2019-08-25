---
Title: Google Summer of Code 2019 at Linkerd
date: 2019-08-24
tags: ["linkerd", "smi", "service-mesh", "gsoc", "traffic"]
---


## My GSoC Journey with Linkerd

My GSoC Internship started with a different aim i.e to write a common GraphQL API to query linkerd metrics and my initial proposal was oriented towards it. I worked on the whole GraphQL spec along with the documentaion and relevant rfc's for it. By the end of community bonding period, we had a concrete proposal and I also got a scholarship to attend KubeCon + CloudNativeCon 2019 EU in Barcelona and get to meet my mentor and the whole Linkerd team. We had great fun discussing and clarifying my doubts regarding my internship and the linkerd project as a whole!

But then some companies in the service mesh space like Linkerd, Azure, Kinvolk, solo.io etc announced [Service Mesh Interface](https://smi-spec.io) which aims to provide a common interface for whole service mesh technologies so that common tools can be built on top of it. This seemed like a great platform to build the graphql API. Then I discussed with my mentor, linkerd team and the other SMI folks and decided that it would be best to spend my time on SMI-metrics which aims to solve a similar problem but on a bigger scale i.e for service meshes on whole.

The initial work was to make the smi-metrics API Server support more service meshes like Istio and Consul Connect. As this work can be a bit hectic and required learning a lot of things about Istio and Consul. My mentor suggested me to continue my contributions on Linkerd while also working on SMI-metrics. So, that I don't get struck on one thing.

So, My GSoC work is divied into my contributions to SMI-metrics and also the Linkerd project.

## SMI-Metrics

Intially the SMI-Metrics, had a very hard dependency on prometheus for metrics as it was still a prototype. This can be a challenge for service mesh projects that don't depend on prometheus for queries or had a drastically different structure of time-series (i.e labels, sum of multiple metrics,etc ). So, I worked on adding a Mesh Interface and de-coupling the prometheus client. This made the repo more extensiable i.e now service meshes can just implement the interface and reply those metrics with or without prometheus.

During my GSoC period, I successfully got the `istio` support for SMI-Metrics working and also created the release automation for the project.

Istio support is available from Smi-Metrics version `v0.2.0`

## Linkerd

Throughout the GSoC period, I also kept my contributions up to Linkerd solving issues ranging from small bug fixes to small scale features.

The total Linkerd and SMI-metric contributions can be found [here](https://github.com/Pothulapati/gsoc-meta-linkerd).

## Conclusion

I really had a great time working with the Linkerd team especially Thomas (my mentor). Their struggle for simplicity and ultralight-weight service mesh can be seen in each and every PR. The team definitely deserves a huge win and we are already seeing that happen ;). They really own the project and make sure all the incoming PR's follow standards. I got to learn a lot from them. Thomas is a great mentor and always gives me long sighted suggestions, insights and also fun tech rants(:p). I definitely can't ask for a better mentor. Thanks to Ivan, Alex, Alejandro, Andrew for unblocking me wherever possible and being friendly. <3

Special thanks to William, for the hospitaly at the Team dinner where I felt a bit awkward and alone :p . I will remember that for a long time.

Linkerd team is a dream team, and it would be really sad to not continue my contributions. I will be continuing my contributions to Linkerd and also will be taking up the work to make Tracing a first class component(like prometheus and grafana) in Linkerd. :)
