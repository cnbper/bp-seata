# seata

```shell
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
 name: seata-system
EOF

cat <<EOF | kubectl -n seata-system apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: seata-registry-conf
data:
  registry.conf: |-    
    registry {
        type = "file"

        file {
            name = "file.conf"
        }
    }

    config {
        type = "file"

        apollo {
            app.id = "seata-server"
            apollo.meta = "http://192.168.1.204:8801"
        }
        file {
            name = "file.conf"
        }
    }
EOF

cat <<EOF | kubectl -n seata-system apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: seata
  labels:
    app: seata
spec:
  replicas: 1
  selector:
    matchLabels:
      app: seata
  template:
    metadata:
      labels:
        app: seata
    spec:
      containers:
      - name: seata
        image: registry.sloth.com/ipaas/seata:0.7.1
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8091
        volumeMounts:
        - name: registry-conf
          mountPath: /opt/seata/conf/registry.conf
          subPath: registry.conf
          readOnly: true
      volumes:
      - name: registry-conf
        configMap:
          name: seata-registry-conf
          items:
          - key: registry.conf
            path: registry.conf
EOF

cat <<EOF | kubectl -n istio-samples apply -f -
apiVersion: v1
kind: Service
metadata:
  name: seata
  labels:
    app: seata
    service: seata
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: seata
EOF
kubectl -n seata-system get pod
kubectl -n seata-system logs -f seata-66dc55d587-d64s7
kubectl -n seata-system exec -it seata-66dc55d587-d64s7 /bin/sh

kubectl delete ns seata-system
```

```shell
kubectl -n seata-system apply -f seata-samples.yaml

cat <<EOF | kubectl -n seata-system apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: business-service
spec:
  rules:
  - host: business.sloth.com
    http:
      paths:
      - path: /
        backend:
          serviceName: business-service
          servicePort: http
EOF
```
