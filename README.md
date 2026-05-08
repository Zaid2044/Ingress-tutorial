# Kubernetes Ingress Routing Demo Using KIND

## Create Project Directory

```bash
mkdir ingress-demo
cd ingress-demo
mkdir k8s
```

---

## Create KIND Config

### kind-config.yaml

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
  - role: control-plane

    extraPortMappings:
      - containerPort: 80
        hostPort: 80
        protocol: TCP

      - containerPort: 443
        hostPort: 443
        protocol: TCP
```

---

## Create KIND Cluster

```bash
kind create cluster --name ingress-cluster --config kind-config.yaml
```

---

## Install NGINX Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

---

## Wait For Ingress Controller

```bash
kubectl get pods -n ingress-nginx
```

---

## Create Namespace

```bash
kubectl create namespace ingress-demo
```

---

# APP 1

## Create Deployment YAML

```bash
kubectl create deployment app1 \
--image=nginx \
-n ingress-demo \
--dry-run=client -o yaml > k8s/app1-deploy.yaml
```

---

## Apply Deployment

```bash
kubectl apply -f k8s/app1-deploy.yaml
```

---

## Create Service YAML

```bash
kubectl expose deployment app1 \
--port=80 \
--target-port=80 \
--type=ClusterIP \
-n ingress-demo \
--dry-run=client -o yaml > k8s/app1-svc.yaml
```

---

## Apply Service

```bash
kubectl apply -f k8s/app1-svc.yaml
```

---

# APP 2

## Create Deployment YAML

```bash
kubectl create deployment app2 \
--image=httpd \
-n ingress-demo \
--dry-run=client -o yaml > k8s/app2-deploy.yaml
```

---

## Apply Deployment

```bash
kubectl apply -f k8s/app2-deploy.yaml
```

---

## Create Service YAML

```bash
kubectl expose deployment app2 \
--port=80 \
--target-port=80 \
--type=ClusterIP \
-n ingress-demo \
--dry-run=client -o yaml > k8s/app2-svc.yaml
```

---

## Apply Service

```bash
kubectl apply -f k8s/app2-svc.yaml
```

---

# Create Ingress

## ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: ingress-routing
  namespace: ingress-demo

spec:
  ingressClassName: nginx

  rules:
    - http:
        paths:
          - path: /app1
            pathType: Prefix

            backend:
              service:
                name: app1
                port:
                  number: 80

          - path: /app2
            pathType: Prefix

            backend:
              service:
                name: app2
                port:
                  number: 80
```

---

## Apply Ingress

```bash
kubectl apply -f k8s/ingress.yaml
```

---

# Verify Resources

## Check Pods

```bash
kubectl get pods -n ingress-demo
```

---

## Check Services

```bash
kubectl get svc -n ingress-demo
```

---

## Check Ingress

```bash
kubectl get ingress -n ingress-demo
```

---

## Check Ingress Controller

```bash
kubectl get pods -n ingress-nginx
```

---

# Test In Browser

## App 1

```text
http://<EC2-PUBLIC-IP>/app1
```

---

## App 2

```text
http://<EC2-PUBLIC-IP>/app2
```

---

# Verify KIND Port Mapping

```bash
docker ps
```

---

# Verify Security Group

Allow:

```text
Port 80
Port 443
```
