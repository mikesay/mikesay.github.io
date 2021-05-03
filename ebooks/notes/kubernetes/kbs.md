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

### Start busybox-curl
```bash
kubectl run -it --image=yauritux/busybox-curl:latest  -- sh
```
> It is run as root user

### Start from a curl image
```bash
kubectl run curl -it --image=curlimages/curl:latest  -- sh
```

### Start busybox
```bash
kubectl run busybox -it --privileged=true --image=busybox  -- sh
```

### Start pod to run kubectl or helm
```bash
kubectl run kubectl -it --image=ubuntu:18.04 -- bash
apt-get update
apt-get install vim
apt-get install curl
cd /usr/local/bin
curl -LO https://dl.k8s.io/release/v1.21.0/bin/linux/amd64/kubectl
chmod a+x kubectl
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

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: dig
  name: dig
spec:
  replicas: 1
  selector:
    matchLabels:
      app: dig
  template:
    metadata:
      labels:
        app: dig
    spec:
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
              - key: kubernetes.io/hostname
                operator: In
                values:
                - cn-shanghai.10.229.249.51
      dnsPolicy: ClusterFirst
      nodeSelector:
        kubernetes.io/os: linux
      terminationGracePeriodSeconds: 300
      containers:
        - name: dig-51
          image: azukiapp/dig
          imagePullPolicy: IfNotPresent
          args:
            - bash
            - -c
            - "sleep 36000"
```
> If the node has taints, also need to add tolerations

## Start testing NFS server

### deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: nfs
  name: nfs-server
  labels:
    app: nfs-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nfs-server
  template:
    metadata:
      labels:
        app: nfs-server
        name: nfs-server
    spec:
      containers:
        - name: nfs-server
          image: itsthenetwork/nfs-server-alpine:12
          securityContext:
            privileged: true
          ports:
            - name: nfs
              containerPort: 2049
            - name: mountd
              containerPort: 20048
            - name: rpcbind
              containerPort: 111
          env:
            - name: SHARED_DIRECTORY
              value: /nfsshare
          volumeMounts:
          - mountPath: /nfsshare
            name: nfsshare
          imagePullPolicy: IfNotPresent
          resources:
            limits:
              cpu: 200m
              memory: 512Mi
            requests:
              cpu: 100m
              memory: 256Mi
      restartPolicy: Always
      volumes:
      - name: nfsshare
        emptyDir: {}
        # For hostPath, the client can mount but has no permission to write.
        #hostPath:
          # directory location on host
        #  path: /Users/mizha53/Documents/MikeStorage/MikeNFS
          # this field is optional
        #  type: Directory
```

### service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nfs-server
  namespace: nfs
spec:
  selector:
    app: nfs-server
  ports:
    - name: nfs
      port: 2049
    - name: mountd
      port: 20048
    - name: rpcbind
      port: 111
```
### Mount to the testing NFS server by PV
+ pv.yaml
    ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
    name: nfs-client
    labels:
        app: nfs-client
    spec:
    capacity:
        storage: 5Gi
    accessModes:
        - ReadWriteMany
    nfs:
        server: "10.101.253.230"
        path: "/"
    mountOptions:
        - vers=4
    ```
+ pvc.yaml
    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
    namespace: nfs
    name: nfs-client
    labels:
        app: nfs-client
    spec:
    accessModes:
        - ReadWriteMany
    storageClassName: ""
    resources:
        requests:
        storage: 5Gi
    selector:
        matchLabels:
        app: nfs-client
    ```
+ deployment.yaml
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    namespace: nfs
    name: nfs-client
    labels:
        app: nfs-client
    spec:
    replicas: 1
    selector:
        matchLabels:
        app: nfs-client
    template:
        metadata:
        name: nfs-client
        labels:
            app: nfs-client
        spec:
        containers:
            - name: nfs-client
            image: busybox
            securityContext:
                privileged: true
            volumeMounts:
            - mountPath: /var/nfs
                name: nfsshare
            imagePullPolicy: IfNotPresent
            resources:
                limits:
                cpu: 200m
                memory: 512Mi
                requests:
                cpu: 100m
                memory: 256Mi
            command: ["/bin/sh"]
            args: ["-c", "while true; do date >> /var/nfs/dates.txt; sleep 5; done"]
        restartPolicy: Always
        volumes:
        - name: nfsshare
            persistentVolumeClaim:
            claimName: nfs-client
    ```

### Mount to the testing NFS server

+ Start a busybox pod
+ Run mount command
    ```bash
    mount -v -o vers=4 nfs-server.nfs.svc.cluster.local:/ /tmp/test
    ```

## Minikube

+ Start Kubernetes
  If had proxy, set the proxy environment and the NO_PROXY especially https://minikube.sigs.k8s.io/docs/handbook/vpn_and_proxy/:  
  ```bash
  export https_proxy=http://192.168.0.208:7890
  export http_proxy=http://192.168.0.208:7890
  export all_proxy=socks5://192.168.0.208:7891
  export no_proxy=localhost,127.0.0.1,192.168.0.208,.aliyuncs.com,.youku.com,k8s.mikesay.com,wp.mikesay.com,docker-registry.local,10.96.0.0/12,192.168.99.0/24,192.168.39.0/24,192.168.49.0/24,192.168.64.0/24

  export HTTPS_PROXY=$https_proxy
  export HTTP_PROXY=$http_proxy
  export ALL_PROXY=$all_proxy
  export NO_PROXY=$no_proxy
  ```
  Use hyperkit driver:  
  ```bash
  minikube start --cpus=4 --memory='6g' --cni='flannel' --disk-size='60g' --driver='hyperkit'
  ```

  Use virtualbox driver:  
  ```bash
  minikube start --cpus=4 --memory='6g' --cni='flannel' --disk-size='60g' --driver='virtualbox'
  ```