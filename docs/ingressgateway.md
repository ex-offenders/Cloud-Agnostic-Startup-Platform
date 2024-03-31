# Configuring Istio IngressGatway with Let's Encrypt Certificate

Now let's jump into the interesting bit. Let's create a sample nginx deployment and expose it through Istio Ingressgateway over TLS. 

## DNS changes

Determine the public IP address of istio-ingressgateway load balancer

```
kubectl get service istio-ingressgateway -n istio-system
```

```
NAME                   TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                                      AGE
istio-ingressgateway   LoadBalancer   10.0.41.165   4.250.87.120   15021:32693/TCP,80:32134/TCP,443:32131/TCP   8d
```

Point the domain to the external IP address. In our case, we have pointed both ex-offenders.co.uk and www.ex-offenders.co.uk to 4.250.87.120. 

## Create a test deployment
In our case, we have exposed our frontend service through the ingressgateway. But for simplicity, let's expose an nginx deployment through the ingressgateway. 

Directory structure
```
clusters
--production
----frontend
------deployment.yaml
------service.yaml
------namespace.yaml
```
Content of namespace.yaml

```
apiVersion: v1
kind: Namespace
metadata:
  name: frontend
  labels:
    istio-injection: enabled
```

Note that, we have enabled istio-injection in this namespace as the traffic in and out of this namespace matter to us. 

Content of deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
  namespace: keycloak
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

Content of service.yaml
```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: nginx
  namespace: keycloak
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: ClusterIP
```
