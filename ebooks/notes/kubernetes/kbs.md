## Entering into node of Kubernetes of Docker for Mac 

### Option1
+ Entering
    ```bash
    screen ~/Library/Containers/com.docker.docker/Data/vms/0/tty
    ```

+ Quit
    Ctrl+a, followed by pressing k and y.

### Option2
+ Entering
    ```bash
    docker run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
    ```
    > http://www.openkb.info/2021/03/docker-for-mac-could-not-find.html

+ Quit
    ```bash
    exit
    ```

## Start a pod in Kubernetes

### Start busybox
```bash
kubectl run -it --image=yauritux/busybox-curl:latest  -- sh
```
> It is run as root user

### Start from a curl image
```bash
kubectl run curl -it --image=curlimages/curl:latest  -- sh
```

### Start a pod on specific node by node affinity
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: curl
  name: curl
spec:
  replicas: 1
  selector:
    matchLabels:
      app: curl
  template:
    metadata:
      labels:
        app: curl
    spec:
      tolerations:
      - key: "usage"
        operator: "Equal"
        value: "nginx-ingress-reverse"
        effect: "NoSchedule"
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: type
                operator: NotIn
                values:
                - virtual-kubelet
              - key: k8s.aliyun.com
                operator: NotIn
                values:
                - "true"
              - key: usage
                operator: In
                values:
                - nginx-ingress-reverse
      dnsPolicy: ClusterFirst
      nodeSelector:
        kubernetes.io/os: linux
      terminationGracePeriodSeconds: 300
      containers:
        - name: curl
          image: curlimages/curl:latest
          imagePullPolicy: IfNotPresent
          args:
            - sh
            - -c
            - "sleep 36000"
```
> If the node has taints, also need to add tolerations