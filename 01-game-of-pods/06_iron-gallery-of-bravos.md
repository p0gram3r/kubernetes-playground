```
apiVersion: v1
kind: Service
metadata:
  name: iron-gallery-service
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30099
  selector:
    run: iron-gallery
---
apiVersion: v1
kind: Service
metadata:
  name: iron-db-service
spec:
  type: ClusterIP
  ports:
  - port: 3306
    targetPort: 3306
  selector:
    db: mariadb
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-gallery
spec:
  replicas: 1
  selector:
      matchLabels:
        run: iron-gallery
  template:
    metadata:
        labels:
          run: iron-gallery
    spec:
      containers:
      - name: iron-db
        image: kodekloud/irongallery:2.0
        volumeMounts:
        - mountPath: /usr/share/nginx/html/data
          name: config
        - mountPath: /usr/share/nginx/html/uploads
          name: images
        resources:
          limits:
            cpu: "50m"
            memory: "100Mi"
      volumes:
      - name: config
        emptyDir: {}
      - name: images
        emptyDir: {}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-db
spec:
  replicas: 1
  selector:
      matchLabels:
        db: mariadb
  template:
    metadata:
        labels:
          db: mariadb
    spec:
      containers:
      - name: iron-db
        image: kodekloud/irondb:2.0
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: db
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: Braavo
        - name: MYSQL_DATABASE
          value: lychee
        - name: MYSQL_USER
          value: lychee
        - name: MYSQL_PASSWORD
          value: lychee
      volumes:
      - name: db
        emptyDir: {}
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: iron-gallery-firewall
spec:
  podSelector:
    matchLabels:
      db: mariadb
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          run: iron-gallery
    ports:
    - protocol: TCP
      port: 3306
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: iron-gallery-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: iron-gallery-braavos.com
    http:
      paths:
      - path: /
        backend:
          serviceName: iron-gallery-service
          servicePort: 80
```
