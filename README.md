# Kubernetes Setup Using Minikube on Cloud Virtual Machines

This project demonstrates how to set up a Kubernetes cluster using Minikube on a cloud-hosted Linux virtual machine. The same setup can be used on AWS, Microsoft Azure, or Google Cloud Platform (GCP).

---

# Supported Cloud Platforms

## Option 1: AWS EC2

### Recommended Configuration

| Component | Value |
|------------|---------|
| Cloud Provider | AWS |
| Service | EC2 |
| OS | Ubuntu 24.04 LTS / 22.04 LTS |
| Instance Type | t3.medium |
| vCPU | 2 |
| RAM | 4 GB |
| Storage | 20 GB |

### Create EC2 Instance

1. Login to AWS Console.
2. Navigate to **EC2**.
3. Click **Launch Instance**.
4. Select **Ubuntu Server**.
5. Choose **t3.medium** instance type.
6. Create or select an existing SSH key pair.
7. Configure Security Group:

Allow:

```text
SSH (22)
HTTP (80)
HTTPS (443)
NodePort Range (30000-32767)
```

8. Launch the instance.
9. Connect to the instance:

```bash
ssh -i my-key.pem ubuntu@<PUBLIC-IP>
```

---

## Option 2: Microsoft Azure VM

### Recommended Configuration

| Component | Value |
|------------|---------|
| Cloud Provider | Microsoft Azure |
| Service | Virtual Machine |
| OS | Ubuntu 24.04 LTS |
| VM Size | Standard_B2s |
| vCPU | 2 |
| RAM | 4 GB |
| Storage | 30 GB |

### Create Azure VM

1. Login to Azure Portal.
2. Navigate to **Virtual Machines**.
3. Click **Create Virtual Machine**.
4. Select Ubuntu Server.
5. Choose **Standard_B2s** machine size.
6. Configure SSH authentication.
7. Open ports:

```text
22
80
443
30000-32767
```

8. Create VM.
9. Connect via SSH:

```bash
ssh -i my-key.pem azureuser@<PUBLIC-IP>
```

---

## Option 3: Google Cloud Platform (GCP)

### Recommended Configuration

| Component | Value |
|------------|---------|
| Cloud Provider | GCP |
| Service | Compute Engine |
| OS | Ubuntu 24.04 LTS |
| Machine Type | e2-medium |
| vCPU | 2 |
| RAM | 4 GB |
| Storage | 20 GB |

### Create GCP VM

1. Login to GCP Console.
2. Navigate to **Compute Engine**.
3. Click **Create Instance**.
4. Select Ubuntu image.
5. Choose **e2-medium** machine type.
6. Allow HTTP and HTTPS traffic.
7. Create instance.
8. Connect via SSH:

```bash
gcloud compute ssh <INSTANCE-NAME>
```

Or

```bash
ssh <USERNAME>@<PUBLIC-IP>
```

---

# Common Firewall Requirements

Regardless of the cloud provider, ensure the following ports are allowed:

```text
22      - SSH Access
80      - HTTP
443     - HTTPS
30000-32767 - Kubernetes NodePort Services
```

---

# Kubernetes Environment Overview

```text
Cloud VM (AWS / Azure / GCP)
          │
          ▼
      Ubuntu Linux
          │
          ▼
        Docker
          │
          ▼
       Minikube
          │
          ▼
 Kubernetes Cluster
          │
          ▼
Applications & Services
```

---

# Step 1: Update Ubuntu Server

```bash
sudo apt-get update -y && sudo apt-get upgrade -y
```

# Step 2: Install Required Dependency Packages

```bash
sudo apt-get install -y curl apt-transport-https ca-certificates gnupg lsb-release
```

These packages help with:

- Downloading files
- Working with HTTPS repositories
- Managing GPG keys
- Identifying Ubuntu release versions

# Step 3: Install Docker

```bash
sudo apt install docker.io -y
```

Verify Docker Installation:

```bash
docker version
```

# Step 4: Add Current User to Docker Group

```bash
sudo usermod -aG docker $USER
```

Apply the changes:

```bash
newgrp docker
```

# Step 5: Validate Docker Installation

```bash
docker run hello-world
```

# Step 6: Download Minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```

# Step 7: Install Minikube

```bash
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Verify:

```bash
minikube version
```

# Step 8: Start Kubernetes Cluster

```bash
minikube start --driver=docker
```

# Step 9: Check Disk Space

```bash
df -h
```

# Step 10: Verify Minikube Status

```bash
minikube status
```

# Step 11: Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

```bash
chmod +x kubectl
```

```bash
sudo mv kubectl /usr/local/bin/
```

Verify:

```bash
kubectl version --client
```

# Step 12: Verify Kubernetes Cluster

```bash
kubectl get nodes
```

Expected Output:

```console
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   XXm   v1.xx.x
```

At this point, your Kubernetes cluster is ready and you can proceed with:

- Deployments
- Services
- ConfigMaps
- Secrets
- Volumes
- Ingress
- Helm
- Monitoring
- Kubeshark
- Production-style Kubernetes practice
