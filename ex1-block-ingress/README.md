# Exercise 1: Block incoming connections

## Description
In this scenario we use `NetworkPolicy` to block the ingress connections to all the Pods running in the Kubernetes cluster.

As a result all the Pods should be inaccessible by other Pods in the cluster and from the public network.
Notice that we will not be blocking access to resources completely. We are just blocking access to the Services that expose the Pods. Administrative access will still be available, so you can still use `kubectl` just fine.

This configuration can be used, when we do not need direct access to the services running on the cluster. We do not need such access for example when our services just download data from a data source of some kind (for example: public Internet), do calculation on it, send the result somewhere and end their lifecycles. These could be for example Kubernetes *Jobs* or *CronJobs*.

Additionally deploying this configuration is a good base for securing the network connections in a Kubernetes cluster. We assume that services will not be receiving any incoming connections and if there is a need to modify this behavior, we deploy next NetworkPolicy resources to allow exceptions. This scheme allows to increse the general level of network security in the cluster and is called *default deny*. 
In the scenario we implement default deny for *Ingress* traffic, but we could also implement it for *Egress* traffic which will be the focus of the next exercise. 

## Diagram
![](https://raw.githubusercontent.com/Blackweather/kubernetes-network-policy/master/ex1-block-ingress/img/arch-diagram.png)

## Steps


### Summary