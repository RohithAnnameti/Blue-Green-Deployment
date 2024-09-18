# Blue-Green-Deployment

To route traffic across two namespaces using a
single Azure Application Gateway
with
NGINX Ingress Controller
in an
Azure Kubernetes Service (AKS)
cluster, the setup involves configuring the
NGINX Ingress Controller
in each namespace and defining routing rules within the single Application Gateway to properly direct traffic to the correct namespace and services.
Here's how you can achieve this setup:
Overview:
• Namespaces: Two namespaces (namespace1 and namespace2) contain different versions of your application.
• Ingress Resources: Each namespace has its own NGINX Ingress Controller and corresponding Ingress resources to handle routing within the namespace.
• Application Gateway: A single Azure Application Gateway will serve as the frontend for external traffic and route it based on rules that direct requests to the appropriate NGINX Ingress in each namespace.

Step-by-Step Guide:
1. Set Up NGINX Ingress Controller in Each Namespace
You will need to deploy a separate
NGINX Ingress Controller
for each namespace. This way, each namespace can manage its own ingress traffic while the Application Gateway handles the external traffic.
Install NGINX Ingress Controller in namespace1:
bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml --namespace=namespace1

Install NGINX Ingress Controller in namespace2:
bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml --namespace=namespace2


Make sure that the controllers are deployed with unique service annotations so they don’t conflict with each other. You can also assign them different external IPs if necessary.
2. Create Services and Deploy Applications in Both Namespaces Deploy your application and services in each namespace (namespace1 and namespace2).

Example for namespace1:

apiVersion: apps/v1
kind: Deployment
metadata:
 name: app-deployment
 namespace: namespace1
spec:
 replicas: 3
 selector:
   matchLabels:
     app: app-v1
 template:
   metadata:
     labels:
       app: app-v1
   spec:
     containers:
     - name: app-container
       image: your-image:v1
       ports:
       - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
 name: app-service
 namespace: namespace1
spec:
 selector:
   app: app-v1
 ports:
 - protocol: TCP
   port: 80
   targetPort: 80
 type: ClusterIP



