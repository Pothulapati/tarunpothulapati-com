---
Title: An Add-On Model for Linkerd
date: 2012-02-28
tags: ["linkerd", "service-mesh", "distributed-tracing", "addons"]
---

I've been working on an Add-On Model for [Linkerd]() under Linux Foundation's Community Bridge Program, with support from Juraci and Thomas. Thanks to them, It was not only great fun working with the project but also the learning curve was good. So, I will start from the requirements, and then tell the story on how it shaped up finally along with the trade-offs that we had to take (P.S There are always Trade-Offs!!!).

So, To give you the context, You can install the Linkerd Control Plane in two ways, i.e **Linkerd CLI** and **Helm**. To make it easy for maintiance, and not to do the same thing twice, We re-use the same helm templates that we wrote for helm, with the CLI and generate the manifests which users can apply to the cluster. 

With that, Let's get in!

## What's an AddOn Model

Specific to Linkerd, We wanted an add-on model that would allow users to install other components along with the Linkerd Control Plane. Think of Jaeger, Cert Manager, etc as some add-ons that would go well with Linkerd. The main goal is to provide users an out of box experience using which they can get started, and then start using stuff that makes sense to their project and priorities.

 For that, You could ask me Tarun, Why are you reinventing the wheel? Users can always use the official helm charts to install those components. The problem is that, We want a way for the Linkerd Control Plane components to influence the configuration of the component and also Vice Versa (the add-on components influencing/configuring the control plane installation.)



