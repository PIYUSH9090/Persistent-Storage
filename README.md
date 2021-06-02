# Google Cloud: Configuring Persistent Storage for Google Kubernetes Engine

PersistentVolumes are storage that is available to a Kubernetes cluster. PersistentVolumeClaims enable Pods to access PersistentVolumes.

## Task 1. Create PVs and PVCs

### Try out the code on Google Cloud Platform
[![Open in Cloud Shell](http://gstatic.com/cloudssh/images/open-btn.png)](https://console.cloud.google.com/cloudshell/open?git_repo=https://github.com/GoogleCloudPlatform/training-data-analyst.git)

### Connect to the lab GKE cluster

In Cloud Shell, type the following command to set the environment variable for the zone and cluster name.

```
export my_zone=us-central1-a
export my_cluster=standard-cluster-1
```

Configure access to your cluster for kubectl

```
gcloud container clusters get-credentials $my_cluster --zone $my_zone
```

### Create and apply a manifest with a PVC

First we have to clone the repo to the lab Cloud Shell.

```
git clone https://github.com/GoogleCloudPlatform/training-data-analyst
```

Now go to the directory where you want to create yaml files.


Let's check the pvc status

```
kubectl get persistentvolumeclaim
```

You will get nothing like no resource found.

To create the PVC, execute the following command:

```
kubectl apply -f pvc-demo.yaml
```

To show your newly created PVC, execute the following command:

```
kubectl get persistentvolumeclaim
```

You will get 1 volume which is bound and you can go to gcloud console and check the kubernetes cluster then go to storage and you will see the created bound volume.

## Task 2. Mount and verify Google Cloud persistent disk PVCs in Pods

In this task, you attach your persistent disk PVC to a Pod. You mount the PVC as a volume as part of the manifest for the Pod.

### Mount the PVC to a Pod

In my repo there is yaml file called "pod-volume-demo.yaml", which is pod that connect persistent volume to disks.

So let's create the Pod with the volume, execute the following command:

```
kubectl apply -f pod-volume-demo.yaml
```

List the Pods in the cluster.

```
kubectl get pods
```

 To verify the PVC is accessible within the Pod, you must gain shell access to your Pod. To start the shell session, execute the following command:

```
kubectl exec -it pvc-demo-pod -- sh
```

To create a simple text message as a web page in the Pod enter the following commands:

```
echo Test webpage in a persistent volume!>/var/www/html/index.html
```

```
chmod +x /var/www/html/index.html
```

Verify the text file contains your message.

```
cat /var/www/html/index.html
```

Now exit the container and open our directory shell.

### Test the persistence of the PV

You will now delete the Pod from the cluster, confirm that the PV still exists, then redeploy the Pod and verify the contents of the PV remain intact.

Delete the pvc-demo-pod.

``` 
kubectl delete pod pvc-demo-pod
```

List the Pods in the cluster.

```
kubectl get pods
```
There should be no Pods on the cluster.

To show your PVC, execute the following command:

```
kubectl get persistentvolumeclaim
```

Your PVC still exists, and was not deleted when the Pod was deleted.

Redeploy the pvc-demo-pod.

```
kubectl apply -f pod-volume-demo.yaml
```

List the Pods in the cluster.

```
kubectl get pods
```

To verify the PVC is is still accessible within the Pod, you must gain shell access to your Pod. To start the shell session, execute the following command:

```
kubectl exec -it pvc-demo-pod -- sh
```

To verify that the text file still contains your message execute the following command:\

```
cat /var/www/html/index.html
```

The contents of the persistent volume were not removed, even though the Pod was deleted from the cluster and recreated.


## Task 3. Create StatefulSets with PVCs

In this task, you use your PVC in a StatefulSet. A StatefulSet is like a Deployment, except that the Pods are given unique identifiers.

### Release the PVC

Before you can use the PVC with the statefulset, you must delete the Pod that is currently using it. Execute the following command to delete the Pod:

```
kubectl delete pod pvc-demo-pod
```

Confirm the Pod has been removed.

```
kubectl get pods
```

You will get nothing.

### kubectl get pods

You will see the yaml file called "statefulset-demo.yaml" 

To create the StatefulSet with the volume, execute the following command:

```
kubectl apply -f statefulset-demo.yaml
```

You now have a statefulset running behind a service named "statefulset-demo-service".

### Verify the connection of Pods in StatefulSets

Use “kubectl describe” to view the details of the StatefulSet:

```
kubectl describe statefulset statefulset-demo
```

Note the event status at the end of the output. The service and statefulset created successfully.

Let's see the how many pods are running there should be 3 pod because 3 replicas.

```
kubectl get pods
```

To list the PVCs, execute the following command:

```
kubectl get pvc
```

Use “kubectl describe” to view the details of the first PVC in the StatefulSet:

```
kubectl describe pvc hello-web-disk-statefulset-demo-0
``` 

## Task 4. Verify the persistence of Persistent Volume connections to Pods managed by StatefulSets

To verify that the PVC is accessible within the Pod, you must gain shell access to your Pod. To start the shell session, execute the following command:

```
kubectl exec -it statefulset-demo-0 -- sh
```

Verify that there is no index.html text file in the /var/www/html directory.

```
cat /var/www/html/index.html
```
You will see "No such file or directory"

To create a simple text message as a web page in the Pod enter the following commands:

```
echo Test webpage in a persistent volume!>/var/www/html/index.html
```

```
chmod +x /var/www/html/index.html
```

Verify the text file contains your message.

```
cat /var/www/html/index.html
```
You will see the output "Test webpage in a persistent volume!"

Now leave the container.


```
kubectl delete pod statefulset-demo-0
```

List the Pods in the cluster.

```
kubectl get pods
```

You will see that the StatefulSet is automatically restarting the statefulset-demo-0 Pod.

Note: You need to wait until the Pod status shows that it is running again.

Connect to the shell on the new statefulset-demo-0 Pod.

```
kubectl exec -it statefulset-demo-0 -- sh
```

 Verify that the text file still contains your message.

```
cat /var/www/html/index.html
```
You will see the output "Test webpage in a persistent volume!"

The StatefulSet restarts the Pod and reconnects the existing dedicated PVC to the new Pod ensuring that the data for that Pod is preserved.

This Concludes our lab demonstration for “Configuring Persistent Storage for Google Kubernetes Engine”.