# k8s 
   # 🚀 NFS Server with Kubernetes Sidecar Pod Demo
   
## 📌 Overview
This demo shows how to:
1. Configure and run an **NFS server** on Ubuntu.  
2. Mount the NFS volume inside a **Kubernetes Pod**.  
3. Use a **Sidecar Container** to log continuously into a shared volume.  

---

## 🖥️ Prepare the NFS Server

Run the following commands on your Ubuntu host:

```bash
sudo apt update
sudo apt install nfs-kernel-server -y

# Create shared directory
sudo mkdir -p /mnt/shared
sudo chown nobody:nogroup /mnt/shared
sudo chmod 777 /mnt/shared

# Configure exports
sudo vim /etc/exports
```

Add the following line to `/etc/exports`:
```
/mnt/shared *(rw,no_root_squash,insecure,sync,no_subtree_check)
```

Start and enable the NFS service:
```bash
sudo exportfs -a
sudo systemctl start nfs-kernel-server
sudo systemctl enable nfs-kernel-server
```

---

## 📦 Pod Manifest with Sidecar

Save the following as `sidecar-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-demo
spec:
  volumes:
    - name: emptydir-volume
      emptyDir: {}
    - name: nfs-volume
      nfs:
        server: 192.168.2.138 # Change this to your NFS server IP
        path: /mnt/shared
  containers:
    - name: main-container
      image: nginx:alpine
      resources:
        requests:
          memory: "64Mi"
          cpu: "250m"
        limits:
          memory: "128Mi"
          cpu: "500m"
      volumeMounts:
        - name: emptydir-volume
          mountPath: /usr/share/nginx/html
        - name: nfs-volume
          mountPath: /mnt/nfs
      ports:
        - containerPort: 80
      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 30
        periodSeconds: 10
      readinessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 30
        periodSeconds: 10

    - name: sidecar-container
      image: busybox
      command: ["/bin/sh", "-c", "while true; do echo $(date) ' - Sidecar logging'; sleep 5; done"]
      resources:
        requests:
          memory: "16Mi"
          cpu: "100m"
        limits:
          memory: "32Mi"
          cpu: "200m"
      volumeMounts:
        - name: emptydir-volume
          mountPath: /mnt/shared
```

---

## ▶️ Deploy the Pod

```bash
kubectl apply -f sidecar-pod.yaml
```

Check the pod status:
```bash
kubectl get pods
```

You should see:
```
NAME            READY   STATUS    RESTARTS   AGE
sidecar-demo    2/2     Running   0          1m
```

---

## 🔍 Tests

- **Login to the main container:**
  ```bash
  kubectl exec -it sidecar-demo -c main-container -- sh
  ```

- **Check logs from the sidecar:**
  ```bash
  kubectl logs -f sidecar-demo -c sidecar-container
  ```

You should see timestamped log entries like:
```
Mon Sep 23 12:00:01 UTC 2025  - Sidecar logging
Mon Sep 23 12:00:06 UTC 2025  - Sidecar logging
```

---

## ✅ Summary
- NFS server is prepared and shared at `/mnt/shared`.  
- Kubernetes Pod runs with **nginx (main-container)** and **busybox (sidecar)**.  
- Sidecar continuously writes logs into the shared volume.  
