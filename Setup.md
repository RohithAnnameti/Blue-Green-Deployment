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
-------------------------------------------------------------------------------------
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


Repeat the same for namespace2, ensuring that the services and deployments are separate for versioning (e.g., app-v2)

3. Create Ingress Resources for Each Namespace
Next, define the Ingress resources in each namespace. This is where the routing between namespaces is managed.
Ingress for namespace1:
yaml

apiVersion:
networking.k8s.io/v1
kind: Ingress
metadata:
 name: app-ingress
 namespace: namespace1
 annotations:
nginx.ingress.kubernetes.io/rewrite-target:
/
spec:
 rules:
 - host:
app.example.com
   http:
     paths:
     - path: /
       pathType: Prefix
       backend:
         service:
           name: app-service
           port:
             number: 80


Ingress for namespace2:
yaml

apiVersion:
networking.k8s.io/v1
kind: Ingress
metadata:
 name: app-ingress
 namespace: namespace2
 annotations:
nginx.ingress.kubernetes.io/rewrite-target:
/
spec:
 rules:
 - host:
app.example.com
   http:
     paths:
     - path: /v2
       pathType: Prefix
       backend:
         service:
           name: app-service
           port:
             number: 80


4. Configure the Application Gateway
Now, set up routing on the Application Gateway to direct traffic based on hostnames or paths to the NGINX Ingress Controllers in each namespace.
Option 1: Host-Based Routing
If you want to route traffic based on different hostnames (e.g., app-v1.example.com and app-v2.example.com), configure the Application Gateway with host-based routing rules.
1 For namespace1, route traffic to the NGINX Ingress Controller of namespace1 based on the hostname app-v1.example.com.
2 For namespace2, route traffic to the NGINX Ingress Controller of namespace2 based on the hostname app-v2.example.com.
In the Application Gateway’s settings:
• Rule for app-v1.example.com : Forward traffic to the namespace1 NGINX Ingress IP.
• Rule for app-v2.example.com : Forward traffic to the namespace2 NGINX Ingress IP.
Option 2: Path-Based Routing
If you want to route traffic based on different paths (e.g., /v1 and /v2), configure path-based routing on the Application Gateway.
1 For namespace1, route traffic to the NGINX Ingress Controller of namespace1 for the path /v1.
2 For namespace2, route traffic to the NGINX Ingress Controller of namespace2 for the path /v2.
In the Application Gateway’s settings:
• Rule for /v1 path : Forward traffic to the NGINX Ingress IP of namespace1.
• Rule for /v2 path : Forward traffic to the NGINX Ingress IP of namespace2.
5. Verify the Setup
Once the Application Gateway is configured, verify that traffic is routed correctly:
• For namespace1 : Access http://app.example.com/v1, and it should route traffic to namespace1.
• For namespace2 : Access http://app.example.com/v2, and it should route traffic to namespace2.
Summary:
• NGINX Ingress Controllers: Deploy an NGINX Ingress Controller in each namespace.
• Ingress Resources: Define separate Ingress resources for each namespace with different routing rules (e.g., paths or hosts).
• Application Gateway: Use Azure Application Gateway to route traffic between the namespaces based on host or path-based rules.
This setup allows you to use a single Application Gateway for routing traffic across multiple namespaces while leveraging NGINX Ingress for application-level routing.
