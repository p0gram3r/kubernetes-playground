### Step 1: add new user and context to kubeconfig
```
kubectl config set-credentials drogo --username=drogo --client-certificate=/root/drogo.crt --client-key=/root/drogo.key
kubectl config set-context developer --cluster=kubernetes --user=drogo --namespace=development
kubectl config use-context developer
```

### Step 2: create necessary K8s objects
```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jekyll-site
  namespace: development
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: jekyll
  namespace: development
  labels:
    run: jekyll
spec:
  initContainers:
    - name: copy-jekyll-site
      image: kodekloud/jekyll
      command: [ "jekyll", "new", "/site" ]
      volumeMounts:
        - mountPath: /site
          name: site
  containers:
    - name: jekyll
      image: kodekloud/jekyll-serve
      volumeMounts:
        - mountPath: /site
          name: site
  volumes:
    - name: site
      persistentVolumeClaim:
        claimName: jekyll-site
---
apiVersion: v1
kind: Service
metadata:
  name: jekyll
  namespace: development
spec:
  type: NodePort
  ports:
    - port: 8080
      targetPort: 4000
      nodePort: 30097
  selector:
    run: jekyll
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer-role
  namespace: development
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods", "persistentvolumeclaims", "services"]
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-rolebinding
  namespace: development
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: developer-role
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: drogo
```
