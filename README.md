# Guestbook Application with Docker, Kubernetes, and OpenShift

This repository contains the Guestbook application project, created as part of the "Introduction to Containers with Docker, Kubernetes & OpenShift" course on Coursera. The project demonstrates containerization, deployment, scaling, rolling updates/rollbacks, and OpenShift-specific capabilities using Docker, Kubernetes, IBM Cloud Container Registry, and RedHat OpenShift.

---

# Table of Contents
- [Project Overview](#project-overview)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Implementation](#implementation)
  - [Building the Guestbook App Docker Image and Pushing to IBM Cloud Container Registry](#1-building-the-guestbook-app-docker-image-and-pushing-to-ibm-cloud-container-registry)
  - [Applying Deployment and Accessing Application By Port Forwarding](#2-applying-deployment-and-accessing-application-by-port-forwarding)
  - [Autoscaling using Horizontal Pod Autoscaler (HPA)](#3-autoscaling-using-horizontal-pod-autoscaler-hpa)
  - [Performing Rolling Updates and Rollbacks on Guestbook Application](#4-performing-rolling-updates-and-rollbacks-on-guestbook-application)
  - [Deploying the Application to OpenShift](#5-deploying-the-application-to-openshift)
  - [Using Redis Master for Storage](#6-using-redis-master-for-storage)
- [Course Certificate](#course-certificate)

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

- **Exporting namespace as env var, building image, pusing image, verifying image push.**
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

- **Autoscale the Guestbook deployment.**
   ```bash
   kubectl autoscale deployment guestbook --cpu-percent=5 --min=1 --max=10
   ```

- **Status of newly made HPA.**
   ```bash
   kubectl get hpa guestbook
   ```

   ![New HPA](/images/hpa.png)


- **Generating load on the app on the server.**
   ```bash
   kubectl run -i --tty load-generator --rm --image=busybox:1.36.0 --restart=Never /bin/sh -c "while sleep 0.01; do wget -q -O- <app URL>; done"
   ```

- **Observing increasing in pod replicas due to increase in load.**
   ```bash
   kubectl get hpa guestbook --watch
   ```

   ![Increase in Pod Replicas by Autoscaling after Increasing Load on Server](/images/hpa2.png)

### 4. Performing Rolling Updates and Rollbacks on Guestbook Application

- **Changing index.html, building and pushing updated app image.**
  ```bash
  docker build . -t us.icr.io/$MY_NAMESPACE/guestbook:v1 && docker push us.icr.io/$MY_NAMESPACE/guestbook:v1
  ```

  ![Pushing Updated App Image](/images/upguestbook.png)

- **Changing resource requests and limits in the deployments.yml from 50 and 20 milli-cores to 5 and 2 milli-cores repectively. Applying the deployment.**

  ![Updated Deployment](/images/deployment.png)

- **Port forward again and get Replica Sets.**

  ![Get replica sets after updating deployment](/images/rs_before.png)

  We see that a new replica set is created and is running. The previous one is stopped.

- **Relaunching Application**

  ![Relaunch Application](/images/up-app.png)

  We see v2 in Application Title. This means rollout was successful.

- **Checking details of Revision of the deployment rollout.**
  ```bash
  kubectl rollout history deployments guestbook --revision=2
  ```
  
  ![Details of Revision of Deployment Rollout](/images/rev.png)

- **Undo the Deloyment and Set to Revision 1, Re-check which replica sets are running post rollback.**
  ```bash
  kubectl rollout undo deployment/guestbook --to-revision=1
  ```

  ![Replica Sets post rollback](/images/rs.png)

  We see that new replica set is stopped and the original one is running again


### 5. Deploying the Application to OpenShift
- **Image Stream pointing to my image in IBM Cloud Container Registry.**
   ```bash
   oc tag us.icr.io/$MY_NAMESPACE/guestbook:v1 guestbook:v1 --reference-policy=local --scheduled
   ```

   With the --reference-policy=local option, a copy of the image from IBM Cloud Container Registry is imported into the local cache of the internal registry and made available to the cluster’s projects as an image stream. The --scheduled option sets up periodic importing of the image from IBM Cloud Container Registry into the internal registry. The default frequency is 15 minutes.

- **Launching OpenShift Web Console, Adding new application using Container Image in Developer Perspective, Viewing in Topology View, Using Route Link from Topology View to Launch App.**

  Topology View:

  ![Topology View](/images/og_topology.png)

  Application:
  
  Forgot to take a photo of the launched application :(

- **Updaing index.html, Building and Pushing App using same image tag, and Updating Image Stream**
  ```bash
  docker build . -t us.icr.io/$MY_NAMESPACE/guestbook:v1 && docker push us.icr.io/$MY_NAMESPACE/guestbook:v1
  ```

  Either wait for 15 minutes for automatic scheduled image import (--scheduled flag used earlier) or run following command to update the image.
  ```bash
  oc import-image guestbook:v1 --from=us.icr.io/$MY_NAMESPACE/guestbook:v1 --confirm
  ```

  ![OpenShift Image import](/images/openshift_image_import.png)

- **Viewing Image Streams in Adminstrator Perspective.**

  ![OpenShift Image Streams](/images/openshift_image_streams.png)

  We see 2 entries. This means OpenShift imported our new image

- **Relaunching Application using Route Link in Topology View in Developer Perspective**

  ![OpenShift Application Updated Version](/images/app_openshift.png)

### 6. Using Redis Master for Storage

- **Clicking on /info (an information endpoint for the guestbook) on the app web page**

  ![OpenShift In-memory datastore (not redis)](/images/not_redis.png)

  Currently, we have only deployed the guestbook web front end, so it is using in-memory datastore to keep track of the entries. This is not very resilient, however, because any update or even a restart of the Pod will cause the entries to be lost.

- **Deleting application from Topology View, Creating Redis master Deployment, Verifying Deployments and Pods creation, Creating Redis master Service.**
  ```bash
  oc apply -f redis-master-deployment.yaml
  oc get deployments
  oc get pods
  ```

  ![Redis master Deployment and Pods](/images/redis_master_deployment_pods.png)

  ```bash
  oc apply -f redis-master-service.yaml
  ```

  Services find the Pods to load balance based on Pod labels. The Pod created in previous step has the labels app=redis and role=master. The selector field of the Service determines which Pods will receive the traffic sent to the Service.

- **Creating Redis slave Deployment, Verifying Deployments and Pods were created, Creating Redis slave Service.**
  ```bash
  oc apply -f redis-slave-deployment.yaml
  oc get deployments
  oc get pods
  ```

  ![Redis slave Deployment and Pods](/images/redis_slave_deployment_pods.png)

  ```bash
  oc apply -f redis-slave-service.yaml
  ```

- **Creating Redis slave Deployment, Verifying Deployments and Pods were created, Creating Redis slave Service.**
  ```bash
  oc apply -f redis-slave-deployment.yaml
  oc get deployments
  oc get pods
  ```

  ![Redis slave Deployment and Pods](/images/redis_slave_deployment_pods.png)

  ```bash
  oc apply -f redis-slave-service.yaml
  ```

- **Adding second version of guestbook app using this GitHub repo, Viewing guestbook deployment in Topology View.**

  ![OpenShift Final Deployment](/images/openshift_final_deployment.png)

- **Viewing Final Application using Route Link**

  ![OpenShift Application Version 2](/images/openshift_app_v2.png)

- **Information Endpoint**

  ![OpenShift Application Version 2 information endpoint](/images/openshift_app_v2_info_endpoint.png)

  Now we get information on Redis instead of "In-memory datastore (not redis)" since we're no longer using the in-memory datastore.
  
---

## Course Certificate

You can view the course certificate [here](https://github.com/KunalSachdev2005/Containerized_Guestbook_Application_with_Docker_Kubernetes_OpenShift/blob/main/Introduction_to_Containers_with_Docker_Kubernetes_%26_OpenShift_5U3I6K29W297.pdf)
