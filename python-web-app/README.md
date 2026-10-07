# Kubernetes Practice: Deploying a Python (Django) Application on Minikube Running in AWS EC2

## 📌 Project Overview

This hands-on Kubernetes project demonstrates how to:

- Build a Docker image for a Python Django application
- Deploy the application using Kubernetes Deployment
- Troubleshoot ImagePullBackOff issues
- Verify Pod self-healing capability
- Expose the application using Kubernetes Services
- Understand Service Discovery concepts
- Push Kubernetes manifests to GitHub
- Learn NodePort and LoadBalancer service types
- Install and use Kubeshark for Kubernetes traffic analysis

---

# 📂 Repository Structure

```text
kubernetes-projects/
│
├── README.md
│
└── python-web-app/
    ├── Dockerfile
    ├── requirements.txt
    ├── deployment.yaml
    └── service.yaml
```

---

# 🔗 Quick Navigation

| File | Description |
|--------|------------|
| python-web-app/Dockerfile | Docker image definition |
| python-web-app/deployment.yaml | Kubernetes Deployment |
| python-web-app/service.yaml | Kubernetes Service |
| python-web-app/requirements.txt | Python dependencies |

---

# Step 1: Clone the GitHub Repository

```bash
git clone https://github.com/iam-veeramalla/Docker-Zero-to-Hero.git
```

Navigate to the application directory:

```bash
cd ~/Docker-Zero-to-Hero/examples/python-web-app
```

Verify the available files:

```bash
ls
```

Output:

```console
Dockerfile
devops
requirements.txt
```

---

# Step 2: Build the Docker Image

Build the Docker image:

```bash
docker build -t pankajf5/python-sample-app-demo:v1 .
```

Verify the image:

```bash
docker images
```

Output:

```console
IMAGE                                                                                ID             DISK USAGE   CONTENT SIZE   EXTRA
gcr.io/k8s-minikube/kicbase:v0.0.51                                                  146cd636030e       1.89GB          532MB
gcr.io/k8s-minikube/kicbase@sha256:4a1c825b61479e6c898851ea66f13c620aaeab6002746e95067fc2c4b38a0b24
                                                                                     4a1c825b6147       1.89GB          532MB    U
hello-world:latest                                                                   5e2309035332       25.9kB         9.49kB    U
pankajf5/python-sample-app-demo:v1                                                   129dd066391b        932MB          239MB
ubuntu:latest
```

---

# Step 3: Create Kubernetes Deployment

Create the Deployment manifest:

**deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-python-app
  labels:
    app: sample-python-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sample-python-app
  template:
    metadata:
      labels:
        app: sample-python-app
    spec:
      containers:
      - name: python-sample-app
        image: pankajf5/python-sample-app-demo:v1
        ports:
        - containerPort: 8000
```

Deploy the application:

```bash
kubectl apply -f deployment.yaml
```

Output:

```console
deployment.apps/sample-python-app created
```

Verify Pods:

```bash
kubectl get pods
```

Output:

```console
NAME                                 READY   STATUS             RESTARTS   AGE
sample-python-app-5df574c4c6-2p9sv   0/1     ImagePullBackOff   0          15s
sample-python-app-5df574c4c6-tr6g4   0/1     ImagePullBackOff   0          15s
```

Verify Deployment:

```bash
kubectl get deploy
```

Output:

```console
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
sample-python-app   0/2     2            0           72s
```

---

# Step 4: Troubleshooting ImagePullBackOff

## Error

```console
Events:
  Type     Reason     Age                 From               Message
  ----     ------     ----                ----               -------
  Normal   Scheduled  118s                default-scheduler  Successfully assigned default/sample-python-app-5df574c4c6-2p9sv to minikube
  Normal   Pulling    26s (x4 over 117s)  kubelet            Pulling image "pankajf5/python-sample-app-demo:v1"
  Warning  Failed     26s (x4 over 117s)  kubelet            pull access denied, repository does not exist or may require authorization
  Warning  Failed     26s (x4 over 117s)  kubelet            Error: ErrImagePull
  Normal   BackOff    12s (x6 over 116s)  kubelet            Back-off pulling image
  Warning  Failed     12s (x6 over 116s)  kubelet            Error: ImagePullBackOff
```

## Check if Minikube Can See the Image

```bash
minikube image ls | grep python-sample-app-demo
```

If no output is displayed, load the image into Minikube:

```bash
minikube image load pankajf5/python-sample-app-demo:v1
```

Verify again:

```bash
minikube image ls | grep python-sample-app-demo
```

Output:

```console
docker.io/pankajf5/python-sample-app-demo:v1
```

Verify deployment:

```bash
kubectl get deploy
```

Output:

```console
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
sample-python-app   2/2     2            2           10m
```

Verify pods:

```bash
kubectl get pods
```

Output:

```console
NAME                                 READY   STATUS    RESTARTS   AGE
sample-python-app-5df574c4c6-2p9sv   1/1     Running   0          10m
sample-python-app-5df574c4c6-tr6g4   1/1     Running   0          10m
```

---

# Step 5: Verify Kubernetes Self-Healing

Check Pod details:

```bash
kubectl get pods -o wide
```

Output:

```console
NAME                                 READY   STATUS    RESTARTS   AGE   IP           NODE
sample-python-app-5df574c4c6-969lm   1/1     Running   0          11m   10.244.0.4   minikube
sample-python-app-5df574c4c6-mq9ff   1/1     Running   0          11m   10.244.0.3   minikube
```

Delete one Pod:

```bash
kubectl delete pod sample-python-app-5df574c4c6-969lm
```

Output:

```console
pod "sample-python-app-5df574c4c6-969lm" deleted from default namespace
```

Verify again:

```bash
kubectl get pods -o wide
```

Output:

```console
NAME                                 READY   STATUS    RESTARTS   AGE   IP           NODE
sample-python-app-5df574c4c6-mnvp4   1/1     Running   0          5s    10.244.0.5   minikube
sample-python-app-5df574c4c6-mq9ff   1/1     Running   0          12m   10.244.0.3   minikube
```

✅ Kubernetes automatically created a replacement Pod. This is known as the **self-healing capability** of Deployments.

---

# Step 6: Verify Application from Inside Cluster

SSH into Minikube:

```bash
minikube ssh
```

Access the application:

```bash
curl -L http://10.244.0.5:8000/demo
```

Output:

```console
<!DOCTYPE html>
<html lang="en">
<head>
<title>CSS Template</title>
```

> Note: This is a Django-based application running under the `/demo` context root.

---

# Step 7: Create Kubernetes Service

Create Service manifest:

**service.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: python-django-sample-app
spec:
  type: NodePort
  selector:
    app: sample-python-app
  ports:
    - port: 80
      targetPort: 8000
      nodePort: 30007
```

Apply Service:

```bash
kubectl apply -f service.yaml
```

Output:

```console
service/python-django-sample-app created
```

Verify:

```bash
kubectl get svc
```

Output:

```console
NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)
kubernetes                 ClusterIP   10.96.0.1      <none>        443/TCP
python-django-sample-app   NodePort    10.105.212.31  <none>        80:30007/TCP
```

---

# Step 8: Push Code to GitHub Repository

Repository:

```text
https://github.com/pan9123/kubernetes-projects.git
```

Configure Git:

```bash
git config --global user.name "Pankaj Roy"
git config --global user.email "pankajroydemo@gmail.com"
git config --global --list
```

Initialize repository:

```bash
git init
git add .
git commit -m "Added Python web app Kubernetes project"
```

Rename branch:

```bash
git branch -M main
```

Add remote:

```bash
git remote add origin https://github.com/pan9123/kubernetes-projects.git
```

Pull latest changes:

```bash
git pull origin main --rebase
```

Push code:

```bash
git push -u origin main
```

---

# Step 9: Test NodePort Service

Check Service:

```bash
kubectl get svc
```

Access Service internally:

```bash
minikube ssh
```

```bash
curl -L http://10.105.212.31:80/demo
```

Output:

```console
<!DOCTYPE html>
<html lang="en">
<head>
<title>CSS Template</title>
```

> Note: Application is accessible inside the cluster.

Exit Minikube:

```bash
exit
```

Get Minikube IP:

```bash
minikube ip
```

Output:

```console
192.168.49.2
```

Access externally:

```bash
curl -L http://192.168.49.2:30007/demo
```

Output:

```console
<!DOCTYPE html>
<html lang="en">
<head>
<title>CSS Template</title>
```

> Note: Accessing the application outside the master node within the organization network.

---

# Step 10: Service Discovery Demonstration

Modify Service selector incorrectly:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: python-django-sample-app
spec:
  type: NodePort
  selector:
    app: sample-python-ap
  ports:
    - port: 80
      targetPort: 8000
      nodePort: 30007
```

Apply changes:

```bash
kubectl apply -f service.yaml
```

Verify access:

```bash
curl -L http://192.168.49.2:30007/demo
```

Output:

```console
curl: (7) Failed to connect to 192.168.49.2 port 30007 after 0 ms: Could not connect to server
```

> Note: Application is not reachable because the Service selector does not match the Pod labels.

Correct the selector:

```yaml
selector:
  app: sample-python-app
```

Apply again:

```bash
kubectl apply -f service.yaml
```

Access the application:

```bash
curl -L http://192.168.49.2:30007/demo
```

Output:

```console
<!DOCTYPE html>
<html lang="en">
<head>
<title>CSS Template</title>
```

> Note: Application is reachable after correcting the selector.

---

# Step 11: LoadBalancer Service

Edit the Service:

```bash
kubectl edit svc python-django-sample-app
```

Change:

```yaml
type: LoadBalancer
```

Verify:

```bash
kubectl get svc
```

Output:

```console
NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)
kubernetes                 ClusterIP      10.96.0.1      <none>        443/TCP
python-django-sample-app   LoadBalancer   10.105.212.31  <pending>     80:30007/TCP
```

> Note: Minikube is running on an AWS EC2 instance. For LoadBalancer Services, a cloud load balancer (ELB/NLB) must be provisioned. Once created, a public IP or DNS endpoint becomes available to expose the application to external users.

---

# How to Install Kubeshark

## Minikube Running in AWS EC2

If you are learning Kubernetes, understanding how traffic flows between services and pods is an essential skill. In this hands-on guide, we will set up Kubeshark on a Minikube cluster running inside an AWS EC2 Ubuntu instance and access the Kubeshark UI from a Windows laptop using SSH port forwarding.

By the end of this tutorial, you will be able to:

- Install Kubeshark
- Capture Kubernetes network traffic
- Access the Kubeshark dashboard remotely
- Monitor live requests flowing through your applications

---

# Architecture Overview

```text
Windows Laptop
|
| SSH Port Forwarding
|
AWS EC2 (Ubuntu)
|
Minikube
|
Kubernetes Pods
|
Kubeshark
```

---

# Prerequisites

Before starting, make sure you have:

- AWS EC2 Ubuntu instance
- Minikube installed and running
- kubectl configured
- A sample Kubernetes application deployed
- SSH access from your Windows machine

---

# Step 1: Verify Minikube Cluster

```bash
kubectl get nodes
```

---

# Step 2: Deploy Your Application

```bash
kubectl get deployment
kubectl get pods
kubectl get svc
```

Example:

```console
NAME                       TYPE       CLUSTER-IP      PORT(S)
python-django-sample-app   NodePort   10.105.212.31  80:30007/TCP
```

---

# Step 3: Install Kubeshark

Download Kubeshark:

```bash
curl -Lo kubeshark https://github.com/kubeshark/kubeshark/releases/latest/download/kubeshark_linux_amd64
```

Make executable:

```bash
chmod +x kubeshark
```

Move to system path:

```bash
sudo mv kubeshark /usr/local/bin/
```

Verify installation:

```bash
kubeshark version
```

---

# Step 4: Start Kubeshark

```bash
kubeshark tap
```

Verify components:

```bash
kubectl get pods -A | grep kubeshark
```

Example:

```console
default       kubeshark-front-686c844595-q928w
default       kubeshark-hub-5cf78b9f4b-skskx
default       kubeshark-worker-daemon-set-bqncw
```

All pods should be in the Running state.

---

# Step 5: Port Forward Kubeshark UI

```bash
kubectl port-forward svc/kubeshark-front 8899:80
```

Expected output:

```console
Forwarding from 127.0.0.1:8899 -> 80
```

Keep this terminal running.

---

# Step 6: Access Kubeshark from Windows

Create SSH tunnel:

```bash
ssh -i $HOME\Downloads\123pan.pem -L 9999:localhost:8899 ubuntu@<EC2-Public-IP>
```

Example:

```bash
ssh -i $HOME\Downloads\123pan.pem -L 9999:localhost:8899 ubuntu@34.xxx.xxx.xxx
```

---

# Step 7: Open the Kubeshark Dashboard

```text
http://localhost:9999
```

---

# Step 8: Generate Traffic

Get Minikube IP:

```bash
minikube ip
```

Generate traffic:

```bash
curl http://192.168.49.2:30007/demo
```

---

# Conclusion

Kubeshark is a powerful Kubernetes traffic analyzer that allows engineers to inspect real-time communication among pods and services. When combined with Minikube on AWS EC2, it provides an excellent learning environment for mastering Kubernetes networking and observability concepts.

For anyone preparing for Kubernetes, DevOps, SRE, or Platform Engineering roles, Kubeshark is an excellent tool to understand the network layer of containerized applications and troubleshoot issues faster.
