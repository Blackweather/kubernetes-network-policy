# Exercise N: Isolating the namespaces

## Description
By default all the Pods running in the Kubernetes cluster can communicate between each other.
This can often lead to unexpected behavior in various situations.

One such example is defining namespaces. In Kubernetes, namespaces allow you to isolate resources, so that they are not visible for certain users or services. However the `namespace` resource in Kubernetes does not provide complete isolation by itself and needs to be supplied by a `NetworkPolicy` to block network traffic between the namespaces.

Suppose there are two development teams sharing the cluster resources, you could create a namespace for each of them, configure the specific Role-Based Access (RBAC) policies, to restrict the users from running commands on the other team's cluster, but this is still not enough. The developers deploying services in a single namespace could send network traffic to other namespaces, willingly or unwillingly causing traffic spikes and failures.
That's where the `NetworkPolicy` comes in.

## Diagram
> insert architecture diagram here

## Steps
### Deploy the resources and verify network traffic between Pods across namespaces
1. Make sure your minikube is up and running

2. Create the `alpha` and `beta` namespaces using the provided manifests
Run this command:
```
$ kubectl apply -f namespaces/
```

Verify the namespaces were created by running:
```
$ kubectl get ns
```

Your output should look like this:
```
output
```

Notice two new namespaces were created: `alpha` and `beta`

3. Create the `hello` pods in both namespaces using the provided manifests
Run this command:
```
$ kubectl apply -f pods/
```

// TODO: verify network connections between Pods

### Fill in the NetworkPolicy YAML manifest with appropriate rules and deploy it to the cluster
1. The file *policy/policy.yaml* contains a barebones manifest for a NetworkPolicy that needs to be filled in with the proper rules
2. Create the `NetworkPolicy` by applying the manifest.
Run this command:
```
$ kubectl apply -f policy/
```
3. Verify the policy was created
// TODO: fill in

### Test if your NetworkPolicy is working as expected


### Clean up
To clean up your Kubernetes cluster of the resources created in this exercise run this command:
```
$ kubectl delete -f namespaces/
```