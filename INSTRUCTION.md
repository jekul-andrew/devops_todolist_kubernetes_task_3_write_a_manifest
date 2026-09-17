# Kubernetes ToDo App Instructions

## 1. Overview

This project runs a Django ToDo application inside a Kubernetes cluster.

The setup uses:

- Django application
- Docker image: `zhekul/todoapp:3.0.0`
- Kubernetes Pod for the ToDo application
- Kubernetes Pod with `busyboxplus:curl` for internal network testing
- Readiness probe
- Liveness probe
- Kubernetes namespace: `todoapp`

All Kubernetes manifests are located in the `.infrastructure` directory.

---

## 2. Apply Kubernetes Manifests

First, create the namespace:

```bash
kubectl apply -f .infrastructure/namespace.yml
```

Then create the ToDo application pod:

```bash
kubectl apply -f .infrastructure/todoapp-pod.yml
```

Check the pod status:

```bash
kubectl get pods -n todoapp
```

Wait until the `todoapp` pod is running and ready.

Then create the BusyBox pod:

```bash
kubectl apply -f .infrastructure/busybox.yml
```

Check all pods:

```bash
kubectl get pods -n todoapp
```

Expected result:

```text
NAME      READY   STATUS    RESTARTS   AGE
busybox   1/1     Running   0          ...
todoapp   1/1     Running   0          ...
```

---

## 3. Test the Application with Port Forwarding

Forward local port `8081` to port `8080` of the `todoapp` pod:

```bash
kubectl port-forward pod/todoapp 8081:8080 -n todoapp
```

Then open the application in a browser:

```text
http://localhost:8081
```

The API can be opened at:

```text
http://localhost:8081/api/
```

Readiness endpoint:

```text
http://localhost:8081/api/readiness/
```

Liveness endpoint:

```text
http://localhost:8081/api/liveness/
```

---

## 4. Test the Application from BusyBox with curl

First, get the pod IP addresses:

```bash
kubectl get pods -n todoapp -o wide
```

Find the IP address of the `todoapp` pod.

Example:

```text
todoapp   1/1   Running   0   ...   10.244.0.8
```

Enter the BusyBox pod:

```bash
kubectl exec -it -n todoapp busybox -- sh
```

Inside the BusyBox container, use the ToDo pod IP address to test the application.

For example:

```bash
curl http://10.244.0.8:8080/api/readiness/
curl http://10.244.0.8:8080/api/liveness/
curl http://10.244.0.8:8080
```

The pod IP may change after the pod is recreated, so always check the current IP address with:

```bash
kubectl get pods -n todoapp -o wide
```
