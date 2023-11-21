## Implemetation details about pod 
* Pod is most basic unit of k8s
* Pods can be created directly too means you can imperatively create pods directly
* You can refer file `./basic-pod.yml`
* You can try to curl to pod IP and you will see a welcome message from nginx
* We can also run dry run command which basically not creates the pod but gives an affirmation that everything is correct and working fine. Note this command is best to be used for imperative purpouses 
    * ```kubectl run web02 --image nginx --dry-run=client```
* Now imp use of the above command is use can use it to generate a manifest file in following way 
``` kubectl run web02 --image nginx --dry-run=client -o yaml | tee web02-pod.yml``` 

* You can also use flag ` kubectl get pods --show-labels` to list pod labels 

* Use this to label existing pod `kubectl label pod vormir hero=rdj`

### Annotations vs Labels
* You can filter different types of pods or resources with a particular label but not with annotations.

* Annotation are majorly used to store metadata information such as commit it, comiter, git branch etc. information.

*  Find this official definition
    > Labels are key/value pairs that can be attached to Kubernetes objects such as Pods and ReplicaSets. They can be arbitrary, and are useful for attaching identifying information to Kubernetes objects. Labels provide the foundation for grouping objects.

    > Annotations, on the other hand, provide a storage mechanism that resembles labels: annotations are key/value pairs designed to hold nonidentifying information that can be leveraged by tools and libraries.

* Multiple perspectives of annotations vs labels : https://stackoverflow.com/questions/67223202/what-is-the-difference-between-annotations-and-labels-in-in-kubernetes





