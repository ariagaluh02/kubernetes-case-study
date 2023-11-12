# kubernetes-case-study

# KUBERNETES CONFIGURATION CASE STUDY

## Case Study :

**1. Create a deployment as follows:**
- Name: nginx-app
- Using container nginx with version 1.11.10-alpine
- The deployment should contain 3 replicas <br><br>

>**Answer:**
- Create deployment yaml named **nginx-deployment.yaml**. Configure as follows:
    ```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
        name: nginx-app
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: nginx-app
    strategy:
      type: RollingUpdate
      rollingUpdate:
        maxSurge: 1
        maxUnavailable: 1
    minReadySeconds: 5
    template:
      metadata:
        labels:
          app: nginx-app
      spec:
        containers:
        - name: nginx
          image: nginx:1.11.10-alpine
          ports:
          - containerPort: 80
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
    ```
  With this configuration:
  - nginx will deploy with name **nginx-app**.
  - nginx-app will use image with version **1.11.10-alpine**.
  - nginx-app container will deploy with 3 replicas.
  - nginx-app will support rolling update strategy.
  <br>
  <br>
- I create an extra yaml (service nginx) to testing app-nginx running on the kubernetes. **nginx-service.yaml** contains:
    ```
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-service
    spec:
      selector:
        app: nginx-app
      ports:
        - protocol: TCP
          port: 80
          targetPort: 80
      type: LoadBalancer
    ```

- Apply deployment and service yaml with this commands:
    ```
    kubectl apply -f nginx-deployment.yaml -f nginx-service.yaml
    ```
   it will deploy to default namespace.

#

**2. Deploy the application with new version 1.11.13-alpine, by performing a rolling update.**

>**Answer:**
- Run command:
  ```
  kubectl set image deployment nginx-app nginx=nginx:1.11.13-alpine
  ```
  
  With this command, the nginx-app will update to newer version.

#

**3. Rollback that update to the previous version 1.11.10-alpine.**

> **Answer:**
- Run command:
  ```
  kubectl rollout undo deployment nginx-app --to-revision=1
  ```
  
  With this command, the nginx-app will revert to the previous verision.

#

**4. Set a node named k8s-node-1 as unavailable and reschedule all the pods running on it.**

> **Answer:**
- Run command:
  ```
  kubectl drain k8s-node-1 --ignore-daemonsets
  ```

  With this command, the k8s-node-1 set to unavailable, and the pods will continue running on available nodes.

- Make the nodes available again, and reschedule all the pods by running this command:
  ```
  kubectl uncordon k8s-node-1
  ```

#

**5. Create a Persistent Volume with name app-data, of capacity 2Gi and access mode ReadWriteMany. The type of volume is hostPath and its location is /srv/app-data. Create a Persistent Volume Claim that requests the Persistent Volume you had created above. The claim should request 2Gi. Ensure that the Persistent Volume Claim has the same storageClassName as the Persistent Volume you had previously created.**

> **Answer:**
- Create persistent volume yaml named **pv-app-data.yaml**. Configure as follows:
  ```
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: app-data
  spec:
    storageClassName: ""
    capacity:
      storage: 2Gi
    accessModes:
      - ReadWriteMany
    hostPath:
      path: /srv/app-data
  ```
  The persistent volume name will deploy as **app-data**. It will have 2Gi storage capacity with access mode **ReadWriteMany**, and the path is **/srv/app-data**.


- Create persistent volume claim yaml named **pvc-app-data.yaml**. Configure as follows:
  ```
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: app-data-claim
  spec:
    storageClassName: ""
    accessMode:
      - ReadWriteMany
    resources:
      requests:
        storage: 2Gi
    volumeName: app-data
  ```

  The persistent volume claim name will deploy as **app-data-claim**. It will use all 2Gi storage, and it will claim the **app-data** volume. Both of the them will use same storageClassName (default).

  #

**6. You have been asked to set up a Kubernetes cluster, one master and one worker node. You have done the initialization of the master, what is the next steps to make the worker node join the cluster?**

> **Answer:**


- First, we generate Token from Master node with kubeadm:
  ```
  kubeadm token create --print-join-command
  ```

- Copy **kubeadm join** command from the output of the previous step and run it on the worker node:
  ```
  kubeadm join <master-node-ip>:<master-node-port> --token <token> --discovery-token-ca-cert-hash sha256:<has>
  ```

- After the worker node has joined the cluster, next we configure **kubectl** on the worker node to communicate with the cluster. Copy **admin.conf** from master node to the worker node, and place it at **/etc/kubernetes/kubelet.conf**:
  ```
  sudo cp /etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf
  ```

- Start kubelet service on the worker node:
  ```
  sudo systemctl enable kubelet && systemctl start kubelet
  ```

  Now our worker node should be part of the kubernetes cluster.
