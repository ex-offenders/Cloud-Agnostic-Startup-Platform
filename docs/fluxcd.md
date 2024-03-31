# FluxCD Installation

[FluxCD](https://fluxcd.io/) is a continuous delivery tool that automates the deployment and lifecycle management of applications on Kubernetes. It uses GitOps principles to synchronize application code stored in Git repositories with Kubernetes clusters, ensuring consistency and reliability in the deployment process. 

# Installation Steps

1. Install Flux CLI

```
curl -s https://fluxcd.io/install.sh | sudo bash
```
2. Create a GitHub Personal Access Token
Flux bootstrap command needs a PAT to access GitHub API. 
Click on profile -> Settings -> Developer Settings and create a Classic Personal Access Token.
And then export the token as an environment variable
```
export GITHUB_TOKEN=<gh-token>
```
3. Bootstrap Flux

```
flux bootstrap github \
  --token-auth \
  --owner=ex-offenders \
  --repository=paas-config \
  --branch=main \
  --path=clusters/production \
  --personal
```
With this, paas-config GH repository in ex-offenders organization will be initialized. 

After this is done, you would be able to see the following files in the new repository

```
clusters
-- production
----flux-system
------gotk-components.yaml
------gotk-sync.yaml
------kustomization.yaml
```

Now we can start deploying resources into the Kubernetes cluster by pushing the changes to this github repository. This will be covered in later sections. 
