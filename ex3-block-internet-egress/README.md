# Exercise 3: Block outgoing Internet connections

## Description
In this scenario we use `NetworkPolicy` to block the Internet connection.

As a result some all Pods will not be able to connect to the Internet but will still be able to communicate with each other.

If we know that our Pods should not connect to the Internet, it is good practice to block this access. This will prevent infiltration by malicious websites and prevent data leakage if the services have been misconfigured. If the network has been compromised for any reason, blocking Internet connectivity will prevent the malware from communicating with the control system.

Additionally deploying this configuration is a good basis for securing the network connections in a Kubernetes cluster. We assume that the services will not send any data out of cluster and if we need to modify this behavior, we deploy more NetworkPolicy resources to allow exceptions. Following this approach will help increase the general level of network security in the cluster and is called default deny.

## Diagram
![](https://raw.githubusercontent.com/Blackweather/kubernetes-network-policy/master/ex3-block-internet-egress/img/arch-diagram.png)

## Steps
### Deploy the resources and verify network traffic between Pods and host machine
Make sure your `minikube` is up and running (with the Calico plugin!)

Create the `nginx` and `hello` pods in the default namespace using the provided manifests. The `nginx` Pod will be exposed outside the cluster using a Service of type `NodePort` and the `hello` Pod will be exposed internally using a Service of type `ClusterIP`. Run this command:
```
$ kubectl apply -f pods/
```
To verify the Pods and Services have been created run this command:
```
$ kubectl get po,svc
```
Your output should look like this:
```
NAME            READY   STATUS    RESTARTS   AGE
pod/hello       1/1     Running   0          4s
pod/nginx       1/1     Running   0          4s

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/hello        ClusterIP   10.109.59.236    <none>        8080/TCP         4s
service/kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          3h33m
service/nginx        NodePort    10.110.115.247   <none>        8080:30127/TCP   4s
```
If the STATUS is ContainerCreating wait a few seconds and run the same command again.

### Verify Pods can access each other
Both of them are exposed internally in the cluster on port 8080 using the `ClusterIP` services.

To verify connection between the two Pods we will have to execute a curl command inside each of the pods.

#### Connection from nginx to hello
1. Connect to the `nginx` Pod
```
$ kubectl exec -it nginx -- /bin/bash
```
If it connects to the nginx Pod, a different command prompt should appear:
```
root@nginx:/# 
```
To exit from the Pod at any time, type `exit`. This will not cause the Pod to stop running.

2. Test the connection from `nginx` to `hello`
Since the Pod is using a barebones ubuntu linux image, we have to install `curl` for the ease of testing connections. If you don't want to install `curl`, feel free to try `telnet` or `nc` (these are not pre-installed either).
Run these commands:
```
root@nginx:/# apt-get update
root@nginx:/# apt-get install -y curl
```

To test the connection to the `hello` server we can use the ClusterIP service and thus do not need to find the private IP.
Run this command:
```
root@nginx:/# curl hello:8080
```

You should get a response:
```
Hello, world!
Version: 1.0.0
Hostname: hello
```

Exit out of the nginx Pod and test the connection hello Pod to nginx Pod.

#### Connection from hello to nginx
1. Connect to the `hello` Pod
Run this command:
```
$ kubectl exec -it hello -- /bin/sh
```

If it connects to the hello Pod, a different command prompt should appear:
```
/ #
```

2. Test the connection from `hello` to `nginx`
Since the Pod is using a barebones alpine linux image, we have to install `curl`.
Run these commands:
```
/ # apk update
/ # apk add curl
```

To test the connection to the `nginx` server we can use the ClusterIP service and thus do not need to find the private IP.
Run this command:
```
/ # curl nginx:8080
```

You should get a response, which is the default nginx web server home page and hello answer.

Exit out of the hello Pod and test the connection from "public" to nginx.

#### Connection from "public" network to nginx
For the purpose of this example, the nginx server was also exposed outside the minikube cluster with a service of type `NodePort` and can be accessed with an open port on the local minikube cluster node.
This was configured to illustrate the incoming connections from the public internet.

1. Find the address of the local minikube node.
Run this command:
```
$ kubectl get no -o wide
```

Your output should look similar to this:
```
NAME       STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
minikube   Ready    master   3h    v1.19.4   <INTERNAL-IP>   <none>        Ubuntu 20.04.1 LTS   4.19.76-linuxkit   docker://19.3.13
```

Note the `INTERNAL-IP` field, it will be used to connect to nginx.

Note: in a more realistic scenario we would have multiple Kubernetes nodes here and any of their internal IPs could have been used to access the service.

2. Find the exposed port
Run this command:
```
$ kubectl get svc
```

Your output should look similar to this:
```
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
hello        ClusterIP   10.109.59.236    <none>        8080/TCP         13m
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          3h46m
nginx        NodePort    10.110.115.247   <none>        8080:<EXPOSED_PORT>/TCP   13m
```

Note the `EXPOSED_PORT` field, it will be used to connect to nginx.

3. Test the external connection.
Use the browser to go to `INTERNAL-IP:EXPOSED_PORT` or just run
```
$ curl INTERNAL-IP:EXPOSED_PORT
```

either way the result should be the nginx default home page.

### Fill in the NetworkPolicy YAML manifest with appropriate rules and deploy it to the cluster
1. The file *policy/policy.yaml* contains a barebones manifest for a NetworkPolicy that needs to be filled in with proper rules.
2. `NetworkPolicy` rules have a policyTypes field, which is Ingress by default. Editing outbound traffic requires this field to be specified and set to Engress.

HINT:
- Connect to website works with DNS
3. Create the `NetworkPolicy` by applying the manifest.
Run this command:
```
$ kubectl apply -f policy/
```
3. Verify the policy was created
Run this command:
```
$ kubectl get networkpolicy
```

### Test if your NetworkPolicy is working as expected
To check if the Pods cannot connect to the Internet, use `curl` with a timeout, for example:
```
/ # curl --max-time 5 google.pl:8080
```

1. Test if the `hello` and `nginx` Pods can't connect to the Internet
2. Test if the `hello` and `nginx` Pods can connect each other
3. Test if the `nginx` Pod is accesible by using the `NodePort` service

### Clean up
To clean up your Kubernetes cluster of the resources created in this exercise, run the following commands:
```
$ kubectl delete -f policy/
$ kubectl delete -f pods/
```

### Summary
This exercise allowed us to discover a use case for NetworkPolicy and a way to implement them.

For grading, please send screenshots of:
- running Pods, Services and the NetworkPolicy
- connection checks (before and after applying the NetworkPolicy, make sure the Pod name you are connected to is visible).

Please also send the NetworkPolicy manifest(s).

