#### Challenge 2

node01 is ready and can schedule pods?


pre

```
NAME           STATUS                     ROLES           AGE   VERSION
controlplane   Ready                      control-plane   88m   v1.27.0
node01         Ready,SchedulingDisabled   <none>          88m   v1.27.0
```
kubectl uncordon node01
post
```
 kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   89m   v1.27.0
node01         Ready    <none>          88m   v1.27.0
```


Create new PersistentVolume = 'data-pv'

PersistentVolume = data-pv, accessModes = 'ReadWriteMany'

PersistentVolume = data-pv, hostPath = '/web'

PersistentVolume = data-pv, storage = '1Gi'

``` yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /web
```
Create new PersistentVolumeClaim = 'data-pvc'
PersistentVolume = 'data-pvc', accessModes = 'ReadWriteMany'
PersistentVolume = 'data-pvc', storage request = '1Gi'
PersistentVolume = 'data-pvc', volumeName = 'data-pv'

```yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  volumeName: data-pv
```

Create a pod for file server, name: 'gop-file-server'
pod: gop-file-server image: 'kodekloud/fileserver'
pod: gop-file-server mountPath: '/web'
pod: gop-file-server volumeMount name: 'data-store'
pod: gop-file-server persistent volume name: data-store

pod: gop-file-server persistent volume claim used: 'data-pvc'

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: gop-file-server
  labels:
    app: gop-file-server
spec:
  containers:
    - name: gop-file-server
      image: kodekloud/fileserver
      ports:
        - containerPort: 8080
          protocol: TCP
      volumeMounts:
        - name: data-store
          mountPath: /web
  volumes:
    - name: data-store
      persistentVolumeClaim:
        claimName: data-pvc
```


New Service, name: 'gop-fs-service'
Service name: gop-fs-service, port: '8080'
Service name: gop-fs-service, targetPort: '8080'

 ``` yaml
apiVersion: v1
kind: Service
metadata:
  name: gop-fs-service
spec:
  selector:
    app: gop-file-server
  ports:
    - name: http
      protocol: TCP
      port: 8080
      targetPort: 8080
```
Copy between nodes!
scp /media/* node01:/web
