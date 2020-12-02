# Exercise N: Isolating the namespaces

## Description
By default all the Pods running in the Kubernetes cluster can communicate between each other.
This can often lead to unexpected behavior in various situations.

One such example is defining namespaces. In Kubernetes, namespaces allow you to isolate resources, so that they are not visible for certain users or services. However the `namespace` resource in Kubernetes does not provide complete isolation by itself and needs to be supplied by a `NetworkPolicy` to block network traffic between the namespaces.

Suppose there are two development teams sharing the cluster resources, you could create a namespace for each of them, configure the specific Role-Based Access (RBAC) policies, to restrict the users from running commands on the other team's cluster, but this is still not enough. The developers deploying services in a single namespace could send network traffic to other namespaces, willingly or unwillingly causing traffic spikes and failures.
That's where the `NetworkPolicy` comes in.

## Diagram
![](https://raw.githubusercontent.com/Blackweather/kubernetes-network-policy/master/exN-namespace-isolation/img/arch-diagram.png)

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
NAME                   STATUS   AGE
alpha                  Active   7s
beta                   Active   7s
default                Active   30d
kube-node-lease        Active   30d
kube-public            Active   30d
kube-system            Active   30d
kubernetes-dashboard   Active   30d
```

Notice two new namespaces were created: `alpha` and `beta`

3. Create the `hello` pods in both namespaces using the provided manifests
Run these commands:
```
$ kubectl apply -f pods/alpha/
$ kubectl apply -f pods/beta/
```

To verify the Pods have been created run this command:
```
kubectl get po -n <namespace>
```

Your output should look like this:
```
NAME        READY   STATUS    RESTARTS   AGE
pod/hello   1/1     Running   0          60s
```

Make sure the `hello` pod is running for both namespaces.

### Verify Pods can access each other
The Pods that were created host a simple web server on port 8080, which is then automatically exposed internally to the cluster by a private IP address.

To verify connection between the two Pods we will have to find the private IP of a Pod in a given namespace and then connect to a Pod in the other namespace and ensure it gets a successful response.

1. Get the IP address of the Pod running in the `beta` namespace
Run this command:
```
$ kubectl get po -o wide -n beta
```

Your output should look like this:
```
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
hello   1/1     Running   0          12m   172.17.0.6   minikube   <none>           <none>
```

Copy the IP field, it will be used later to test connection.

2. Connect to the Pod running in the `alpha` namespace
Run this command:
```
$ kubectl exec -n alpha -it hello -- /bin/sh
```

If it connects to the alpha Pod, a different command prompt should appear:
```
/ #
```

To exit from the Pod at any time, type `exit`. This will not cause the Pod to stop running.

3. Test connection from `alpha` to `beta`
Since the Pod is using a barebones alpine linux image, we have to install `curl` for the ease of testing connections. If you don't want to install `curl`, feel free to try `telnet` or `nc`.
Run these commands:
```
/ # apk update
/ # apk add curl
```

To test the connection use the private IP we got earlier on.
Run this command:
```
/ # curl {IP}:8080
```

You should get a response:
```
Hello, world!
Version: 1.0.0
Hostname: hello
```

Exit out of the alpha Pod and repeat the process to test the connection from beta to alpha namespace.

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

### Summary
This exercises allowed us to discover a use case for NetworkPolicy and a way to implement them.

For grading, please send screenshots of:
- running Pods with their IPs listed,
- connection checks between the namespaces (before and after applying the NetworkPolicy, make sure the Pod name you are connected to is visible).
Please also send the NetworkPolicy manifest(s). 