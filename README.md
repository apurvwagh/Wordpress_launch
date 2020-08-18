# Wordpress_launch


steps Part 1 A: 


Createed a GKE cluster with the following details:


Name: cc-interview-apurv

Location type: Zonal

Zone: asia-south1-a

Nodepool:

Number of nodes: 3

Machine type: n1-standard-1

-------------------------------------------------------------------------------------------------

Part 1 B: 


Deploy a 2-tier application in different namespaces (Frontend application in frontend-ns and backend in backend-ns). The choice of application technologies will be dependent on the candidate.
The applications must be able to communicate with each other across namespaces.




here i am deploying wordpress application in two different namespace

1) create two namespace

# kubectl create ns wp
# kubectlcreate ns mysql
-------------------------------------------------------------------------------------------

2) create wordpres-deployment 


# create wordpress deployment
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer

apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: wordpress
  labels:

    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:4.8-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 80
          name: wordpress
----------------------------------------------------------------

3) create mysql-deployment 

#create service
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql

  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None

# create MYSQL deployment

apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:

  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 3306
          name: mysql
-------------------------------------------------------------------------------
4) create Kustomization file and pass through the namespaces

# create Kustomization file for wordpress and mysql

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
secretGenerator:
- name: mysql-pass
  literals:
  - password=redhat
resources:
  - mysql-deployment.yaml
  - wordpress-deployment.yaml

# kubectl apply -k ./ -n mysql -n mysql
# kubectl apply -k ./ -n mysql -n wp
--------------------------------------------------------------------------------

Part 2:


Apply HPA to the frontend application. Do a load test on the application in order to test the functionality of the HPA. We need to consider the HPA events to confirm the same.

Load test can be done by using any 3rd party tool.

#create metrics-server for autoscaling


# kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.7/components.yaml

# kubectl autoscale deployment wordpress-mysql --cpu-percent=50 --min=1 --max=10 -n wp



# kubectl get hpa -n mysql
# kubectl get hpa -n wp
-----------------------------------------------------------------------------------------------------------
