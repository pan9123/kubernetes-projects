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

---

# app1-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
  namespace: ingress-lab
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - name: app1
        image: nginxdemos/hello
        ports:
        - containerPort: 80
```

---

# app1-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app1-service
  namespace: ingress-lab
spec:
  selector:
    app: app1
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

---

# app2-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app2
  namespace: ingress-lab
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app2
  template:
    metadata:
      labels:
        app: app2
    spec:
      containers:
      - name: app2
        image: hashicorp/http-echo
        args:
        - "-text=Welcome to APP2"
        ports:
        - containerPort: 5678
```

---

# app2-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app2-service
  namespace: ingress-lab
spec:
  selector:
    app: app2
  ports:
  - port: 80
    targetPort: 5678
  type: ClusterIP
```

---

# Part 1: Basic Host-Based Ingress

# ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-no-auth
  namespace: ingress-lab

spec:
  ingressClassName: nginx

  rules:
  - host: app1.demo.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
```

---

# Prerequisites

- Ubuntu Linux
- Minikube
- kubectl
- NGINX Ingress Controller
  
## Enable NGINX Ingress Controller

> **Important:** Creating an Ingress resource (for example, `ingress.yaml`) and applying it using `kubectl apply -f ingress.yaml` is not enough. An Ingress resource is only a set of routing rules.
>
> To make those rules work, you must have an **Ingress Controller** running in the cluster. The Ingress Controller continuously watches Ingress resources and configures the required routing to forward incoming traffic to the appropriate Kubernetes **Services**, which then direct traffic to the backend Pods.
>
> Common Ingress Controllers include:
>
> - NGINX Ingress Controller
> - Traefik
> - HAProxy Ingress
> - AWS Load Balancer Controller (ALB)
> - Azure Application Gateway Ingress Controller (AGIC)
>
> Without an Ingress Controller, the Ingress resource exists in the cluster, but no traffic routing will occur.

```bash
minikube addons enable ingress
```

Verify that the NGINX Ingress Controller is running:

Note : Ingress Controller in also a Pod

```bash
kubectl get pods -n ingress-nginx
```

Expected Output:

```text
NAME                                        READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-xxxxx              1/1     Running   0          2m
```

### Traffic Flow

```text
Client Request
      |
      v
Ingress Controller
      |
      v
Kubernetes Service
      |
      v
Application Pods
```

---

# Create Namespace

```bash
kubectl create namespace ingress-lab
```

---

# Deploy Resources

```bash
kubectl apply -f app1-deployment.yaml
kubectl apply -f app1-service.yaml

kubectl apply -f app2-deployment.yaml
kubectl apply -f app2-service.yaml

kubectl apply -f ingress.yaml
```

---

# Verification

Check all resources:

```bash
kubectl get all -n ingress-lab
```

Check services:

```bash
kubectl get svc -n ingress-lab
```

Check ingress:

```bash
kubectl get ingress -n ingress-lab
```

Describe ingress:

```bash
kubectl describe ingress ingress-no-auth -n ingress-lab
```

Expected:

```bash
NAME              CLASS   HOSTS             ADDRESS        PORTS   AGE
ingress-no-auth   nginx   app1.demo.local   192.168.49.2   80
```

---

# Configure Host Mapping

Get Minikube IP:

```bash
minikube ip
```

Example:

```text
192.168.49.2
```

Edit hosts file:

```bash
sudo vi /etc/hosts
```

Add:

```text
192.168.49.2 app1.demo.local
```

---

# Testing

Using curl:

```bash
curl http://app1.demo.local
```

Or:

```bash
curl -H "Host: app1.demo.local" http://$(minikube ip)
```

Using browser:
> **Note:** If you are practicing on a cloud-hosted VM (AWS EC2, Azure VM, GCP VM, etc.), this URL will not resolve directly from your local machine because the hostname is mapped only within the VM. In such cases, either:
>
> - Add the hostname entry to the VM's `/etc/hosts` file and test using `curl`, or
> - Access the application using the Ingress IP and appropriate host header.

```text
http://app1.demo.local
```

---

# Sample Output

```bash
kubectl get all -n ingress-lab
```

```text
NAME                        READY   STATUS    RESTARTS   AGE
pod/app1-5c499bc58d-t8sf2   1/1     Running   0          16m
pod/app1-5c499bc58d-t9h8g   1/1     Running   0          16m
pod/app2-78779d7b49-tq97x   1/1     Running   0          16m
pod/app2-78779d7b49-xl74q   1/1     Running   0          16m

NAME                   TYPE        CLUSTER-IP      PORT(S)
service/app1-service   ClusterIP   10.105.78.80    80/TCP
service/app2-service   ClusterIP   10.99.12.181    80/TCP
```

---

# Learning Outcomes

- Created Kubernetes Deployments
- Created ClusterIP Services
- Deployed NGINX Ingress Controller
- Configured Host-Based Routing
- Verified Ingress Connectivity
- Understood Kubernetes Service and Ingress Networking

---

# Part 2: Path-Based Routing

In this implementation, a single Ingress resource routes incoming requests to different backend services based on the URL path.

## Architecture

```text
                    NGINX Ingress
                           |
                      demo.local
                           |
          ---------------------------------
          |                               |
      /app1                           /app2
          |                               |
    app1-service                   app2-service
          |                               |
      App1 Pods                      App2 Pods
```

## Path-Based Routing Flow

| URL | Backend Service |
|------|----------------|
| demo.local/app1 | app1-service |
| demo.local/app2 | app2-service |

---

## path-based-ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-based-ingress
  namespace: ingress-lab

spec:
  ingressClassName: nginx

  rules:
  - host: demo.local
    http:
      paths:

      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80

      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
```

---

## Deploy Path-Based Ingress

```bash
kubectl apply -f path-based-ingress.yaml
```

Verify:

```bash
kubectl get ingress -n ingress-lab
```

Describe Ingress:

```bash
kubectl describe ingress path-based-ingress -n ingress-lab
```

---

## Configure Host Mapping

Get Minikube IP:

```bash
minikube ip
```

Add the following entry to the hosts file:

```bash
sudo vi /etc/hosts
```

```text
192.168.49.2 demo.local
```

Replace the IP address with your Minikube IP if different.

---

## Testing

Test Application 1:

```bash
curl http://demo.local/app1
```

Or:

```bash
curl -H "Host: demo.local" http://$(minikube ip)/app1
```

Expected Output:

```text
Nginx Demo Application Response
```

---

Test Application 2:

```bash
curl http://demo.local/app2
```

Or:

```bash
curl -H "Host: demo.local" http://$(minikube ip)/app2
```

Expected Output:

```text
Welcome to APP2
```

---

## Verification Commands

```bash
kubectl get ingress -n ingress-lab

kubectl describe ingress path-based-ingress -n ingress-lab

kubectl get svc -n ingress-lab

kubectl get endpoints -n ingress-lab
```

---

## Learning Outcomes

- Implemented Path-Based Routing using NGINX Ingress
- Routed traffic to multiple backend services using a single hostname
- Configured URL-based traffic management
- Verified backend service routing through Ingress
- Understood how Kubernetes Ingress maps URL paths to services

# Part 3: Wildcard Host Routing

## Overview

Wildcard Host Routing allows a single Ingress rule to handle requests from multiple subdomains without creating separate rules for each host.

Instead of creating individual entries such as:

```text
dev.demo.local
test.demo.local
qa.demo.local
prod.demo.local
```

a single wildcard host can be used:

```text
*.demo.local
```

This simplifies Ingress management and is commonly used for Development, Testing, QA, and Production environments.

## Architecture

```text
                    NGINX Ingress
                           |
                     *.demo.local
                           |
                     app1-service
                           |
                        App1 Pods
```

## Wildcard Ingress Configuration

### ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wildcard-ingress
  namespace: ingress-lab

spec:
  ingressClassName: nginx

  rules:
  - host: "*.demo.local"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
```

## Apply Configuration

```bash
kubectl apply -f ingress.yaml
```

## Verify Ingress

```bash
kubectl get ingress -n ingress-lab
```

Expected Output:

```text
NAME               CLASS   HOSTS           ADDRESS        PORTS
wildcard-ingress   nginx   *.demo.local    192.168.49.2   80
```

## Configure Host Entries

Edit the hosts file:

```bash
sudo vi /etc/hosts
```

Add:

```text
192.168.49.2 dev.demo.local
192.168.49.2 test.demo.local
192.168.49.2 qa.demo.local
192.168.49.2 prod.demo.local
```

> Replace `192.168.49.2` with your Minikube IP if different.

## Testing

### Test Development Environment

```bash
curl http://dev.demo.local
```

### Test QA Environment

```bash
curl http://qa.demo.local
```

### Test Production Environment

```bash
curl http://prod.demo.local
```

### Alternative Testing Using Host Header

```bash
curl -H "Host: dev.demo.local" http://$(minikube ip)
```

```bash
curl -H "Host: test.demo.local" http://$(minikube ip)
```

```bash
curl -H "Host: prod.demo.local" http://$(minikube ip)
```

## Learning Outcomes

- Implemented Wildcard Host Routing using NGINX Ingress
- Configured a single Ingress rule for multiple subdomains
- Reduced routing configuration complexity
- Understood real-world use cases for Development, QA, and Production environments
- Verified routing using custom host headers


---

# Part 4: TLS Configuration (HTTPS Ingress)

## Overview

Transport Layer Security (TLS) enables secure communication between the client and the application by encrypting traffic. In this lab, a self-signed certificate is used to secure traffic between the client and the NGINX Ingress Controller.

Before TLS:

```text
User
 |
HTTP (Port 80)
 |
Ingress
 |
Application
```

After TLS:

```text
User
 |
HTTPS (Port 443)
 |
NGINX Ingress
 |
Application
```

## Architecture

```text
                    HTTPS Request
                           |
                     app1.demo.local
                           |
                    NGINX Ingress
                           |
                      TLS Secret
                           |
                     app1-service
                           |
                        App1 Pods
```

## Generate Self-Signed Certificate

Create a directory for certificates:

```bash
mkdir certs
cd certs
```

Generate the certificate and private key:

```bash
openssl req -x509 -nodes -days 365 \
-newkey rsa:2048 \
-keyout server.key \
-out server.crt \
-subj "/CN=app1.demo.local/O=ingress-lab"
```

Verify:

```bash
ls
```

Expected Output:

```text
server.crt
server.key
```

---

## Create TLS Secret

```bash
kubectl create secret tls tls-secret \
--cert=server.crt \
--key=server.key \
-n ingress-lab
```

Verify:

```bash
kubectl get secret -n ingress-lab
```

Expected Output:

```text
NAME         TYPE                DATA   AGE
tls-secret   kubernetes.io/tls   2      10s
```

---

## File: tls-ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
  namespace: ingress-lab

spec:
  ingressClassName: nginx

  tls:
  - hosts:
    - app1.demo.local
    secretName: tls-secret

  rules:
  - host: app1.demo.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
```

---

## Deploy TLS Ingress

```bash
kubectl apply -f tls-ingress.yaml
```

Verify:

```bash
kubectl get ingress -n ingress-lab
```

Expected Output:

```text
NAME          CLASS   HOSTS             ADDRESS        PORTS     AGE
tls-ingress   nginx   app1.demo.local   192.168.49.2   80,443    2m
```

---

## Configure Host Mapping

Get Minikube IP:

```bash
minikube ip
```

Example:

```text
192.168.49.2
```

Add the following entry to the hosts file:

```bash
sudo vi /etc/hosts
```

```text
192.168.49.2 app1.demo.local
```

---

## Verify TLS Configuration

```bash
kubectl describe ingress tls-ingress -n ingress-lab
```

Expected Output:

```text
TLS:
  tls-secret terminates app1.demo.local
```

---

## Testing

> **Note:** Use **HTTPS** instead of **HTTP** to access the application, as TLS is enabled for this Ingress configuration.

```text
https://app1.demo.local

Access the application using HTTPS:

```bash
curl -k https://app1.demo.local
```

Or:

```bash
curl -vk https://app1.demo.local
```

### Alternative Testing

```bash
curl -k -H "Host: app1.demo.local" https://$(minikube IP)
```
---

# Part 5: Basic Authentication

## Overview

Basic Authentication protects applications by requiring users to provide a valid username and password before accessing the application.

In this lab, NGINX Ingress is configured to challenge users for credentials before forwarding traffic to the backend service.

## Architecture

```text
                  User
                    |
          Username / Password
                    |
             NGINX Ingress
                    |
            app1.demo.local
                    |
              app1-service
                    |
                 App1 Pods
```

## Generate Authentication File

Install the required package:

```bash
sudo apt update

sudo apt install apache2-utils -y
```

Create a username and password:

```bash
htpasswd -c auth pankaj
```

Example Output:

```text
New password:
Re-type new password:
Adding password for user pankaj
```

Verify:

```bash
cat auth
```

Example:

```text
pankaj:$apr1$xxxxxxxxxxxxxxxxxxxxxxxx
```

---

## Create Kubernetes Secret

Create a secret from the generated authentication file:

```bash
kubectl create secret generic basic-auth \
--from-file=auth \
-n ingress-lab
```

Verify:

```bash
kubectl get secret -n ingress-lab
```

Output:

```text
NAME         TYPE                DATA   AGE
basic-auth   Opaque              1      92s
tls-secret   kubernetes.io/tls   2      51m
```

---

## File: basic-auth-ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: basic-auth-ingress
  namespace: ingress-lab
  annotations:
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    nginx.ingress.kubernetes.io/auth-realm: "Authentication Required"

spec:
  ingressClassName: nginx

  rules:
  - host: app1.demo.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
```

---

## Deploy Authentication Ingress

```bash
kubectl apply -f basic-auth-ingress.yaml
```

Verify:

```bash
kubectl get ingress -n ingress-lab
```

Output:

```text
NAME                 CLASS   HOSTS             ADDRESS        PORTS   AGE
basic-auth-ingress   nginx   app1.demo.local   192.168.49.2   80      4m3s
```

---

## Verification

Verify the authentication secret:

```bash
kubectl get secret basic-auth -n ingress-lab
```

Output:

```text
NAME         TYPE     DATA   AGE
basic-auth   Opaque   1      9m30s
```

Verify all ingress resources:

```bash
kubectl get ing
```

Output:

```text
NAME                 CLASS   HOSTS             ADDRESS        PORTS   AGE
basic-auth-ingress   nginx   app1.demo.local   192.168.49.2   80      4m38s
```

---

## Testing Without Credentials

Get Minikube IP:

```bash
minikube ip
```

Output:

```text
192.168.49.2
```

Send a request without authentication:

```bash
curl -I -H "Host: app1.demo.local" http://$(minikube ip)
```

Expected Output:

```text
HTTP/1.1 401 Unauthorized
Date: Sat, 03 Oct 2026 07:46:59 GMT
Content-Type: text/html
Content-Length: 172
Connection: keep-alive
WWW-Authenticate: Basic realm="Authentication Required"
```

The `401 Unauthorized` response confirms that NGINX Ingress is successfully enforcing authentication.

---

## Testing With Credentials

Send a request using valid credentials:

```bash
curl -u pankaj:<your-password> -H "Host: app1.demo.local" http://$(minikube ip)
```

Example:

```bash
curl -u pankaj:Password@123 -H "Host: app1.demo.local" http://$(minikube ip)
```

Expected Result:

```text
Application response from app1-service
```

A successful response confirms that:

- Basic Authentication is working correctly.
- Credentials are validated by NGINX Ingress.
- Authenticated traffic is forwarded to the backend service.
- Unauthorized users are blocked.

---

## Verification Commands

```bash
kubectl get ingress -n ingress-lab

kubectl describe ingress basic-auth-ingress -n ingress-lab

kubectl get secret basic-auth -n ingress-lab

kubectl get svc -n ingress-lab
```

---

## Learning Outcomes

- Implemented Basic Authentication using NGINX Ingress
- Generated credentials using htpasswd
- Created a Kubernetes Secret for authentication
- Protected applications with username and password authentication
- Verified access control using authenticated and unauthenticated requests
- Understood how NGINX Ingress enforces user authentication

---


# Author

**Pankaj Roy**
