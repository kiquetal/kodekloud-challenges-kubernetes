Create a deployment: name = 'vote-deployment'
image = 'kodekloud/examplevotingapp_vote:before'
status: 'Running'

kubectl create deployment vote-deployment --image=kodekloud/examplevotingapp_vote:before -n vote

Tasks not completed!
Create a new service: name = vote-service
port = '5000'
targetPort = '80'
nodePort= '31000'
service endpoint exposes deployment 'vote-deployment'

kubectl expose deployment vote-deployment --port=5000 --target-port=80 --type=NodePort --name=vote-service -n vote
kubectl patch service vote-service --namespace=vote --type='json' --patch='[{"op": "replace", "path": "/spec/ports/0/nodePort", "value":31000}]'

#### Challenges4
Create new deployment, name: 'redis-deployment'
image: 'redis:alpine'
Volume Type: 'EmptyDir'
Volume Name: 'redis-data'
mountPath: '/data'

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
  namespace: vote
  labels:
    app: redis
spec:
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:alpine
        volumeMounts:
        - name: redis-data
          mountPath: /data
      volumes:
      - name: redis-data
        emptyDir: {}
    
```
kubectl expose deployment redis-deployment --port=6379 --target-port=6379 --type=ClusterIP --name=redis -n vote

Create new deployment. name: 'worker'
image: 'kodekloud/examplevotingapp_worker'

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: worker
  namespace: vote
  labels:
    app: worker
spec:
  selector:
    matchLabels:
      app: worker
  replicas: 1
  template:
    metadata:
      labels:
        app: worker
    spec:
      containers:
      - name: worker
        image: kodekloud/examplevotingapp_worker

  
```

Create new deployment. name: 'db-deployment'
image: 'postgres:9.4' and add the env: 'POSTGRES_HOST_AUTH_METHOD=trust'
Volume Type: 'EmptyDir'
Volume Name: 'db-data'
mountPath: '/var/lib/postgresql/data'

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db-deployment
  namespace: vote
  labels:
    app: db
spec:
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
        - name: db
          image: postgres:9.4
          env:
            - name: POSTGRES_HOST_AUTH_METHOD
              value: trust

          volumeMounts:
            - name: db-data
              mountPath: /var/lib/postgresql/data
      volumes:
        - name: db-data
          emptyDir: {}
```
kubectl expose deployment db-deployment --port=5432 --target-port=5432 --type=ClusterIP --name=db -n vote


Create new deployment, name: 'result-deployment'
image: 'kodekloud/examplevotingapp_result:before'
status: 'Running'

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: result-deployment
  namespace: vote
  labels:
    app: result
spec:
  selector:
    matchLabels:
      app: result
  template:
    metadata:
      labels:
        app: result
    spec:
      containers:
        - name: result
          image: kodekloud/examplevotingapp_result:before
```

kubectl expose deployment result-deployment --port=5001 --target-port=80 --type=NodePort --name=result-service -n vote
kubectl patch service result-service --namespace=vote --type='json' --patch='[{"op": "replace", "path": "/spec/ports/0/nodePort", "value":31001}]'
