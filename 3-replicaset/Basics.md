### Why we need replicaset

* In prod if you have 5 pods as requirement and one pod crashes then how you might loose your traffick
* How to maintain a desired state of 5 pods
* Expectation is if any of the pod crash new one should be spinned up immediately

### Squence of Steps

* Defining replicaset : You can push in a replicaset manifet file. This definition includes the desired number of replicas, a template for creating Pods, and labels to select the Pods controlled by the ReplicaSet.

* ReplicaSet Controller: In the control plane, there is a ReplicaSet controller that continuously watches the API server for changes in the desired state of the ReplicaSets. When a new ReplicaSet is created or an existing one is updated, the controller takes action.
  > I think this should be clear that when I say replicaset controller that means we are talking abuot controll plane which basically resides in master node 

* It's worth to mention too that whatever information kubernetes components needs about the worker nodes or the pods running on the worker nodes that its retrieves only via the api server. So our replicaset controller also gets the information via API server and compares it and based on that it communicatest the required action for worker nodes like creating, deleteing or updating pods 
* Desired State Calculation: The ReplicaSet controller calculates the difference between the current state (number of running Pods) and the desired state (as defined in the ReplicaSet). 

* Kubelet: The kubelet running on each node is responsible for monitoring the state of the node and the Pods. It ensures that the specified number of Pods are running on the node. If a Pod is terminated or if a new Pod needs to be created, the kubelet takes action to bring the node's state in line with the desired state.

* Label Matching: The Pods created by the ReplicaSet are labeled with a specific label. This label allows the ReplicaSet to track which Pods it controls and ensures that only the correct Pods are managed by the ReplicaSet.

* Pod Creation: If there are fewer Pods than desired, the controller creates new Pods using the template defined in the ReplicaSet. These Pods are assigned unique names and labels. The Pods are scheduled onto suitable nodes by the scheduler.

* Pod Termination: If there are too many Pods, the controller selects the excess Pods and sends a termination signal to them. The termination process ensures that the Pods shut down gracefully, allowing them to complete any work they are doing.

### Breakdown of ./first-rs.yml
```sh
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: first-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: marvel
  template:
```
* As you see above we follow same `AKMS` pattern in replicaset too
* New fields in spec are the critical ones, for e.g `replicas`. As name suggest it stands for number of replicas

* Now we also have selector `label`, now what the case is that replicaset controller will retain pods labeled with `app:marvel`
    * Most amazing thing about this point is that if you have any existing pod running with same label `app:marvel`, kubernets will automatically detect and wont trigger only 2 replicas because you already have one pod with same label