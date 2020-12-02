# Exercise 0: Setting up the environment

In this excercise you'll install all the required software to run Kubernetes cluster inside your system.

Depending on your system, choose either:
*   Windows,
*   Linux (Ubuntu)

For Windows, we recommend **Chocolatey Package Manager** to download Minikube and Kubectl.
To install *Choco*, open cmd.exe with administrative rights and run the following command:
```
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "[System.Net.ServicePointManager]::SecurityProtocol = 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```
Or, open Power-Shell with administrative rights and run the following command:
```
Get-ExecutionPolicy
```
If it returns Restricted, then run **Set-ExecutionPolicy AllSigned** or **Set-ExecutionPolicy Bypass -Scope Process**
Then run:
```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```
Then you can use `choco` command. 

In case of any trouble, follow the official docs:
*   https://docs.docker.com/docker-for-windows/install/
*   https://docs.docker.com/engine/install/ubuntu/
*   https://minikube.sigs.k8s.io/docs/start/
*   https://kubernetes.io/docs/tasks/tools/install-kubectl/

## Windows

If you have already installed Docker for Windows, proceed to Minikube installation. 

**Docker for Windows**

---
*   Go to [Docker docs](https://docs.docker.com/docker-for-windows/install/)
*   Press *'Download from Docker Hub'* button
*   Ensure the Enable Hyper-V Windows Features option is selected
*   Follow the instructions on the installation wizard
*   Start Docker Desktop

**Minikube**

---
*   If you have Chocolatey Package Manager, type:
```
choco install minikube
```
voila!
*   Otherwise, download and run the [Windows installer](https://storage.googleapis.com/minikube/releases/latest/minikube-installer.exe)
*   Check if `minikube version` works.

**Kubectl**

---
*   If you have Chocolatey Package Manager, type:
```
choco install kubernetes-cli
```
voila!

*   Otherwise, use `curl` and run:
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.19.0/bin/windows/amd64/kubectl.exe
```
And add the binary in to your PATH.
*   Check if `kubectl version` works.

## Linux (Ubuntu)

**Docker**

---
*   Follow [official documentation](https://docs.docker.com/engine/install/ubuntu/)

**Minikube**

---
* Either binary download:
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
* or Debian package:
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
sudo dpkg -i minikube_latest_amd64.deb
```
* or RPM package:
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-latest.x86_64.rpm
sudo rpm -ivh minikube-latest.x86_64.rpm
```
**Kubectl**

---
*   Run the following commands:
```
sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2 curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

## Starting up a local Kubernetes cluster
Assuming you have minikube and kubectl installed, run the following command:
```
$ minikube start --network-plugin=cni --cni=calico
```

This can take a moment the first time you run that command.

To verify that your kubectl is configured properly to use with Minikube run the following command:
```
$ kubectl get no
```

The output should list your single-node Minikube cluster as ready:
```
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   30d   v1.19.2
```

If you wish to stop minikube at any time, run the following command:
```
$ minikube stop
```

To completely delete the minikube cluster, run the following command:
```
$ minikube delete
```