# Crafting a Cloud-Agnostic Infrastructure on Kubernetes for a Startup Company

As a startup ourselves, we're excited to share our tech stack built on Kubernetes. From initial setup to ongoing optimizations, we've documented our journey to provide insights for fellow startups. We welcome your feedback and contributions as we continue to refine our infrastructure. 

## Motivation

As a startup, managing cloud costs constitutes a significant aspect of our financial planning. Recognizing the importance of this, major cloud providers offer generous cloud credits to help startups like ours initiate operations. Currently, we are leveraging Azure's cloud services and are grateful for the $5,000 credit valid for one year. We extend our gratitude to Azure for their support. While our intention is to remain with Azure, we understand the value of establishing a cloud-agnostic platform to enhance flexibility and mitigate dependency on any single provider.

## Core Components
![Alt text](images/ex-offenders-platform.png?raw=true "Title")

1. [Deploying Kubernetes Cluster on Azure](kubernetes) 
2. [Configuring FluxCD - Our GitOps tool](docs/fluxcd.md)
3. Creating HelmRepositories
4. Deploying Istio - Our service mesh
5. Deploying Monitoring Tools
6. CircleCI
7. Configuring Sealed-Secrets
8. Configuring Cert-Manager
9. Configuring Istio Ingressgateway with Let's Encrypt
10. Keycloak, Istio and Let's Encrypt Certificate
11. MinIO Deployment - Our object storage

