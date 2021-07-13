### Description
We are practicing a business case for services and deployments in Kubernetes. We are creating a simple service that provides a list of products that other pods can use in the future. We will deploy three replicas of the service's pods into a cluster and then create a Kubernete's service to provide access to these replica pods. We will use a test pod to check whether the Kubernetes Service works. We will use the busybox Docker container image to create a testing pod and we will use a Linux Academy Docker image of a container application that will serve out a product list.

### Architecture

### Prerequisites
- You only need a master cluster server to implement this. You can learn how to start up a master server in my [cluster bootstrap repo.](https://github.com/aacastillo/Kubernetes_Cluster_Bootstrap)

### Installation
1. Log into the master node
2. Deploy the test pod, I will be using a busy box pod

```
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: busybox1
  labels:
    app: busybox1
spec:
  containers:
  - image: busybox
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: busybox
  restartPolicy: Always
EOF 
```
3. Deploy the product-list service deployment with three pods. We will be using linuxacademy's Docker image to get the container application that will get the product list.
```
cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-list
  labels:
    app: product-list
spec:
  replicas: 3
  selector:
    matchLabels:
      app: product-list
  template:
    metadata:
      labels:
        app: product-list
    spec:
      containers:
      - name: product-list
        image: linuxacademycontent/store-products:1.0.0
        ports:
        - containerPort: 80
EOF 
```
4. Create a Kubernetes service for the product-list pods

```
cat << EOF | kubectl apply -f -
kind: Service
apiVersion: v1
metadata:
  name: product-list
spec:
  selector:
    app: product-list
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
EOF
```

5. Make sure the service is up in the cluster

```kubectl get svc product-list```
6. Query the product-list from the test pod

```kubectl exec busybox1 -- curl -s product-list```
