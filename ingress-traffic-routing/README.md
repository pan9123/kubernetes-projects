# Kubernetes Ingress Traffic Routing

This project demonstrates:

- Deployment creation
- ClusterIP Services
- NGINX Ingress Controller
- Multi-Host-based Routing
- Path-based Routing
- Wildcard Host Routing
- TLS Configuration
- Authentication

# Kubernetes Ingress Traffic Routing

This project demonstrates how to expose Kubernetes applications using the NGINX Ingress Controller. Two applications are deployed inside a Kubernetes cluster, exposed through ClusterIP services, and accessed using an Ingress resource.

## Architecture

```text
                    NGINX Ingress
                           |
                    app1.demo.local
                           |
                    app1-service
                           |
                         App1 Pods
```

## Project Structure

```text
ingress-traffic-routing/
├── README.md
├── app1-deployment.yaml
├── app1-service.yaml
├── app2-deployment.yaml
├── app2-service.yaml
└── ingress.yaml
```

## Resources Created

### Application 1

- Deployment: app1
- Replicas: 2
- Image: nginxdemos/hello

### Application 2

- Deployment: app2
- Replicas: 2
- Image: hashicorp/http-echo

### Services

- app1-service (ClusterIP)
- app2-service (ClusterIP)

### Ingress

- Ingress Name: ingress-no-auth
- Ingress Class: nginx
- Host: app1.demo.local

## Prerequisites

- Ubuntu Linux
- Minikube
- kubectl
- NGINX Ingress Controller

## Enable NGINX Ingress Controller

```bash
minikube addons enable ingress
```

Verify:

```bash
kubectl get pods -n ingress-nginx
```

## Create Namespace

```bash
kubectl create namespace ingress-lab
```

## Deploy Application Resources

```bash
kubectl apply -f app1-deployment.yaml
kubectl apply -f app1-service.yaml

kubectl apply -f app2-deployment.yaml
kubectl apply -f app2-service.yaml
```

## Deploy Ingress

```bash
kubectl apply -f ingress.yaml
```

## Verify Resources

Check all resources:

```bash
kubectl get all -n ingress-lab
```

Check services:

```bash
kubectl get svc -n ingress-lab
```

Check pods:

```bash
kubectl get pods -n ingress-lab
```

Check ingress:

```bash
kubectl get ingress -n ingress-lab
```

Describe ingress:

```bash
kubectl describe ingress ingress-no-auth -n ingress-lab
```

## Sample Output

```bash
kubectl get ingress -n ingress-lab

NAME              CLASS   HOSTS             ADDRESS        PORTS   AGE
ingress-no-auth   nginx   app1.demo.local   192.168.49.2   80      4m
```

## Configure Host Mapping

Get Minikube IP:

```bash
minikube ip
```

Edit hosts file:

```bash
sudo vi /etc/hosts
```

Add the following entry:

```text
192.168.49.2 app1.demo.local
```

Replace the IP address with your Minikube IP if different.

## Testing

Using curl:

```bash
curl http://app1.demo.local
```

Or

```bash
curl -H "Host: app1.demo.local" http://192.168.49.2
```

Using a browser:

```text
http://app1.demo.local
```

## Learning Outcomes

- Created Kubernetes Deployments
- Created ClusterIP Services
- Configured NGINX Ingress Controller
- Implemented Host-Based Routing
- Verified Ingress Connectivity
- Understood Service and Ingress Networking Concepts

## Author

**Pankaj Roy**
