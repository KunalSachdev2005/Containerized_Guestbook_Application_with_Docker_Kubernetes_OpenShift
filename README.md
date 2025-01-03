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

### 1. Building the Guestbook App Docker Image and Pushing to IBM Cloud Container Registry

- **Exporting namespace as env var, building image, pusing image, verifying image push**
   ```bash
   export MY_NAMESPACE=sn-labs-$USERNAME
   docker build . -t us.icr.io/$MY_NAMESPACE/guestbook:v1
   docker push us.icr.io/$MY_NAMESPACE/guestbook:v1
   ```

   ![Docker Image in IBM Cloud Container Registry](/images/crimages.png)

### 2. Applying Deployment and Accessing Application By Port Forwarding

```bash
kubectl apply -f deployment.yml
kubectl port-forward deployment.apps/guestbook 3000:3000
```
![Deployed Application on Kubernetes](/images/app.png)

### 3. Autoscaling using Horizontal Pod Autoscaler (HPA)

- **Autoscale the Guestbook deployment**
   ```bash
   kubectl autoscale deployment guestbook --cpu-percent=5 --min=1 --max=10
   ```

- **Status of newly made HPA**
   ```bash
   kubectl get hpa guestbook
   ```

   ![New HPA](/images/hpa.png)


- **Generating load on the app on the server**
   ```bash
   kubectl run -i --tty load-generator --rm --image=busybox:1.36.0 --restart=Never /bin/sh -c "while sleep 0.01; do wget -q -O- <app URL>; done"
   ```

- **Observing increasing in pod replicas due to increase in load**
   ```bash
   kubectl get hpa guestbook --watch
   ```

   ![Increase in Pod Replicas by Autoscaling after Increasing Load on Server](/images/hpa2.png)

### 4. Performing Rolling Updates and Rollbacks on Guestbook Application

- **Changing index.html, building and pushing updated app image**
  ```bash
  docker build . -t us.icr.io/$MY_NAMESPACE/guestbook:v1 && docker push us.icr.io/$MY_NAMESPACE/guestbook:v1
  ```

  ![Pushing Updated App Image](/images/upguestbook.png)

- **Changing resource requests and limits in the deployments.yml from 50 and 20 milli-cores to 5 and 2 milli-cores repectively. Applying the deployment**

  ![Updated Deployment](/images/deployment.png)

- **Port forward again and get Replica Sets**

  ![Get replica sets after updating deployment](/images/rs_before.png)

  We see that a new replica set is created and is running. The previous one is stopped.

- **Relaunching Application**

  ![Relaunch Application](/images/up-app.png)

  We see v2 in Application Title. This means rollout was successful.

- **Checking details of Revision of the deployment rollout**
  ```bash
  kubectl rollout history deployments guestbook --revision=2
  ```
  
  ![Details of Revision of Deployment Rollout](/images/rev.png)

- **Undo the Deloyment and Set to Revision 1, Re-check which replica sets are running post rollback**
  ```bash
  kubectl rollout undo deployment/guestbook --to-revision=1
  ```

  ![Replica Sets post rollback](/images/rs.png)

  We see that new replica set is stopped and the original one is running again


### 5. Deploying the Application to OpenShift
- **Image Stream pointing to my image in IBM Cloud Container Registry**
   ```bash
   oc tag us.icr.io/$MY_NAMESPACE/guestbook:v1 guestbook:v1 --reference-policy=local --scheduled
   ```

   With the --reference-policy=local option, a copy of the image from IBM Cloud Container Registry is imported into the local cache of the internal registry and made available to the cluster’s projects as an image stream. The --scheduled option sets up periodic importing of the image from IBM Cloud Container Registry into the internal registry. The default frequency is 15 minutes.

- **Launching OpenShift Web Console, Adding new application using Container Image in Developer Perspective, Viewing in Topology View:
```bash
oc new-project guestbook
```



---

## Course Certificate

You can view the course certificate ![here](https://github.com/KunalSachdev2005/Containerized_Guestbook_Application_with_Docker_Kubernetes_OpenShift/blob/main/Introduction_to_Containers_with_Docker_Kubernetes_%26_OpenShift_5U3I6K29W297.pdf)
