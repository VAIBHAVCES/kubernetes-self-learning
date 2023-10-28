### Facts Learnt

##### I saw an issue during my setup that a wrong IP was assigned to my master node , when you run `kubectl get nodes` and whatever the internalIP we see there is used by worked nodes. This internal IP is the IP assisgned by your cloud provider. 

> To chaneg ip of worker nodes go to kubeadm config file \
> When we right `kubeadm init` this file only is used for creating  init pods(initial kube system pods) \
> FIle : `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf` \
> And append `--node-ip={IP TO BE ASSIGNED}` in the end of env variable `KUBELET_CONFIG_ARGS`

> Sample snippet
```sh
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml --node-ip=10.122.0.5"
```

##### What is `IP_ALLOC_RANGE` in weavenet ? What does it even signify, as mentioned in step guides

> Actually it stands for the IP range which should be assigned to the pod when the pods will be created for e.g if I set it to `"172.30.0.0/16"` 

> And when I created my first pod it was assigned an IP of `172.30.128.1`,

> how that IP is defined that is an internal topic but the point to note is that this is the IP range for the new pods which will be created
