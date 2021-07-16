# K8s Cluster Setup

## SUDO Privilege's are required

Privileged access to your Ubuntu System via sudo command is required

Switch to Bash Shell

```text
bash
```

Verify the SUDO Privileges

```bash
sudo apt update -y
```

## Clone Training files

Clone the code repository created for this training on all the master and worker Nodes

**Step 1:** Clone the repository

```bash
git clone https://github.com/nishanthkumarpathi/Kubernetes-Advanced-Microfocus.git
```

**Step 2:** Move to the cloned repository

```bash
cd "Kubernetes-Advanced-Microfocus/00GettingStarted/"
```

**Step 3:** Enable Executable permissions

```bash
chmod +x *.sh
```

> Run the above commands on all the servers

## Docker Installation - Master Node

Run the Below Command to Install Docker on master and worker node

```bash
chmod +x docker-installation.sh
```

```bash
./docker-installation.sh
```

> Logout from the console and login again before you go to next step

## Docker Installation - Worker Nodes

Run the Below Command to Install Docker on master and worker node

```bash
chmod +x docker-installation.sh
```

```bash
./docker-installation.sh
```

> Logout from the console and login again before you go to next step

## Proxy Configuration - Micro focus Environment

Perform all the below Proxy Configuration Operations on Master Node and Worker Node

Switch to Bash Shell

```text
bash
```

Step 1: Create a systemd drop-in directory for the docker service:

```bash
sudo mkdir /etc/systemd/system/docker.service.d
```

Step 2: Create a file called `/etc/systemd/system/docker.service.d/http-proxy.conf` and add the HTTP\_PROXY and HTTPS\_PROXY environment variable.

```bash
sudo touch /etc/systemd/system/docker.service.d/http-proxy.conf
```

Note : Use Nano Editor for adding the below lines to the above `http-proxy.conf` file. You can also use any other editor

```bash
sudo nano /etc/systemd/system/docker.service.d/http-proxy.conf
```

```bash
[Service]

Environment="HTTP_PROXY=http://web-proxy.in.softwaregrp.net:8080/"

Environment="HTTPS_PROXY=http://web-proxy.in.softwaregrp.net:8080/"

Environment="NO_PROXY=localhost,127.0.0.1,10.0.0.0/8,192.168.0.0/16"
```

> To Save the File use : Ctrl + X then type "y" then Enter

{% hint style="danger" %}
You need to ensure that NO\_PROXY should have your local Network configured, so that local requests to your network are not forwarded to proxy, rather they are sent directly to destination system in the Network
{% endhint %}

Step 3: Create a systemd drop-in directory for the containerd service:

```bash
sudo mkdir -p /etc/systemd/system/containerd.service.d/
```

Step 4: Create a file called `/etc/systemd/system/containerd.service.d/http-proxy.conf` and add the HTTP\_PROXY and HTTPS\_PROXY environment variable.

```bash
sudo touch /etc/systemd/system/containerd.service.d/http-proxy.conf
```

Note : Use Nano Editor for adding the below lines to the above `http-proxy.conf` file. You can also use any other editor

```bash
sudo nano /etc/systemd/system/containerd.service.d/http-proxy.conf
```

```bash
[Service]

Environment="HTTP_PROXY=http://web-proxy.in.softwaregrp.net:8080/"

Environment="HTTPS_PROXY=http://web-proxy.in.softwaregrp.net:8080/"

Environment="NO_PROXY=localhost,127.0.0.1,10.0.0.0/8,192.168.0.0/16"
```

> To Save the File use : Ctrl + X then type "y" then Enter

{% hint style="danger" %}
You need to ensure that NO\_PROXY should have your local Network configured, so that local requests to your network are not forwarded to proxy, rather they are sent directly to destination system in the Network
{% endhint %}

Step 5: Append the Proxy Value is as below to `/etc/environment` file

```bash
sudo nano /etc/environment
```

append the below line of configuration

```bash
export no_proxy=localhost,127.0.0.1,10.0.0.0/8,192.168.0.0/16
```

Step 6: use systemd as the default driver for docker.

```bash
cd ~
```

```bash
sudo cat > daemon.json <<EOF
{
"exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
```

```text
sudo mv daemon.json /etc/docker/
```

Step 7: Flush changes

```bash
sudo systemctl daemon-reload
```

```bash
sudo systemctl restart docker
```

```bash
sudo systemctl restart containerd
```

Step 8: Verify that the configuration has been loaded. Observe the Output

```bash
sudo systemctl show --property Environment docker
```

The above command output should return Proxy Environmental Variables that we have configured before.

**Press Q to exit from the above command**

If you see proxy Environmental Variables in the output, then Goto **Step 9**. 

{% hint style="info" %}
 If you don't observe Proxy Environment Variables, then discuss with your Instructor 😢
{% endhint %}

Step 9: Pull the Busybox Image, to verify weather docker deamon is able to download or not

```bash
docker pull busybox
```

{% hint style="danger" %}
In Case if you get a permission error. Please logout of the machine and login again.
{% endhint %}

{% hint style="info" %}
If the **Step 9** is successful, then good and wait for the Next Instructor Commands
{% endhint %}

{% hint style="danger" %}
If there is an issue, then inform Instructor.
{% endhint %}

{% hint style="info" %}
Now its to go back and install kubernetes. But before you do that confirm with Instructor
{% endhint %}

## Kubernetes Installation - **Master Node Installation**

 Move to the cloned repository

```bash
cd "Kubernetes-Advanced-Microfocus/00GettingStarted/"
```

```text
./k8s-master-containerd.sh
```

> Please observe the output from the above command constantly till it ends.

## Kubernetes Installation - **Worker Nodes Installation**

Move to the cloned repository

```bash
cd "Kubernetes-Advanced-Microfocus/00GettingStarted/"
```

> You need to run these below commands on both Worker Nodes

```text
./k8s-worker-containerd.sh
```

## **Join Worker Nodes to Master Nodes**

#### **On Master Node : Run the following Command**

```bash
kubeadm token create --print-join-command
```

> **Copy the Output of the above command and run that output command on Worker Nodes.**

{% hint style="danger" %}
**Append "sudo" to the above command while pasting it on Worker Node.**
{% endhint %}

#### **On Worker Nodes : Run the following Command**

**The below is a sample command. Please use the command that you got from your Kubernetes Master Server**

{% tabs %}
{% tab title="Sample Command look like this" %}
```text
sudo kubeadm join ip:6443 --token XXXXXXX \
--discovery-token-ca-cert-hash \
XXXXXXXX
```
{% endtab %}
{% endtabs %}

#### On Master Node : Run the Below Command

```text
kubectl get nodes
```

{% hint style="success" %}
Congrats ! If you see the list of nodes, then you have successfully configured the Kubernetes Cluster
{% endhint %}

