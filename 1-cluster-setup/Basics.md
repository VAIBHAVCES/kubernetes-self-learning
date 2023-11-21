## How to setup kubernetes cluster

### Basic Notes Useful For Knowledge

#### Requirements for K8s Cluster 
- Basic kubernetes requires min of `2CPUs` and `1.7GiB` of RAM so make sure whenever you deploy a cloud instance it should follow this min requirement

- Note this specification is not applicable to worker nodes, this is for the master node 

- < TBD > -Specifications for worker node

#### [Capacity of a K8s cluster]( https://kubernetes.io/docs/setup/best-practices/cluster-large/#:~:text=No%20more%20than%205%2C000%20nodes,more%20than%20300%2C000%20total%20containers )
- A k8s cluster can't hold more than 5000 nodes

- No more than 150,000 total pods
- No more than 300,000 total containers

### Steps Performed on DigitalOcean Droplets
1. Got 3 droplets with required specification
2. Used [script]( https://raw.githubusercontent.com/syedmudasir20/Kubernetes-cluster/main/k8s-pre-req-scp.sh ) to deploy some of the dependency.
    * There are few highlighting problems with script
    * replace the kubeadm, kubectl specific versions take the lates versions
    * if possible take containerd CNI(Container Native Interface) latest too(not mandatory)
3. Simply ran `kubeadm init`
4. This raised me few problems which I discusss below

### Problem with above steps
#### Continuously failing k8s components
1. Right after `kubeadm init` I was seeing all k8s component were crashing. Refer below snippet I was seeing. Below one shows all pods in ready state, as I lost the code snippet. But as you can see below they were crashing in a sequential mannner you can see some frequent restarts
```sh
root@kube-master-try-2:~# kubectl get pods --all-namespaces
NAMESPACE     NAME                                        READY   STATUS            RESTARTS        AGE
kube-system   coredns-5dd5756b68-d9bph                    0/1     Pending           0               5h36m
kube-system   coredns-5dd5756b68-xwsl6                    0/1     Pending           0               5h36m
kube-system   etcd-kube-master-try-2                      1/1     Running           8 (5h8m ago)    5h32m
kube-system   kube-apiserver-kube-master-try-2            1/1     Running           9 (5h5m ago)    5h37m
kube-system   kube-controller-manager-kube-master-try-2   1/1     Running           11 (5h6m ago)   5h33m
kube-system   kube-proxy-rh88g                            1/1     Running           10 (5h4m ago)   5h36m
kube-system   kube-scheduler-kube-master-try-2            
```

2. So the problem here was in containerd 
3. Actually the problem lies inside the containerd configuration file which we need to change, follow the last message of this github issue
https://github.com/kubernetes/kubernetes/issues/110177
4. To be simple my above script applied a containerd config file, so after that I just ran this below command and it fixed it 
```sh
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```

Note: It is also possible you might not be able to run even the `kubectl get pods ` command because what behaviour I saw is `kube-apiserver` staets healthy in starting few mins, but as it sees other components such as `kubeproxy` `kube-controller-manager` failing then it probably tunrs off the other process.
*  In such scenarion you can once try a `systemctl restart kubelet`


Note : The exact problem here was : https://github.com/kubernetes/kubernetes/issues/110177#issuecomment-1161647736


* Note - Above command needs to be ran on containerd using worked nodes. In my case I ran it on master node due to which master nodes components were running healthy. But the worker kube system pod kube-proxy was failing non stop with no error in logs it was just showing this in describe
```sh
  Nameserver limits were exceeded, some nameservers have been omitted, the applied nameserver line is: 67.207.67.2 67.207.67.3 67.207.67.2
```
But the solution again was same running 
```sh
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```
and after that restarting kubelet and containerd
#### Core DNS not starting up 

1. Note core dns don't starts up until you have applied the weaveset configuration so take a chill pill.
2. If its in `Pending` state or some other then just ensure that your k8s components are healthy(runing without restart atleast for 5mins or more) then only prefer to apply pod networking configurations


#### Weavenet Fails with Network Overlap Error
1. Now after applying the default weave net file you will see soemtimes that your container starts to fail with some below error like 
```sh
Network 10.32.0.0/12 overlaps with existing route 10.47.0.0/16 on host
```
or 
```sh
Network 10.47.0.0/16 overlaps with existing route 10.47.0.0/16 on host
```
2. In both the cases most probably issues is that you have deinfe the custom IP range in the weavenet configuration file
```sh

              env:
                - name: INIT_CONTAINER
                  value: "true"
                - name: HOSTNAME
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: spec.nodeName
                # THIS BELOW IS THE PART WHICH YOU NEED TO ADD      
                - name: IPALLOC_RANGE
                  value: "10.0.0.0/16"
              image: 'weaveworks/weave-kube:latest'
              imagePullPolicy: Always
              readinessProbe:
                httpGet:
                  host: 127.0.0.1
                  path: /status
                  port: 6784
```

3. Or in the same directory I have provided the weave set file which worked for me prefer using it.


#### All Node Are Ready but Pods not getting created
1. Afterr doing everything I saw all my nodes were in ready state
2. Hence right after it I created one nginx pod
3. But there only when I listed I saw kube-proxy and weave-net pods are crashing and also my pod was failing on one of the worker nodes
4. I don't know what the hell happened !!! But after rebooting all the nodes I started to see all of them creating successfully.
5. Thanks to : https://stackoverflow.com/questions/50744371/pod-creation-stuck-in-containercreating-state
˚