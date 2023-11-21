## Problems

* SO far we worked with replicaset and there we saw we can use it to maintain a particular desired state

* But problem wiht replicaset is :
    * It was never created for updating the new configurations for example if there is an image update or anything it will just destroy the old pods and will start creating new
    * Under the hood anyway deploymnents uses also the replicaset but the point is deployment is abstraction over the replicasets

    * Rollback is not possible


## Features of Deployment

* Rolling Upgrades: Deployments allow you to update your application by creating a new ReplicaSet with the updated version of your application and gradually scaling it up while scaling down the old ReplicaSet. This ensures that the update is performed with zero downtime.

* Rollbacks: If an update fails or introduces issues, Deployments allow you to roll back to a previous version easily.

* Pod Template Updates: Deployments allow you to modify the pod template (container image, environment variables, etc.) and apply those changes to the running pods.

* **Note** - You can directly go and update replicaset and update image and replica counts etc. too. But the problem is that its not graceful. 
    * Deployments give you a capability where it will slowly scale up the new image pods and slowly scale down the old image and manually update the replicaset.

    * So consider an application having 5 pods each with 5GiB of memory and thats max capacity our node have too so it will first gracefully create new deployment with 1 replica and move the old image replica count to 4, hence new: old will look like 1:4, 2:3, 3:1, 4:0 