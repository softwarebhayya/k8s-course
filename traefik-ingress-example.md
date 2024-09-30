
# Traefik v3 Ingress Controller Installation on Kubernetes (Docker Desktop, macOS)

This document contains all the commands you need to install Traefik v3 on Kubernetes using Docker Desktop and expose applications using `Ingress` resources.

---

## Prerequisites

Make sure you have:
- Docker Desktop with Kubernetes enabled.
- `kubectl` installed and pointing to your Docker Desktop cluster.
- `helm` installed.

---

## 1. Add Traefik Helm Repository

```bash
helm repo add traefik https://traefik.github.io/charts
helm repo update
```

---

## 2. Install Traefik with IngressClass

Create the `traefik` namespace and install Traefik with an `IngressClass`:

```bash
kubectl create namespace traefik

helm install traefik traefik/traefik \
  --namespace traefik \
  --set service.type=LoadBalancer \
  --set ingressClass.enabled=true \
  --set ingressClass.isDefaultClass=true
```

---

## 3. Verify Installation

Check if Traefik's pods and services are running:

```bash
kubectl get pods -n traefik
kubectl get svc -n traefik
```

Ensure the Traefik `IngressClass` is created:

```bash
kubectl get ingressclass
```

---

## 4. Deploy a Sample Application (helloapp)

### 4.1. Create hello app pod

Save the following as `helloapp-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: helloapp
  labels:
    app: helloapp
spec:
  containers:
  - name: helloapp
    image: softwarebhayya/helloapp
    ports:
    - containerPort: 8080
```


### 4.2. Create hello app service

Save the following as `helloapp-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: helloapp-service
spec:
  selector:
    app: helloapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
```

### 4.3. Create hello app ingress

Save the following as `helloapp-ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: web
spec:
  ingressClassName: traefik
  rules:
  - host: helloapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: helloapp-service
            port:
              number: 80
```

Apply the files:

```bash
kubectl apply -f helloapp-pod.yaml
kubectl apply -f helloapp-service.yaml
kubectl apply -f helloapp-ingress.yaml
```

---

## 5. Update `/etc/hosts` File

To access `helloapp.local`, add the following line to your `/etc/hosts` file:

```bash
127.0.0.1 helloapp.local
```


## 6. Test the Ingress

Now, open your browser and visit:

```
http://helloapp.local
```

---

## 7. Verify Traefik Logs

Check the Traefik logs to ensure the routing is working properly:

```bash
kubectl logs -l app.kubernetes.io/name=traefik -n traefik
```

---
## Conclusion

This document contains all commands to set up Traefik v3 as the Ingress controller using `IngressClass` on Docker Desktop's Kubernetes. After following these steps, you can expose applications using standard Kubernetes `Ingress` resources.
