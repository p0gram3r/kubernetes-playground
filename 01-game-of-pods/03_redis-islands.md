- prepare data directories
```
ssh node01
mkdir /redis01 /redis02 /redis03 /redis04 /redis05 /redis06
exit
```

- prepate PersistentVolumes
  - do not create the StatefulSet until you have finished creating the PersistentVolumeClaim!
  - In StatefulSets, the PersistentVolumeClaim name follows a predictable pattern: (volumeclaimtemplates-name)-(statefulset-name)-(replica-index).
  - note: replica indexes start with 0, not 1 ;-)
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis01
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  claimRef:
    namespace: default
    name: data-redis-cluster-0
  hostPath:
    path: "/redis01"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis02
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  claimRef:
    namespace: default
    name: data-redis-cluster-1
  hostPath:
    path: "/redis02"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis03
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  claimRef:
    namespace: default
    name: data-redis-cluster-2
  hostPath:
    path: "/redis03"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis04
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  claimRef:
    namespace: default
    name: data-redis-cluster-3
  hostPath:
    path: "/redis04"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis05
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  claimRef:
    namespace: default
    name: data-redis-cluster-4
  hostPath:
    path: "/redis05"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis06
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  claimRef:
    namespace: default
    name: data-redis-cluster-5
  hostPath:
    path: "/redis06"
```

- create StatefulSet and Service
```
apiVersion: v1
kind: Service
metadata:
  name: redis-cluster-service
spec:
  type: ClusterIP
  ports:
    - name: client
      port: 6379
      targetPort: 6379
    - name: gossip
      port: 16379
      targetPort: 16379
  selector:
    app: redis
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-cluster
spec:
  serviceName: "redis-cluster"
  selector:
    matchLabels:
      app: redis-cluster
  replicas: 6
  template:
    metadata:
      labels:
        app: redis-cluster
    spec:
      containers:
      - name: redis
        image: redis:5.0.1-alpine
        command: ["/conf/update-node.sh", "redis-server", "/conf/redis.conf"]
        env:
        - name: "POD_IP"
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        ports:
        - name: client
          containerPort: 6379
        - name: gossip
          containerPort: 16379
        volumeMounts:
        - name: conf
          mountPath: /conf
          readOnly: false
        - name: data
          mountPath: /data
          readOnly: false
      volumes:
      - name: conf
        configMap:
          name: redis-cluster-configmap
          defaultMode: 0755
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

- run the Redis cluster init command
  - note: keep the trailing space at the end of the jsonpath!
```
kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 $(kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 ')
```
