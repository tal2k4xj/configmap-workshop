# Configmap with volume

```
$ nano configmap.yaml
```

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: tal-cm
data:
  index.html: Tal

```

```
$ nano podcm.yaml
```

```yml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-test
  labels:
    app: configmap-test
spec:
  containers:
    - name: test-container
      image: nginx
      ports:
        - name: http-server
          containerPort: 80
      volumeMounts:
      - name: config-volume
        mountPath: /usr/share/nginx/html
  volumes:
    - name: config-volume
      configMap:
        name: tal-cm
```

```
$ nano servicecm.yaml
```

```yml
apiVersion: v1
kind: Service
metadata:
  name: configmap-test
  labels:
    app: configmap-test
spec:
  ports:
  - port: 80
    targetPort: http-server
  selector:
    app: configmap-test
  type: NodePort
```

```
$ kubectl create -f configmap.yaml
```

```
$ kubectl create -f podcm.yaml
```

```
$ kubectl create -f servicecm.yaml
```

```
$ kubectl get svc
```

# Configmap with env var

1) create configmap 

```
$ kubectl create configmap logger --from-literal=log_level=error
```

2) see configmap yaml

```
$ kubectl get configmaps logger -o yaml
```

3) create reader-cm-deployment.yaml file

```
$ nano reader-cm-deployment.yaml
```
4) Copy & Paste yaml

```yml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: log-reader-configmap
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: logreader
    spec:
      containers:
      - name: logreader
        image: karthequian/reader:latest
        env:
        - name: log_level
          valueFrom:
            configMapKeyRef:
              #Read from a configmap called log-level
              name: logger
              #Read the key called log_level
              key: log_level
```

5) create deploy yaml

```
$ kubectl apply -f reader-cm-deployment.yaml
```

6) get configmap reader log 

```
$ kubectl get pods | grep -i log-reader-configmap

log-reader-configmap-678cbc5868-r4sfj   1/1     Running   0          2m
```
7) View logd of configmap reader pod

```
$ kubectl logs log-reader-configmap-<random>
```















































