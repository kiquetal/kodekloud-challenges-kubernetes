#### Challenges

- developer-role

      developer-role', should have all(*) permissions for services in development namespace
      developer-role', should have all permissions(*) for persistentvolumeclaims in development namespace
      developer-role', should have all(*) permissions for pods in development namespace

- developer-role-binding

      create rolebinding = developer-rolebinding, role= 'developer-role', namespace = development
      rolebinding = developer-rolebinding associated with user = 'martin'

- kube-config


      set context 'developer' with user = 'martin' and cluster = 'kubernetes' as the current context.

- jekyll-pvc


    Storage Request: 1Gi
    Access modes: ReadWriteMany
    pvc name = jekyll-site, namespace = development
    'jekyll-site' PVC should be bound to the PersistentVolume called 'jekyll-site'.

- jekyll-pv

jekyll-site pv is already created. Inspect it before you create the pvc.




