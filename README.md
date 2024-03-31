# Crafting a Cloud-Agnostic Infrastructure on Kubernetes for a Startup Company

As a startup ourselves, we're excited to share our tech stack built on Kubernetes. From initial setup to ongoing optimizations, we've documented our journey to provide insights for fellow startups. We welcome your feedback and contributions as we continue to refine our infrastructure. 

## Motivation

As a startup, managing cloud costs constitutes a significant aspect of our financial planning. Recognizing the importance of this, major cloud providers offer generous cloud credits to help startups like ours initiate operations. Currently, we are leveraging Azure's cloud services and are grateful for the $5,000 credit valid for one year. We extend our gratitude to Azure for their support. While our intention is to remain with Azure, we understand the value of establishing a cloud-agnostic platform to enhance flexibility and mitigate dependency on any single provider.

## Core Components
![Alt text](images/ex-offenders-platform.png?raw=true "Title")

1. Deploying Kubernetes Cluster on Azure
2. Deploying Istio - Our service mesh
3. Deploying Monitoring Tools
5. Configuring FluxCD - Our GitOps tool
6. CircleCI
7. Creating HelmRepositories
8. Configuring Sealed-Secrets
9. Configuring Cert-Manager
10. Configuring Istio Ingressgateway with Let's Encrypt
11. Keycloak, Istio and Let's Encrypt Certificate
12. MinIO Deployment - Our object storage
