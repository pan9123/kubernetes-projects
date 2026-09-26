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

```bash
minikube addons enable ingress
```

Verify:

```bash
kubectl get pods -n ingress-nginx
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

# Author

**Pankaj Roy**
