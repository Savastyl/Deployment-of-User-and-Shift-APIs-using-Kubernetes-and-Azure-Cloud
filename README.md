# Deployment of User and Shift APIs using Kubernetes and Azure Cloud

## Overview
The objective of this project is to deploy two containers that run small APIs to return users and shifts data respectively from isolated databases. The system is hosted on the Azure cloud platform and Kubernetes resources in the form of YAML files are used for deployment. The system is designed to auto scale based on high traffic during the day and low traffic during the night. The deployment must also be able to handle rolling deployments and rollbacks while restricting access to certain commands for the development team using IAM controls.

Here's the project structure that can be used for this deployment:
```
project/
├── deployment/
│   ├── user-api-deployment.yaml
│   ├── shift-api-deployment.yaml
│   ├── user-api-service.yaml
│   ├── shift-api-service.yaml
│   ├── hpa.yaml
│   └── ingress.yaml
└── helm/
    ├── charts/
    ├── templates/
    ├── values-staging.yaml
    ├── values-production.yaml
    └── Chart.yaml
```


## Technologies Used
+ Azure cloud platform
+ Kubernetes
+ YAML
+ Docker
+ NGINX

## Tasks
Create the necessary YAML files for deploying the user and shift containers in separate pods.
Implement horizontal pod autoscaling to ensure the system can handle high traffic during the day.
Configure rolling deployments and rollbacks to ensure the system can be easily updated or reverted.
Implement IAM controls to restrict access to certain commands for the development team.
Apply helm chart configuration to multiple environments such as staging and production.
Use NGINX to reduce Kubernetes latency with autoscaling instead of CPU.

## Solution
1. Deploying Containers in Separate Pods
To deploy the user and shift containers, YAML files will be used to define the Kubernetes resources required for the deployment. Each container will be deployed in a separate pod to ensure isolation. The YAML files will specify the required resources such as CPU, memory, and storage, as well as the image to be used for each container.

2. Implementing Horizontal Pod Autoscaling
To ensure the system can handle high traffic during the day, horizontal pod autoscaling will be implemented. This will allow the system to automatically increase or decrease the number of replicas of the user and shift pods based on the demand for the APIs. The autoscaling will be triggered when the average CPU reaches 70%.

3. Configuring Rolling Deployments and Rollbacks
To ensure the system can be easily updated or reverted, rolling deployments and rollbacks will be configured. Rolling deployments will allow new versions of the containers to be deployed without downtime, while rollbacks will allow the system to be reverted to the previous version if necessary.

4. Implementing IAM Controls
IAM controls will be implemented to restrict access to certain commands for the development team. This will ensure that only authorized users can perform certain operations on the Kubernetes cluster. For example, the development team will be able to deploy and rollback the system, but will not be able to perform other administrative tasks.

5. Applying Helm Chart Configuration
Helm chart configuration files will be used to apply configurations to multiple environments such as staging and production. This will ensure that the deployment is consistent across different environments and that changes can be easily managed.

6. Using NGINX to Reduce Kubernetes Latency
NGINX will be used to reduce Kubernetes latency with autoscaling instead of CPU. This will ensure that the system can handle high traffic without impacting performance. NGINX will be used to cache responses and serve them directly from memory, reducing the load on the backend APIs.

## Conclusion
The deployment of the user and shift APIs using Kubernetes and Azure cloud platform involves the use of YAML files to define the required Kubernetes resources. The system is designed to auto scale based on high traffic, and to handle rolling deployments and rollbacks. IAM controls are used to restrict access to certain commands, while Helm chart configuration is used to apply configurations to multiple environments. Finally, NGINX is used to reduce Kubernetes latency with autoscaling instead of CPU

## Here's an example project structure that can be used for this deployment:

```
project/
├── deployment/
│   ├── user-api-deployment.yaml
│   ├── shift-api-deployment.yaml
│   ├── user-api-service.yaml
│   ├── shift-api-service.yaml
│   ├── hpa.yaml
│   └── ingress.yaml
└── helm/
    ├── charts/
    ├── templates/
    ├── values-staging.yaml
    ├── values-production.yaml
    └── Chart.yaml
```


### Here are example YAML files for the user API deployment, shift API deployment, and user API service:

#### user-api-deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: user-api
  template:
    metadata:
      labels:
        app: user-api
    spec:
      containers:
      - name: user-api
        image: myuserapi:latest
        ports:
        - containerPort: 80
        env:
        - name: DATABASE_URL
          value: "mysql://user:password@user-db:3306/user-db"
        resources:
          limits:
            cpu: "1"
            memory: "512Mi"
          requests:
            cpu: "500m"
            memory: "256Mi"
```

#### shift-api-deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shift-api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: shift-api
  template:
    metadata:
      labels:
        app: shift-api
    spec:
      containers:
      - name: shift-api
        image: myshiftapi:latest
        ports:
        - containerPort: 80
        env:
        - name: DATABASE_URL
          value: "mysql://user:password@shift-db:3306/shift-db"
        resources:
          limits:
            cpu: "1"
            memory: "512Mi"
          requests:
            cpu: "500m"
            memory: "256Mi"
```

#### user-api-service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: user-api
spec:
  selector:
    app: user-api
  ports:
    - name: http
      port: 80
      targetPort: 80
  type: ClusterIP
```

### Here are example YAML files for the shift API service, HPA, and Ingress:

#### shift-api-service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: shift-api
spec:
  selector:
    app: shift-api
  ports:
    - name: http
      port: 80
      targetPort: 80
  type: ClusterIP
```

#### hpa.yaml
```
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: shift-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: shift-api
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Object
    object:
      metricName: cpu_usage
      target:
        apiVersion: v1
        kind: Pod
        name: shift-api
      targetValue: 70
```

#### ingress.yaml
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /user
        pathType: Prefix
        backend:
          service:
            name: user-api
            port:
              name: http
      - path: /shift
        pathType: Prefix
        backend:
          service:
            name: shift-api
            port:
              name: http
```

### Here are the example files for the helm directory:

#### values-staging.yaml
This is file for the values.yaml file for the staging environment.
```
image:
  repository: nginx
  tag: latest

service:
  type: LoadBalancer

ingress:
  enabled: true
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  hosts:
    - host: staging.example.com
      paths:
        - path: /user
          backend:
            serviceName: user-api
            servicePort: http
        - path: /shift
          backend:
            serviceName: shift-api
            servicePort: http

autoscaling:
  enabled: true
  minReplicas: 1
  maxReplicas: 10
  cpuTargetPercentage: 70

replicaCount: 1

database:
  host: my-database
  name: user_db
  user: myuser
  password: mypassword
```

#### values-production.yaml
This is for the values.yaml file for the production environment.
```
image:
  repository: nginx
  tag: latest

service:
  type: LoadBalancer

ingress:
  enabled: true
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  hosts:
    - host: example.com
      paths:
        - path: /user
          backend:
            serviceName: user-api
            servicePort: http
        - path: /shift
          backend:
            serviceName: shift-api
            servicePort: http

autoscaling:
  enabled: true
  minReplicas: 1
  maxReplicas: 20
  cpuTargetPercentage: 70

replicaCount: 2

database:
  host: my-database
  name: user_db
  user: myuser
  password: mypassword
```

#### Chart.yaml
This is the metadata file for the helm chart.
```
apiVersion: v2
name: my-app
description: A Helm chart for deploying my app
type: application
version: 1.0.0
appVersion: 1.0.0

```
