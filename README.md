# 🌐 Kubernetes Ingress Example
This project demonstrates how to deploy multiple web applications on Kubernetes and expose them externally using an NGINX Ingress Controller.
It includes three services (www, admin, and api) with routing rules defined in an Ingress resource.
---

## 📁 Project Structure

├── www.yaml   # Pod and Service for the frontend (www)
├── admin.yaml # Pod and Service for the admin app
├── api.yaml # Pod and Service for the API app
└── ingress.yaml # Ingress configuration for routing traffic

---

## ⚙️ Components

### 🖥️ www (Frontend)
- **Image:** `nginx`
- **Path:** `/`
- **Accessible at:** `http://www.example.com/`

### 🔧 admin (Admin Panel)
- **Image:** `ealen/echo-server:latest`
- **Environment Variable:** `ECHO_SERVER_BASE_PATH=/admin`
- **Path:** `/admin`
- **Accessible at:** `http://www.example.com/admin`

### 🔗 api (Backend API)
- **Image:** `ealen/echo-server:latest`
- **Path:** `/`
- **Accessible at:** `http://api.example.com/`

---

## 🚀 Deployment Steps

1. **Apply the YAML files:**
   ```bash
   kubectl apply -f www.yaml
   kubectl apply -f admin.yaml
   kubectl apply -f api.yaml
   kubectl apply -f ingress.yaml
 2 Verify the resources:
 kubectl get pods
 kubectl get svc
 kubectl get ingress
 3. Add host mappings (for testing locally):
 
 127.0.0.1 www.example.com
127.0.0.1 api.example.com


   You can edit the /etc/hosts file on your local machine to add these entries
Access the applications in your browser:

http://www.example.com/

http://www.example.com/admin

http://api.example.com/
🧰 Requirements

Kubernetes cluster (e.g., Minikube, Kind, or Docker Desktop)

NGINX Ingress Controller installed

kubectl CLI configured to access your cluster

🧾 Notes

Ensure that the Ingress Controller is running before applying the Ingress manifest.

You can modify hostnames or paths as needed for your environment.





             🔒 (Not Applied Yet) Enabling TLS Termination on Ingress

              ⚠️ NOTE (IMPORTANT):
The following section explains how to enable HTTPS/TLS for the Ingress,
but it has NOT been applied in this setup yet.

Step 1: Generate a Self-Signed Certificate

   openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout tls.key -out tls.crt \
-subj "/CN=example.com/O=example.com"

Step 2: Create a Kubernetes TLS Secret


kubectl create secret tls example-tls-secret --cert=tls.crt --key=tls.key

Step 3: Update the Ingress File

Add this section under spec: in your Ingress file:

tls:
  - hosts:
    - www.example.com
    - api.example.com
    secretName: example-tls-secret


Optional HTTPS redirection:

metadata:
  annotations:
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"

Step 4: Apply the Changes
kubectl apply -f ingress.yaml

🧠 Summary

This project demonstrates:

Creating Pods and Services in Kubernetes

Configuring an NGINX Ingress Controller

Routing traffic based on host and path

(Optional) Adding TLS support for HTTPS access

🎯 Project Goal

This project was created for learning purposes — to understand how Ingress works in Kubernetes and how to manage routing between multiple services.
