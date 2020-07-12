- create hostPath directories on node01
```
ssh node01
mkdir /drupal-mysql-data
mkdir /drupal-data
exit
```

- create all necessary K8s objects
```
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: drupal-mysql-pv
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 5Gi
  hostPath:
    path: "/drupal-mysql-data"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: drupal-pv
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 5Gi
  hostPath:
    path: "/drupal-data"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: drupal-mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: drupal-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
apiVersion: v1
kind: Secret
metadata:
  name: drupal-mysql-secret
data:
  MYSQL_ROOT_PASSWORD: "cm9vdF9wYXNzd29yZA=="
  MYSQL_DATABASE: "ZHJ1cGFsLWRhdGFiYXNl"
  MYSQL_USER: "cm9vdA=="
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: drupal-mysql
spec:
  replicas: 1
  selector:
      matchLabels:
        type: b
  template:
    metadata:
        labels:
          type: b
    spec:
      containers:
        - name: drupal-mysql
          image: mysql:5.7
          env:
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: drupal-mysql-secret
                  key: MYSQL_USER
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: drupal-mysql-secret
                  key: MYSQL_DATABASE
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: drupal-mysql-secret
                  key: MYSQL_ROOT_PASSWORD
          volumeMounts:
            - mountPath: /var/lib/mysql
              subPath: dbdata
              name:  drupal-mysql
      volumes:
        - name: drupal-mysql
          persistentVolumeClaim:
            claimName: drupal-mysql-pvc
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: drupal
  labels:
    app: drupal
spec:
  replicas: 1
  selector:
      matchLabels:
        app: drupal
  template:
    metadata:
        labels:
          app: drupal
    spec:
      initContainers:
        - name: init-sites-volume
          image: drupal:8.6
          command: [ "/bin/bash", "-c" ]
          args: [ 'cp -r /var/www/html/sites/ /data/; chown www-data:www-data /data/ -R' ]
          volumeMounts:
            - mountPath: /data
              name:  drupal
      containers:
        - name: drupal
          image: drupal:8.6
          volumeMounts:
            - mountPath: /var/www/html/modules
              subPath: modules
              name:  drupal
            - mountPath: /var/www/html/profiles
              subPath: profiles
              name:  drupal
            - mountPath: /var/www/html/sites
              subPath: sites
              name:  drupal
            - mountPath: /var/www/html/themes
              subPath: themes
              name:  drupal
      volumes:
        - name: drupal
          persistentVolumeClaim:
            claimName: drupal-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: drupal-mysql-service
spec:
  type: ClusterIP
  ports:
    - targetPort: 3306
      port: 3306
  selector:
    type: b
---
apiVersion: v1
kind: Service
metadata:
  name: drupal-service
spec:
  type: NodePort
  ports:
    - port: 80
      nodePort: 30095
  selector:
    app: drupal
```
