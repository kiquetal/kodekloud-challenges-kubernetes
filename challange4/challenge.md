
PersistentVolume - Name: redis06

Access modes: ReadWriteOnce

Size: 1Gi

hostPath: /redis06, directory should be created on worker node

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis01
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /redis01
```
StatefulSet - Name: redis-cluster
Replicas: 6
Pods status: Running (All 6 replicas)
Image: redis:5.0.1-alpine, Label = app: redis-cluster
container name: redis, command: ["/conf/update-node.sh", "redis-server", "/conf/redis.conf"]
Env: name: 'POD_IP', valueFrom: 'fieldRef', fieldPath: 'status.podIP' (apiVersion: v1)
Ports - name: 'client', containerPort: '6379'
Ports - name: 'gossip', containerPort: '16379'
Volume Mount - name: 'conf', mountPath: '/conf', readOnly:'false' (ConfigMap Mount)
Volume Mount - name: 'data', mountPath: '/data', readOnly:'false' (volumeClaim)
volumes - name: 'conf', Type: 'ConfigMap', ConfigMap Name: 'redis-cluster-configmap',
Volumes - name: 'conf', ConfigMap Name: 'redis-cluster-configmap', defaultMode = '0755'
volumeClaimTemplates - name: 'data'
volumeClaimTemplates - accessModes: 'ReadWriteOnce'
volumeClaimTemplates - Storage Request: '1Gi'

//create a statefulset redis-cluster

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-cluster
spec:
  serviceName: redis-cluster
  replicas: 6
  selector:
    matchLabels:
      app: redis-cluster
  template:
    metadata:
      labels:
        app: redis-cluster
    spec:
      containers:
      - name: redis
        image: redis:5.0.1-alpine
        ports:
        - containerPort: 6379
          name: client
        - containerPort: 16379
          name: gossip
        command: ["/conf/update-node.sh", "redis-server", "/conf/redis.conf"]
        env:
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
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
service

```yaml 

apiVersion: v1
kind: Service
metadata:
  name: redis-cluster-service
spec:
  ports:
  - port: 6379
    name: client
    targetPort: 6379
    name: client
  - port: 16379
    name: gossip
    targetPort: 16379
    name: gossip
  clusterIP: None
  selector:
    app: redis-cluster
```


Command: kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 $(kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 {end}')
