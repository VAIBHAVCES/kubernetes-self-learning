## Problem
* Pods have an epheremal IP. Their IP gets changed whenever they are delete or anythign happends

    * Note pods IP are allocated by **Container Network Interface** which we basically installed on worker and master nodes at the time of cluster setup 

    * It's worth to mention here that this assignment works in association with kubelet which tells that which IPs are already assigned and which are yet to be assigned

* And also a ReplicaSet in Kubernetes is primarily responsible for maintaining the desired number of pod replicas and ensuring high availability. While it does not inherently provide traffic routing or load balancing functionality, you can still access the individual pods created by the ReplicaSet without using a Kubernetes Service.


## Networking Inside  Pod
* One question comes here is what if one pod itself have 3 container server running how do they interact and do they have there own IPs too ?

* By default, containers within a pod share the same network namespace, and they have the same IP address as the pod. They effectively behave as if they are on the same machine.

* Containers within a pod communicate with each other using the loopback interface (localhost), and they can communicate with the world outside the pod using the pod's IP address.
* Some CNI plugins support container-level networking, where each container in a pod can have its own unique IP address. This is less common and might not be supported by all CNI plugins.

## Features of a Service
* It provides a static IP to access set of pod with an static IP
* The set of Pods targeted by a Service is usually determined by a selector.
*  A Service in Kubernetes is a REST object, similar to a Pod.
*  The default protocol for Services is TCP; you can also use any other supported protocol.