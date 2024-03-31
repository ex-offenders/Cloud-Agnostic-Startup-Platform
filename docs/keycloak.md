# Keycloak Deployment

Keycloak is an open-source identity and access management solution that enables secure authentication, authorization, and single sign-on for web applications and services. This document describes how we have deployed keycloak on kubernetes cluster and how we have exposed it via Istio ingressgateway. This may have overlooked some of the best practices during this implementation. We highly appreciate your feedback on this.

## Create namespaces and SAs

As usual, Let's create two namespaces. One for the keycloak database, and the other one for the keycloak deployment. We thought of deploying the database in a separate namespace so that we can better visualize the traffic. 

Directory structure
```
clusters
--production
----keycloak
------namespace.yaml
------service-account.yaml
----keycloakdb
------namespace.yaml
------service-account.yaml
```

Content of keycloak/namespace.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: keycloak
  labels:
    istio-injection: enabled

```
Content of keycloak/service-account.yaml
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: keycloak
  namespace: keycloak
```

Content of keycloakdb/namespace.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: keycloakdb
  labels:
    istio-injection: enabled

```
Content of keycloakdb/service-account.yaml
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: keycloakdb
  namespace: keycloakdb
```

## Creating Secrets

We need to create secrets for the database (MySQL) root password and database keycloak user password. In addition to that, we needt o create a secret for Keycloak admin user password. Keycloak database user password needs to be created in both the namespaces as secrets cannot be shared. 

Directory structure
```
clusters
--production
----keycloak
------namespace.yaml
------service-account.yaml
------secret-enc.yaml
----keycloakdb
------namespace.yaml
------service-account.yaml
------secret-enc.yaml
```

Content of keycloak/secret-enc.yaml
```
---
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  annotations:
    sealedsecrets.bitnami.com/cluster-wide: "true"
  creationTimestamp: null
  name: keycloak
  namespace: keycloak
spec:
  encryptedData:
    admin-password: <enc>
    db-password: <enc>
  template:
    metadata:
      annotations:
        sealedsecrets.bitnami.com/cluster-wide: "true"
      creationTimestamp: null
      name: keycloak
      namespace: keycloak
```
Please note that this secret has been encrypted with sealed-secrets. Please see [this](sealed-secrets.md) for more information. 

admin-password would be the admin password of keycloak deployment. db-password is the keycloak database user password. 

Content of keycloakdb/secret-enc.yaml
```
---
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  annotations:
    sealedsecrets.bitnami.com/cluster-wide: "true"
  creationTimestamp: null
  name: keycloakdb
  namespace: keycloakdb
spec:
  encryptedData:
    mysql-password: <enc>
    mysql-root-password: <enc>
  template:
    metadata:
      annotations:
        sealedsecrets.bitnami.com/cluster-wide: "true"
      creationTimestamp: null
      name: keycloakdb
      namespace: keycloakdb
```

Where mysql-password is the keycloak mysql user password. mysql-root-passowrd is the root password of the MySQL deployment. 

## MySQL Database Deployment
We deploy MySQL using a helmrelease

Directory structure

```
clusters
--production
----keycloak
------namespace.yaml
------service-account.yaml
------secret-enc.yaml
----keycloakdb
------namespace.yaml
------service-account.yaml
------secret-enc.yaml
------database.yaml
```
Content of database.yaml
```
apiVersion: helm.toolkit.fluxcd.io/v2beta2
kind: HelmRelease
metadata:
  name: keycloakdb
  namespace: keycloakdb
spec:
  chart:
    spec:
      chart: mysql
      reconcileStrategy: ChartVersion
      sourceRef:
        kind: HelmRepository
        name: bitnami
        namespace: flux-system
      version: 10.1.0
  interval: 1m0s
  timeout: 20m
  targetNamespace: keycloakdb
  values:
    auth:
      createDatabase: true
      database: keycloakdb
      username: keycloak-user
      existingSecret: keycloakdb
    serviceAccount:
      name: keycloakdb
      create: false
    image:
      debug: true
```
Note: Ideally, we should mention the resources (limits and requests) in the manifest. 

## Keycloak Deployment

We were not able to find a helm chat which supports MySQL. Currently available charts support only PostgreSQL as the backend. Therefore we go ahead with plain Kubernetes manifest to deploy Keycloak. 

```
clusters
--production
----keycloak
------namespace.yaml
------service-account.yaml
------secret-enc.yaml
------keycloak.yaml
----keycloakdb
------namespace.yaml
------service-account.yaml
------secret-enc.yaml
------database.yaml
```

Content of keycloak.yaml
