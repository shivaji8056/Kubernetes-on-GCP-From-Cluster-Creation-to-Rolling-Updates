# Kubernetes-on-GCP-From-Cluster-Creation-to-Rolling-Updates
This document provides a step-by-step guide on how to create a Kubernetes cluster on Google Cloud Platform (GCP), deploy a simple "Hello World" application, and perform rolling updates to update the application without downtime.

Prerequisites
A Google Cloud Platform (GCP) account with billing enabled.
The Google Cloud SDK (gcloud) installed and configured on your local machine.
The kubectl command-line tool installed on your local machine.
1. Creating a Kubernetes Cluster on GCP
We'll create a Kubernetes cluster using the GCP Console.

Navigate to Kubernetes Engine:
Open the GCP Console in your web browser.
In the navigation menu, go to "Kubernetes Engine" -> "Clusters".
Create a New Cluster:
Click the "CREATE" button.
Choose "GKE Standard" for more control over networking.
Click "CONFIGURE".
Configure Basic Cluster Settings:
Name: Enter a name for your cluster (e.g., my-hello-cluster).
Location type: Select "Zonal" for simplicity and cost-effectiveness.
Zone: Select a zone in a region (e.g., us-central1-a).
Release channel: Select "Regular channel" for a balance of stability and features.
Configure Networking:
In the left sidebar, click on "Networking".
Network: Select the VPC network you want to use (or create a new one). For this example, we'll assume you have a VPC network named k8s-network with a subnet named private-subnet.
Node subnet: Select the subnet for your nodes (e.g., private-subnet).
Enable Private Nodes: Check this box for enhanced security.
Configure Node Pool:
In the left sidebar, click on "default-pool" under "Node Pools".
Number of nodes: Set to 2 for a small cluster.
Machine type: Select e2-small for cost-effectiveness.
Create the Cluster:
Click "CREATE" at the bottom of the page.
The cluster creation process will take several minutes.

2. Connecting to Your Kubernetes Cluster
Install kubectl (if not already installed):
gcloud components install kubectl
Connect to Your Cluster:
Once the cluster is created, click the "CONNECT" button in the GCP Console. Choose "Run in Cloud Shell" or copy the command to run locally:

gcloud container clusters get-credentials <cluster-name> --zone <zone> --project <project-id>
Replace <cluster-name>, <zone>, and <project-id> with your actual values.

Verify the Connection:
kubectl get nodes
This command should display the nodes in your cluster.

3. Deploying a "Hello World" Application
Create a Deployment File (hello-app.yaml):
Create a file named hello-app.yaml with the following content:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-app
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
      - name: hello-app
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: hello-app-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: hello-app
Apply the Deployment:
kubectl apply -f hello-app.yaml
Verify the Deployment:
kubectl get deployments
kubectl get pods
kubectl get services
Wait for the EXTERNAL-IP to be assigned to the hello-app-service.

Access the Application:
Open the EXTERNAL-IP in your web browser. You should see the "Hello World" application.

4. Performing Rolling Updates
Rolling updates allow you to update your deployments without downtime.

Update the Deployment Configuration:
Edit the hello-app.yaml file and change the image version to gcr.io/google-samples/hello-app:2.0:

spec:
  containers:
  - name: hello-app
    image: gcr.io/google-samples/hello-app:2.0  # Updated image version
    ports:
    - containerPort: 8080
Apply the Updated Deployment:
kubectl apply -f hello-app.yaml
Monitor the Rollout:
kubectl rollout status deployment/hello-app
This command will show you the progress of the rolling update.

Verify the Update:
kubectl get pods -l app=hello-app
You should see new pods with the updated image version.

5. Basic kubectl Commands
Here are some basic kubectl commands that you'll use frequently:

kubectl get nodes: List the nodes in your cluster.
kubectl get pods: List the pods in your cluster.
kubectl get deployments: List the deployments in your cluster.
kubectl get services: List the services in your cluster.
kubectl describe pod <pod-name>: Get detailed information about a pod.
kubectl logs <pod-name>: View the logs for a pod.
kubectl exec -it <pod-name> -- /bin/bash: Get a shell inside a pod.
kubectl apply -f <file.yaml>: Create or update resources from a YAML file.
kubectl delete -f <file.yaml>: Delete resources from a YAML file.
kubectl scale deployment <deployment-name> --replicas=<number>: Scale a deployment.
6. Rolling Update Strategies
Kubernetes supports different deployment strategies for rolling updates:

RollingUpdate (Default): Gradually replaces old pods with new ones, minimizing downtime. This is the strategy we used in this guide. It's controlled by two parameters:
maxSurge: The maximum number of pods that can be created above the desired number of replicas during the update.
maxUnavailable: The maximum number of pods that can be unavailable during the update.
Recreate: Terminates all existing pods before creating new ones. This strategy causes downtime.
Canary Deployment: Releases a new version of the application to a small subset of users before rolling it out to the entire infrastructure. This allows you to test the new version in a real-world environment and identify any issues before they affect all users.
Blue/Green Deployment: Creates two identical environments: a "blue" environment running the current version of the application and a "green" environment running the new version. Once the new version has been tested and verified, traffic is switched from the blue environment to the green environment.
Conclusion
This document has provided a basic introduction to creating a Kubernetes cluster on GCP, deploying a simple application, and performing rolling updates. By following these steps, you can start building and deploying your own applications on Kubernetes.
