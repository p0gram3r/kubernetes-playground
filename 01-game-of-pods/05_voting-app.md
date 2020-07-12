```
apiVersion: v1
kind: Namespace
metadata:
  name: vote
---
apiVersion: v1
kind: Service
metadata:
  name: vote-service
  namespace: vote
spec:
  type: NodePort
  ports:
    - port: 5000
      targetPort: 80
      nodePort: 31000
  selector:
    app: vote-deployment
---
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: vote
spec:
  type: ClusterIP
  ports:
    - port: 6379
      targetPort: 6379
  selector:
    app: redis-deployment
---
apiVersion: v1
kind: Service
metadata:
  name: db
  namespace: vote
spec:
  type: ClusterIP
  ports:
    - port: 5432
      targetPort: 5432
  selector:
    app: db-deployment
---
apiVersion: v1
kind: Service
metadata:
  name: result-service
  namespace: vote
spec:
  type: NodePort
  ports:
    - port: 5001
      targetPort: 80
      nodePort: 31001
  selector:
    app: result-deployment
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vote-deployment
  namespace: vote
spec:
  replicas: 1
  selector:
      matchLabels:
        app: vote-deployment
  template:
    metadata:
        labels:
          app: vote-deployment
    spec:
      containers:
        - name: vote
          image: kodekloud/examplevotingapp_vote:before
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
  namespace: vote
spec:
  replicas: 1
  selector:
      matchLabels:
        app: redis-deployment
  template:
    metadata:
        labels:
          app: redis-deployment
    spec:
      containers:
        - name: redis
          image: redis:alpine
          volumeMounts:
            - mountPath: /data
              name: redis-data
      volumes:
        - name: redis-data
          emptyDir: {}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: worker
  namespace: vote
spec:
  replicas: 1
  selector:
      matchLabels:
        app: worker-deployment
  template:
    metadata:
        labels:
          app: worker-deployment
    spec:
      containers:
        - name: worker
          image: kodekloud/examplevotingapp_worker
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db-deployment
  namespace: vote
spec:
  replicas: 1
  selector:
      matchLabels:
        app: db-deployment
  template:
    metadata:
        labels:
          app: db-deployment
    spec:
      containers:
        - name: postgres
          image: postgres:9.4
          env:
            - name: POSTGRES_HOST_AUTH_METHOD
              value: trust
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: db-data
      volumes:
        - name: db-data
          emptyDir: {}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: result-deployment
  namespace: vote
spec:
  replicas: 1
  selector:
      matchLabels:
        app: result-deployment
  template:
    metadata:
        labels:
          app: result-deployment
    spec:
      containers:
        - name: worker
          image: kodekloud/examplevotingapp_result:before
```
