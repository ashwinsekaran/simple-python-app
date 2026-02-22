# Simple Python App – Docker, ECR, and EKS Deployment Guide

This document explains step-by-step how to:

1. Authenticate Docker with AWS ECR  
2. Build and tag a Docker image  
3. Push the image to Amazon ECR  
4. Configure kubectl for EKS  
5. Deploy the application to Kubernetes  
6. Expose the application using a LoadBalancer service  
7. Resolve scheduling issues in small node groups  

---

## 1️⃣ Authenticate Docker with AWS ECR

Before pushing images to Amazon Elastic Container Registry (ECR), Docker must be authenticated.

```bash
aws ecr get-login-password --region eu-north-1 \
| docker login \
--username AWS \
--password-stdin 350732689609.dkr.ecr.eu-north-1.amazonaws.com/simple-python-app
```

### What this does:
- Retrieves a temporary authentication token from AWS.
- Logs Docker into your private ECR repository.
- Allows you to securely push Docker images to ECR.

---

## 2️⃣ Build the Docker Image

```bash
docker build -t simple-python-app:1.0 .
```

### What this does:
- Builds a Docker image using the Dockerfile in the current directory.
- Tags the image as `simple-python-app:1.0`.
- The `.` indicates the current directory as the build context.

---

## 3️⃣ Tag the Image for ECR

```bash
docker tag simple-python-app:1.0 \
350732689609.dkr.ecr.eu-north-1.amazonaws.com/simple-python-app:1.0
```

### What this does:
- Creates a new tag for the image.
- Associates the local image with your ECR repository URI.
- This step is required before pushing to ECR.

---

## 4️⃣ Push the Image to ECR

```bash
docker push \
350732689609.dkr.ecr.eu-north-1.amazonaws.com/simple-python-app:1.0
```

### What this does:
- Uploads the Docker image layers to Amazon ECR.
- Makes the image available for your EKS cluster to pull and deploy.

---

## 5️⃣ Configure kubectl for EKS Cluster

```bash
aws eks update-kubeconfig --name platform-learning-eks
```

### What this does:
- Updates your local kubeconfig file.
- Adds cluster endpoint and authentication details.
- Enables `kubectl` to communicate with your EKS cluster.

---

## 6️⃣ Deploy Kubernetes Resources

Navigate to your Kubernetes manifests directory:

```bash
cd k8s-manifest
```

### Apply Deployment

```bash
kubectl apply -f deployment.yaml
```

This creates a Kubernetes Deployment that:
- Pulls the Docker image from ECR.
- Creates and manages Pods.
- Maintains the desired replica count.
- Automatically restarts failed containers.

---

### Apply Service

```bash
kubectl apply -f service.yaml
```

This creates a Kubernetes Service (LoadBalancer type) that:
- Exposes your application externally.
- Automatically provisions an AWS Elastic Load Balancer.
- Routes external traffic to your application Pods.

---

## 7️⃣ Scale CoreDNS (If Pods Are Stuck in Pending State)

If you're using a small instance type (like `t3.micro`), you may hit the maximum pod limit.

You might see an error like:

```
0/1 nodes are available: 1 Too many pods
```

To free up capacity:

```bash
kubectl scale deployment coredns -n kube-system --replicas=1
```

### Why this works:
- Small EC2 instances have a low maximum pod limit (e.g., 4 pods).
- By default, EKS runs 2 CoreDNS pods.
- Reducing CoreDNS replicas frees one pod slot.
- Your application pod can then be scheduled successfully.

---

## 8️⃣ Verify Deployment

Check pods:

```bash
kubectl get pods
```

Check all namespaces:

```bash
kubectl get pods -A
```

Check services:

```bash
kubectl get svc
```

Once:
- Pod status = `Running`
- Service has an `EXTERNAL-IP`

Open the LoadBalancer URL in your browser to access your application.

---
In case of any issues,

kubectl edit configmap aws-auth -n kube-system

Add this under maproles:
```  
  - rolearn: arn:aws:iam::350732689609:role/github-actions-eks-deployer
    username: github-actions
    groups:
      - system:masters 
 ```

# ✅ Deployment Complete

Your Python application is now:

- Built using Docker  
- Stored in Amazon ECR  
- Deployed to Amazon EKS  
- Exposed via AWS LoadBalancer  

You now have a complete containerized deployment workflow 🚀


[//]: # (aws ecr get-login-password --region eu-north-1 \                                                                                                                                   INT ✘  21:48:28 )

[//]: # (| docker login \)

[//]: # (--username AWS \)

[//]: # (--password-stdin 350732689609.dkr.ecr.eu-north-1.amazonaws.com/simple-python-app)

[//]: # ()
[//]: # ()
[//]: # (docker build -t simple-python-app:1.0 .   )

[//]: # ()
[//]: # ()
[//]: # (docker tag simple-python-app:1.0 \                                                                                                                                                            INT ✘  21:51:54 )

[//]: # (350732689609.dkr.ecr.eu-north-1.amazonaws.com/simple-python-app)

[//]: # ()
[//]: # ()
[//]: # (docker push \                                                                                                                                                                                     ✔  21:52:24 )

[//]: # (350732689609.dkr.ecr.eu-north-1.amazonaws.com/simple-python-app)

[//]: # ()
[//]: # ()
[//]: # (aws eks update-kubeconfig --name platform-learning-eks )

[//]: # ()
[//]: # (cd k8s-manifest)

[//]: # (kubectl apply -f deployment.yaml)

[//]: # (kubectl apply -f service.yaml)

[//]: # ()
[//]: # (kubectl scale deployment coredns -n kube-system --replicas=1 )

[//]: # ()
[//]: # (kubectl get pods)