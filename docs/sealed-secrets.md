# Sealed Secrets
Sealed Secrets is a Kubernetes tool that enhances security by enabling the encryption of Kubernetes Secrets at rest in a Git repository. It allows users to commit encrypted versions of their Secrets to source control while still maintaining the ability to decrypt and use them within the Kubernetes cluster. Sealed Secrets employs public-key cryptography, where the cluster holds the public key, enabling it to encrypt Secrets, while a client tool called "kubeseal" uses a private key to decrypt the Secrets for use within the cluster. This approach ensures that sensitive information, such as passwords or API keys, remains secure even when stored in version control.

## Helm Repository
Let's create a HelmRepository for sealed-secrets
Directory structure
```
clusters
--production
----flux-system
----kustomization.yaml
------helmrepositories
--------sealed-secrets.yaml
```

The content of sealed-secrets.yaml
```
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: sealed-secrets
  namespace: flux-system
spec:
  interval: 1h0m0s
  url: https://bitnami-labs.github.io/sealed-secrets
```
The content of kustomization.yaml
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - gotk-components.yaml
  - gotk-sync.yaml
  - helmrepositories/sealed-secrets.yaml
```

## Creating the namespace
Lets create a namespace for sealed-secrets

Directory structure
```
clusters
---production
-----sealed-secrets
-------namespace.yaml
```
Content of namespace.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: sealed-secrets
```
Note, we do not want to inject istio sidecar for this namespace. 

## Creating the helmrelease

Let's deploy sealed-secrets using a helmrelease CRD
Directory structure
```
clusters
--production
----sealed-secrets
------namespace.yaml
------sealed-secrets.yaml
```

Content of sealed-secrets.yaml
```
apiVersion: helm.toolkit.fluxcd.io/v2beta2
kind: HelmRelease
metadata:
  name: sealed-secrets
  namespace: flux-system
spec:
  chart:
    spec:
      chart: sealed-secrets
      sourceRef:
        kind: HelmRepository
        name: sealed-secrets
      version: ">=1.15.0-0"
  interval: 1h0m0s
  releaseName: sealed-secrets-controller
  targetNamespace: sealed-secrets
  install:
    crds: Create
  upgrade:
    crds: CreateReplace
```


