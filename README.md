# Guestbook Application with Docker, Kubernetes, and OpenShift

This repository contains the Guestbook application project, created as part of the "Introduction to Containers with Docker, Kubernetes & OpenShift" course on Coursera. The project demonstrates containerization, deployment, scaling, rolling updates/rollbacks, and OpenShift-specific capabilities using Docker, Kubernetes, IBM Cloud Container Registry, and RedHat OpenShift.

---

## Project Overview
The Guestbook application is a simple web app with a text input form for users to submit messages. The application demonstrates:

1. Building and deploying a containerized application.
2. Managing Kubernetes deployments and pods.
3. Implementing Horizontal Pod Autoscaling.
4. Performing rolling updates and rollbacks.
5. Deploying, scaling, and managing the application using OpenShift.

---

## Features
- **Dockerized Application**: Multi-stage Dockerfile for efficient builds.
- **Kubernetes Deployment**: YAML configuration for deploying the app.
- **Horizontal Pod Autoscaling**: Auto-adjusts replicas based on CPU usage.
- **Rolling Updates**: Seamless updates to the application.
- **Rollback Capability**: Reverts deployments to previous stable versions.
- **OpenShift Deployment**: Simplified application management with OpenShift tools.

---

## Prerequisites
Ensure you have the following tools installed:

- Docker
- Kubernetes CLI (`kubectl`)
- IBM Cloud CLI (optional for IBM Cloud Container Registry)
- OpenShift CLI (`oc`)

---

## Implementation

### 1. Building the Docker Image and Pushing to IBM Cloud Container Registry
Navigate to the `v1/guestbook` directory and build the Docker image:
```bash
cd v1/guestbook
export MY_NAMESPACE=sn-labs-$USERNAME
docker build . -t us.icr.io/$MY_NAMESPACE/guestbook:v1
```

Push the image to IBM Cloud Container Registry:
```bash
docker push us.icr.io/$MY_NAMESPACE/guestbook:v1
```

![Docker Image in IBM Cloud Container Registry](path/to/confirmation-image)

### 2. Deploying the Application to Kubernetes
Update the `deployment.yml` file with your namespace:
```yaml
image: us.icr.io/<your-namespace>/guestbook:v1
```

Apply the deployment:
```bash
kubectl apply -f deployment.yml
```

Access the application by forwarding the port:
```bash
kubectl port-forward deployment.apps/guestbook 3000:3000
```

![Deployed Application on Kubernetes](path/to/kubernetes-deployment-image)

### 3. Deploying the Application to OpenShift
Login to OpenShift Cluster:
```bash
oc login --token=<your-token> --server=<your-openshift-server>
```

Create a new project:
```bash
oc new-project guestbook
```

Deploy the application:
```bash
oc new-app us.icr.io/$MY_NAMESPACE/guestbook:v1 --name=guestbook
```

Expose the application:
```bash
oc expose svc/guestbook
```

Obtain the route URL:
```bash
oc get route guestbook
```

![Deployed Application on OpenShift](path/to/openshift-deployment-image)

### 4. Autoscaling and Monitoring
**Kubernetes:**
Enable Horizontal Pod Autoscaler:
```bash
kubectl autoscale deployment guestbook --cpu-percent=5 --min=1 --max=10
```

Monitor the HPA:
```bash
kubectl get hpa guestbook --watch
```

Generate load to observe autoscaling:
```bash
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://localhost:3000; done"
```

![Kubernetes Autoscaling](path/to/kubernetes-autoscaling-image)

**OpenShift:**
Enable autoscaling:
```bash
oc autoscale deployment/guestbook --min=1 --max=10 --cpu-percent=5
```

View metrics:
```bash
oc get hpa guestbook
```

![OpenShift Autoscaling](path/to/openshift-autoscaling-image)

### 5. Rolling Updates and Rollbacks
**Kubernetes:**
1. Modify the `index.html` to include new content.
2. Rebuild and push the Docker image:
   ```bash
   docker build . -t us.icr.io/$MY_NAMESPACE/guestbook:v2
   docker push us.icr.io/$MY_NAMESPACE/guestbook:v2
   ```
3. Update the `deployment.yml` with the new image tag and apply the changes:
   ```bash
   kubectl apply -f deployment.yml
   ```

Rollback to a previous version:
```bash
kubectl rollout undo deployment/guestbook --to-revision=1
```

![Kubernetes Rollback](path/to/kubernetes-rollback-image)

**OpenShift:**
Update the application image:
```bash
oc set image deployment/guestbook guestbook=us.icr.io/$MY_NAMESPACE/guestbook:v2
```

Rollback to the previous image:
```bash
oc rollout undo deployment/guestbook
```

![OpenShift Rollback](path/to/openshift-rollback-image)

---

## ![Course Certificate](https://github.com/KunalSachdev2005/Containerized_Guestbook_Application_with_Docker_Kubernetes_OpenShift/blob/main/Introduction_to_Containers_with_Docker_Kubernetes_%26_OpenShift_5U3I6K29W297.pdf)
