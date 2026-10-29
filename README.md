# Secure Kubernetes Application with AWS ALB, ACM & HTTPS

## Project Overview

This project demonstrates how to expose an insurance web application running on a KOPS-managed Kubernetes cluster through an AWS Application Load Balancer (ALB).

AWS Certificate Manager (ACM) provides the TLS certificate used by the ALB listener to serve HTTPS traffic. Amazon Route 53 maps the application domain to the load balancer.

## Architecture

User / Browser  
↓  
Route 53 DNS  
↓  
AWS Application Load Balancer  
↓ HTTPS listener with ACM certificate  
↓ HTTP-to-HTTPS redirect on port 80  
↓  
Kubernetes NodePort Service  
↓  
Insurance Deployment (3 replicas)

## Technologies Used

- Kubernetes
- KOPS
- AWS Application Load Balancer Controller
- AWS Certificate Manager (ACM)
- Amazon Route 53
- Helm
- Docker container image
- Kubernetes Ingress, Deployment and Service
- Liveness, Readiness and Startup Probes

## Features

- Three application replicas
- Kubernetes health checks
- Host-based traffic routing
- AWS-managed TLS certificate
- HTTPS access through the ALB
- HTTP-to-HTTPS redirection
- DNS integration
- Load balancer health checks

## Prerequisites

- A working KOPS-managed Kubernetes cluster
- kubectl and Helm installed
- AWS CLI configured with an appropriately scoped IAM identity
- AWS Load Balancer Controller installed and configured
- An ACM certificate in the same AWS Region as the ALB
- A domain name and access to its DNS configuration
- Correct subnet discovery tags and network permissions

## Repository Structure

```text
.
├── README.md
├── manifests/
│   ├── application-deploy-service.yml
│   └── ingress.yml
├── docs/
│   └── architecture.md
└── screenshots/
```

## Deployment

### 1. Verify the Kubernetes cluster

```bash
kubectl get nodes
kubectl get pods -A
```

### 2. Deploy the application and Service

```bash
kubectl apply -f manifests/application-deploy-service.yml
kubectl rollout status deployment/insurance-deployment
kubectl get pods -o wide
kubectl get service insurance-service
```

### 3. Apply the Ingress

Update the ACM certificate ARN, hostname and any environment-specific settings in `manifests/ingress.yml`.

```bash
kubectl apply -f manifests/ingress.yml
kubectl get ingress insurance-ingress
kubectl describe ingress insurance-ingress
```

### 4. Configure DNS

After the ALB has been provisioned, create the appropriate DNS record pointing the application hostname to the ALB DNS name.

### 5. Test the application

```bash
curl -I http://YOUR_DOMAIN
curl -I https://YOUR_DOMAIN
kubectl get pods
kubectl get endpoints insurance-service
```

Confirm that HTTP redirects to HTTPS and that the HTTPS endpoint presents a valid certificate for the hostname.

## Troubleshooting

- Check pod status and logs.
- Inspect the Ingress events.
- Verify ALB target health.
- Check IAM permissions and controller logs.
- Confirm subnet tags, security groups and NodePort reachability.
- Verify that the ACM certificate covers the hostname and is in the correct Region.
- Confirm that DNS resolves to the intended ALB.

## Cleanup

Remove the Ingress first so its controller can clean up the associated AWS load balancer.

```bash
kubectl delete -f manifests/ingress.yml
kubectl delete -f manifests/application-deploy-service.yml
```

Check AWS resources and billing after cleanup. Do not delete shared cluster resources or certificates used by other applications.

## Disclaimer

This is a learning project. Review IAM permissions, network exposure, availability, security and cost before adapting it for production.
