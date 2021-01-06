# Exercise 5: Isolating the backend of an application

## Description
In this scenario we use `NetworkPolicy`  to block the communication between two or more services residing in Pods in Kubernetes cluster.

Since we know that Network Policies are applied to Pods using labels and selectors, we will use this mechanism to provide security policies within one cluster.

In our exercise we have an environment in which reside 3 separate applications.
The images are already defined in Deployment files. Two of them, called `front 1` and `front 2` will communicate with the `backend`. Front apps are just static files which are hosted by using nginx service and the Backend app is written in .NET Core.

As we learned, once a policy is applied to a Pod, only allowed traffic in the policy will be permitted (which is called default-deny). So, this scenario can be useful when our cluster contains some running apps, which can communicate between each other and we would like to block the communication from on of them. The policy ensures that we only allow network traffic which is declared by us and nothing else. 

## Diagram
![](https://raw.githubusercontent.com/Blackweather/kubernetes-network-policy/master/ex5-backend-isolation/img/arch-diagram.png)

## Steps
### Deploy the resources and verify network traffic between Pods and host machine
Make sure your minikube is up and running (with the Calico plugin!)

Create the `backend`, `front1` and `front2` pods in the default namespace using the provided manifests.
All three Pods will be exposed outside the cluster for the demonstration using a `Service` of type `LoadBalancer`.

Run this command:
```
$ kubectl apply -f pods/
```

To verify the Pods and Services have been created run this command:
```
$ kubectl get po,svc
```

Your output should look like this:
```
pod/backend   1/1     Running   0          8s
pod/front1    1/1     Running   0          8s
pod/front2    1/1     Running   0          8s

NAME                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/backend      LoadBalancer   10.111.130.159   <pending>     5000:32430/TCP   8s
service/front1       LoadBalancer   10.102.104.162   <pending>     8080:31952/TCP   8s
service/front2       LoadBalancer   10.101.154.16    <pending>     8081:31356/TCP   8s
```

`EXTERNAL-IP` fields are marked as `<pending>` since type of the service is LoadBalancer. With this method, each service gets its own IP address.

In minikube, this can be achieved using following command:
(run in new, seperate window with **Administrator** rights)
```
$ minikube tunnel
```
which will expose all 3 services of type `LoadBalancer`.
The output will look like this:
```
$ minikube tunnel
* Starting tunnel for service backend.
* Starting tunnel for service front1.
* Starting tunnel for service front2.
```
And external IP field will be assigned.

### WARNING!
If the external IPs are NOT 127.0.0.1 (localhost), you are using newer version of minikube.
To make it work you have to open a new terminal and port forward backend service to localhost. 
Enter the following command:
```
$ kubectl port-forward svc/backend 5000:5000
```

### Verify Pods are running and working
When the services are being tunneled, you can open a browser and type 2 addresses:

- EXTERNAL-IP:8080 for front1 app
- EXTERNAL-IP:8081 for front2 app

After 1-2 seconds you will be greeted from the `backend` server with the current time.

To verify connection between the Pods we will have to execute a `curl` command inside each of the pods.

#### Connection from front1 to backend
Connect to the `front1` Pod.
Run this command:
```
$ kubectl exec -it front1 -- /bin/sh
```

Call API endpoint by using `curl`:
```
/ # curl backend:5000/app
```

Repeat this for `front2` Pod.

Exit out of the nginx Pod and test the connection from `front2` Pod to `backend` Pod.
Run this command:
```
$ kubectl exec -it backend -- /bin/bash
```
And then, request the content of index.html from nginx:
```
/ # curl front1:8080
```
```
/ # curl front2:8081
```

Backend app logs all the incoming requests which can be checked by typing following command:
```
$ kubectl logs backend
```

### Fill in the NetworkPolicy YAML manifest with appropriate rules and deploy it to the cluster
The file *policy/policy.yaml* contains a barebones manifest for a NetworkPolicy that needs to be filled in with proper rules.
Create the `NetworkPolicy` by applying the manifest.

Run this command:
```
$ kubectl apply -f policy/
```

Verify the policy was created
Run this command:
```
$ kubectl get networkpolicy
```

### Test if your NetworkPolicy is working as expected
After applying the policy `front1` should have connection to `backend`! On the other hand, `front2` traffic must be filtered.

To check if the `front2` can't connect to `backend`, use `curl` with a timeout, for example:
```
/ # curl --max-time 5 nginx:8080
```

1. Test if `front2` has connection with `backend`.
2. Test if `front2` has connection with `front1`.
3. Test if the Pods can still connect to the Internet (for example: `curl auschwitz.org`)

### Clean up
To clean up your Kubernetes cluster of the resources created in this exercise, run the following commands:
```
$ kubectl delete -f policy/
$ kubectl delete -f pods/
```

### Summary
This exercise showed us how to manipulate network traffic between separated pods using ``NetworkPolicy``.

For grading, please send **screenshots** of:
- `front app` in the browser.
- log trace from `backend app` with 'kubectl logs' cmd.
- log of failed request (after applying policy) from curl.