## etcd-ansible-operator

This operator implements the [etcd-operator](https://github.com/coreos/etcd-operator/) using ansible and is run using [ansible-operator](https://github.com/water-hole/ansible-operator)

#### Steps to bring an etcd cluster up

1. Create rbac `kubectl create -f deploy/rbac.yaml`
2. Create crds `kubectl create -f deploy/crd/yaml`
3. Deploy the operator `kubectl create -f deploy/operator.yaml`
4. Create a cluster `kubectl create -f deploy/cr.yaml`
5. Verify that cluster is up by `kubectl get pods -l app=etcd`. You should see something like this
    ```
    $ kubectl get pods -l app=etcd
    NAME                              READY     STATUS    RESTARTS   AGE
    example-etcd-cluster-1a7d2c2f8b   1/1       Running   0          14m
    example-etcd-cluster-5afd8f00ce   1/1       Running   0          14m
    example-etcd-cluster-e43636bc7c   1/1       Running   0          14m
    ```

#### Scale cluster up

1. Bring a cluster up as discussed above
2. Edit the `deploy/cr.yaml` file as follows:

    ```
    apiVersion: "etcd.database.coreos.com/v1beta2"
    kind: "EtcdCluster"
    metadata:
      name: "example-etcd-cluster"
      ## Adding this annotation make this cluster managed by clusterwide operators
      ## namespaced operators ignore it
      # annotations:
      #   etcd.database.coreos.com/scope: clusterwide
    spec:
      size: 5
    #  TLS:
    #    static:
    #      member:
    #        peerSecret: etcd-peer-tls
    #        serverSecret: etcd-server-tls
    #      operatorSecret: etcd-client-tls
      version: "3.2.13"
    ```
   This shoudl scale up the cluster by 2 pods.

3. Apply the changes `kubectl apply -f deploy/cr.yaml`
4. Verify that the cluster has scaled up by `kubectl get pods -l app=etcd`. You should see something like this:
    ```
    $ kubectl get pods -l app=etcd
    NAME                              READY     STATUS    RESTARTS   AGE
    example-etcd-cluster-1a7d2c2f8b   1/1       Running   0          18m
    example-etcd-cluster-1c497c44c5   1/1       Running   0          29s
    example-etcd-cluster-5afd8f00ce   1/1       Running   0          18m
    example-etcd-cluster-a3f3b02a1b   1/1       Running   0          18s
    example-etcd-cluster-e43636bc7c   1/1       Running   0          18m
    ```
#### Check failure recovery
1. Bring a cluster up.
2. Delete a pod to simulate a failure `kubectl delete pod example-etcd-cluster-1a7d2c2f8b`
3. Within sometime, you should see the deleted pod going away and being replaced by a new pod, something like this:
    
    ```$ kubectl get pods -l app=etcd
       NAME                              READY     STATUS    RESTARTS   AGE
       example-etcd-cluster-1c497c44c5   1/1       Running   0          3m
       example-etcd-cluster-25f6bd225a   1/1       Running   0          8s
       example-etcd-cluster-5afd8f00ce   1/1       Running   0          21m
       example-etcd-cluster-a3f3b02a1b   1/1       Running   0          3m
       example-etcd-cluster-e43636bc7c   1/1       Running   0          21m
       ```

#### TLS

Work in progress

#### Upgrades

Work in progress