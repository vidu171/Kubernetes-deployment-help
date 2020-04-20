# Kubernetes-deployment-help
A repository so that I can revisit Kuberenetes deployment files 

## Kubernetes cheatsheet
This is a list of most usable commands, for an exhaustive list of kubectl commands go to :
` https://github.com/dennyzhang/cheatsheet-kubernetes-A4 `

### Steps To push docker images:
```
docker pull or docker build
docker tag SOURCE_IMAGE[:TAG] host/<project name>/IMAGE[:TAG]
docker login host
docker push host/IMAGE[:TAG]
```

### To apply kubectl config
 ```
 kubectl apply -f <config-name>.yaml
 ```
 
### Get kubectl command
 ```
 kubectl get all -o wide
 kubectl get pod -o wide
 kubectl get ing -o wide
 kubectl get service -o wide
 ```
 
### Kubectl describe
 ```
 kubectl describe pod/<pod-name>
 kubectl describe service/<service-name>
 kubectl describe deployment/<deployment-name>
 ```
### Kubectl deslete
 ```
 kubectl delete pod/<pod-name>
 kubectl delete service/<service-name>
 kubectl delete deployment/<deployment-name>
 ```
 
 ## Kubectl Deployment File
 
 ```
 apiVersion: apps/v1

# describe the deployment kind
kind: Deployment

metadata:
  name: mysql-deployment     # name of the deployment
  labels:
    app: mysql-deployment    #label neseccary to be used for selector

spec:
  replicas: 1      #number of replicas
  selector: 
    matchLabels:      # select the pod with pod name mysql-pod -(1)
      app: mysql-pod

  template:           # Pod defination
    metadata:           
      name: mysql-pod   
      labels:
        app: mysql-pod    # label that is used by selector in (1) to chose the pod

    spec:
      volumes:            #configuration for persistan storage
        - name: db-volume
          nfs:
            server: <storage_server_host>
            path: <storage Nas Path>
      containers:
        - name: mysql                       #actual container used by pod: mysql-pod
          image: centos/mysql-57-centos7    # image from docker hub
          env:
            - name: MYSQL_DATABASE          #enviorenment varaible such as username password
              value: tes_db
            - name: MYSQL_USER
              value: tes_user
            - name: MYSQL_PASSWORD
              value: test_pass
          ports:                            # container port
            - containerPort: 3306
          volumeMounts:                     # folder path that needs to be persisted
            - name: db-volume               # name should match the pod volume name
              mountPath: /var/lib/mysql/data
          resources:
            limits:
              memory: "500Mi"
              cpu: "200m"
            requests:
              memory: "300Mi"
              cpu: "100m"
 ```
 
 
 ## kubectl service file
 ```
 apiVersion: v1

kind: Service

metadata:
  name: mysql-db
  labels:
    app: mysql-service
    
spec:
  type: LoadBalancer
  ports:
    - port: 3306                  # port that will be used by service
      targetPort: 3306            # port inside the pod/container
  selector:
    app: mysql-pod   # the pod name to which this service is binded to. 
    
 ```

## kubectl ingress file
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata: 
  name: dscbot-ingress
  namespace: dscbot-ns
  annotations:            # annotations for https redirect
    kubernetes.io/ingress.allow-http: "false"
    ncp/http-redirect: "true"
spec:
  rules:
     - host: database.domain.com
       http:
         paths:
         - backend:
             serviceName: mysql-service
             servicePort: 3306
     - host: server.domain.com
       http:
         paths:
         - backend:
             serviceName: server-service
             servicePort: 3200
  tls:            # For ssl
    -
      hosts:
        - database.domain.com
        - Server.domain.com
      secretName: <mynamespace-tls-secret>
```

## TLS Certificate 
If you have the private.key certificate.cert file for ssl
just run the following command
```
 kubectl create secret -n <your-namespace> tls <mynamespace-tls-secret> --key private.key --cert certificate.crt
 ```
 Note: <mynamespace-tls-secret> this can be any value, but remember to put the same in ingress
  
  
