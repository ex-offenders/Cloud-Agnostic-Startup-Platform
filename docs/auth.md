# Authentication and Authorization with Istio and Keycloak

In this guide, we will explore how to leverage Istio to implement authentication and authorization using Keycloak. The goal is to simplify development, allowing developers to focus on their core tasks without worrying about authentication and authorization. We will cover this step-by-step with practical examples and working sample codes.

![Alt text](../images/auth.png?raw=true "Mesh")

## Contents

1. Introduction to Keycloak
2. Introduction to Istio
3. Introduction to FastAPI
4. Deploying the job-service Microservice without Authentication and Authorization
5. Istio Weight-Based Traffic Routing between job-service-v1 and job-service-v2
6. Implementing Authentication with Istio
7. Passing the JWT Token to Backend Services
8. Implementing Authorization with Istio
9. Checking the Ownership of an Item
10. Decoding the Token

## Introduction to Keycloak

Keycloak is an open-source identity and access management solution that offers single sign-on (SSO) capabilities, allowing users to authenticate once and access multiple applications and services with a single set of credentials. One of the features I find particularly impressive is Keycloak's ability to simplify the development process by enabling the integration of custom themes for the authentication flow, such as the login page. In this scenario, we have deployed Keycloak within the same Kubernetes cluster.

Following is the Dockerfile which we use to build a custom Keycloak image with our own theme and the event listener. 
```
FROM quay.io/keycloak/keycloak:24.0.3
COPY ./ex-offenders-theme /opt/keycloak/themes/ex-offenders-theme
COPY ./providers/create-account-custom-spi.jar /opt/keycloak/providers/create-account-custom-spi.jar
```
We use the following deployment manifest to deploy Keycloak. 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: keycloak
  namespace: keycloak
  labels:
    app: keycloak
spec:
  replicas: 1
  selector:
    matchLabels:
      app: keycloak
  template:
    metadata:
      labels:
        app: keycloak
    spec:
      serviceAccountName: keycloak
      automountServiceAccountToken: true
      containers:
        - name: keycloak
          image: eocontainerregistry.azurecr.io/keycloak:v1.8.6 # {"$imagepolicy": "flux-system:keycloak"}
          args: ["start"]
          env:
            - name: KEYCLOAK_ADMIN
              value: "admin"
            - name: KEYCLOAK_ADMIN_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: keycloak
                  key: admin-password
            - name: KC_HOSTNAME
              value: auth.ex-offenders.co.uk
            - name: KC_PROXY
              value: "edge"
            - name: KC_DB
              value: mysql
            - name: KC_DB_URL
              value: "jdbc:mysql://keycloakdb-keycloakdb-mysql.keycloakdb.svc.cluster.local:3306/keycloakdb"
            - name: KC_DB_USERNAME
              value: "keycloak-user"
            - name: jgroups.dns.query
              value: keycloak
            - name: KC_DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: keycloak
                  key: db-password
          ports:
            - name: http
              containerPort: 8080
            - name: jgroups
              containerPort: 7600
      imagePullSecrets:
        - name: acr-secret
```
Notice FluxCD `imagepolicy` reference in the manifest file. With this, we can automate the deployment whenever a new image is available in the image repository. 

# Introduction to Istio

Istio is an open-source service mesh platform designed to manage how microservices communicate and share data. It provides a variety of features to improve the observability, security, and management of microservice applications. We will soon discuss how we have configured Istio.

# Introduction to FastAPI

FastAPI is a modern Python framework that is rapidly gaining popularity. It is designed for rapid development and to maximize the developer experience. In this example, we will use two versions of a job API ([V1](https://github.com/ex-offenders/job-service-v1),  [V2](https://github.com/ex-offenders/job-service-v2),) written in FastAPI. The API utilizes the SQLModel library to interact with the backend database, combining features from both SQLAlchemy and Pydantic. 

SQLModel is developed by the same author as FastAPI.

# Deploying job-service without Authentication and Authorization

Let's start with something simpler: a working microservice without any authentication or authorization. 

Here are the deployment manifests for job-service v1 and job-service v2, both running in the "job-service" namespace. Take note of the version label in each deployment manifest. 
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: job-service
    version: v1
  name: job-service
  namespace: job-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: job-service
      version: v1
  template:
    metadata:
      labels:
        app: job-service
        version: v1
    spec:
      serviceAccountName: job-service
      automountServiceAccountToken: true
      containers:
      - image: eocontainerregistry.azurecr.io/job-service:v1.0.3 # {"$imagepolicy": "flux-system:job-service-v1"}
        name: job-service
        env:
        - name: DB_HOST
          value: "keycloakdb-keycloakdb-mysql.keycloakdb.svc.cluster.local"
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: job-service
              key: db-password
        - name: DB_PORT
          value: "3306"
        - name: DB_USER
          value: "job-service"
        - name: DB_NAME
          value: "job-service"
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
      imagePullSecrets:
      - name: acr-secret
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: job-service
    version: v2
  name: job-service-v2
  namespace: job-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: job-service
      version: v2
  template:
    metadata:
      labels:
        app: job-service
        version: v2
    spec:
      serviceAccountName: job-service
      automountServiceAccountToken: true
      containers:
      - image: eocontainerregistry.azurecr.io/job-service-v2:v1.1.3 # {"$imagepolicy": "flux-system:job-service-v2"}
        name: job-service
        env:
        - name: DB_HOST
          value: "keycloakdb-keycloakdb-mysql.keycloakdb.svc.cluster.local"
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: job-service
              key: db-password
        - name: DB_PORT
          value: "3306"
        - name: DB_USER
          value: "job-service"
        - name: DB_NAME
          value: "job-service"
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
      imagePullSecrets:
      - name: acr-secret
```

We also have a ClusterIP service with the selector "app=job-service." This configuration ensures that both job-service v1 and job-service v2 are added as endpoints of this service.

```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: job-service
    kustomize.toolkit.fluxcd.io/name: flux-system
    kustomize.toolkit.fluxcd.io/namespace: flux-system
  name: job-service
  namespace: job-service
spec:
  ports:
  - port: 80
    name: http
    protocol: TCP
    targetPort: 8080
  selector:
    app: job-service
  type: ClusterIP
```

Additionally, note that we are using the same database instance for both Keycloak and the job-service versions. However, the respective users are restricted from accessing each other's databases. This setup, despite sharing the same instance, effectively mimics a microservice architecture.

For the first step, we want to route all traffic exclusively to the v1 deployment. (If you look at job-service v1, you'll see that it has been written without any authentication or authorization in the code. We plan to implement these features using Istio in the upcoming steps.)

To achieve this, we create a virtual service and a destination rule as follows. 

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: job-service
  namespace: job-service
spec:
  hosts:
    - "www.ex-offenders.co.uk"
    - "ex-offenders.co.uk"
    - job-service.job-service.svc.cluster.local
  gateways:
    - istio-system/gateway
    - mesh
  http:
    - match:
        - uri:
            prefix: "/api/jobs"
        - uri:
            prefix: "/api/jobcategories"
      route:
        - destination:
            host: job-service.job-service.svc.cluster.local
            subset: v1
          weight: 100
        - destination:
            host: job-service.job-service.svc.cluster.local
            subset: v2
          weight: 0
```

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: job-service
  namespace: job-service
spec:
  host: job-service.job-service.svc.cluster.local
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```
Note that under the gateways section, we specify both our ingress gateway and "mesh." This is because we expect traffic from both the external gateway and other microservices within the cluster. Observe how we have directed 100% of the traffic to the v1 deployment.


Below is Istio gateway resource. It handles traffic destined for the ex-offenders.co.uk domain. Additionally, we've attached a Let's Encrypt TLS certificate to the gateway using cert-manager.
```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: gateway
  namespace: istio-system
spec:
  selector:
    istio: ingressgateway
  servers:
    - port:
        number: 443
        name: https
        protocol: HTTPS
      tls:
        mode: SIMPLE
        credentialName: ex-offenders-tls
      hosts:
      - "www.ex-offenders.co.uk"
      - "ex-offenders.co.uk"
      - "auth.ex-offenders.co.uk"
```
