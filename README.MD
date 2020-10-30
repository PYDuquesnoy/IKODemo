

# IKO Demo on Docker for Windows



As of August 2020, the IRIS Kubernetes Operator (IKO) is still in demo, and not available from any docker registry. 

For this reason, this tutorial uses a docker account and pushes there the IKO container. Feel free to replace it with your own through this tutorial:



Docker Registry for this Demo

| Docker User     | Docker Password |
| --------------- | --------------- |
| intersystemsicm | kq2Nh@!]i_jT@#2 |



The IKO Operator has already been pushed to this Registry with

```
docker push intersystemsicm/iris-operator:v2.0.0
```



Note: 

This demo can be run on GCP as well. The necesary changes for GCP are:

* install (git clone or Upload) the demo files on the GCP Cloud Shell VM.
* Use a iris-ssd-sc-gke.yaml Storage class.
* Create a gcloud cluster (using the gcloud API or the GCP portal) before running this code



## Kubernetes Preparation

In Docker Desktop Icon, Click on Settings, and choose Kubernetes. 

Add the Check for "Enable Kubernetes" (v.16.5). + Apply and Restart. 

### Install Helm

Helm is the preferred method to deploy the InterSystems Kubernetes Operator to the Cluster. Following steps use chocolatey to install helm:

in PowerShell with Admin Privileges, Install Chocolatey

```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

Now Install helm:

```
choco install kubernetes-helm
```



### Deploy the Kubernetes Dashboard to the Cluster

Kubernetes offers a browser based interface to monitor the state of the cluster, but it is not started by default. Following Steps allow to deploy the UI and connect securely to it.

More information  about the WebUI can be found @

```
https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
```



#### Deploy webui with:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
```

Verify the correct deployment

```
kubectl get pod --all-namespaces
```

-> the kubernetes-dashboard-* should be in "running" state



#### Connect to WebUI

In order to create a secure channel to the Kubenetes Cluster, we need to start a local proxy:

```
kubectl proxy
```



To login to webui, we need to obtain a valid token, with following commands

```
kubectl describe secret -n kube-system | grep deployment -A 12

(
or if you don't have grep, the powerShell equivalence:
  kubectl describe secret -n kube-system | Select-String -Pattern deployment -Context 0,10
)
```

This returns a token to paste as authentication in the Webui:

```
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkFpT1gtamhoRlpRTlk2SnRsc1BzNUh5WXFpSWxDWWJLSkJxcGo5X3FneFkifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkZXBsb3ltZW50LWNvbnRyb2xsZXItdG9rZW4tbnA2eHAiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVwbG95bWVudC1jb250cm9sbGVyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiNDNlMTM1YzktMGUxNC00YzEzLWJjODQtYzUwNGMxNjM5MTY3Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmRlcGxveW1lbnQtY29udHJvbGxlciJ9.urEMOavrGeYNyEtDgEPVG5UbSmmAhO7ytNb1O4zzLrPd_DL7cF7gtZnpV0kdIcuGm1nJloBG6zBkpcrveI_-Itb9BkTZGZ32omQJaKpov0PN0njvqyRt2QAIQVYj6gRAi21wKxzwtTDkGCe5JRPY1LEuCQ46ojCLNDkMg0_465Q7Cy5xTMj7jSjVf_0aBBnxIYdegwSaE6W13QiA8GoUqMyP6fb8P8NGLwGZMkKIStjY53ifgez8X6FCawqpdUQnIUtL7VaMf3rVuD-dNVL01og8UBoQq9dDpZQ5XATx-4mycdyFQz9w3qUAPXrARBQfaCHbJnetpCatUk38pTcgKA
```



Connect to the Kubernetes Dashboard on this URL:

```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

And paste the token as authentication. 

Following dashboard is now visible:

![image-20200826225204955](C:\Users\pyd\AppData\Roaming\Typora\typora-user-images\image-20200826225204955.png)

## IKO Prerequisites

- Install iris_operator-2.0.0.218.0-unix.tar.gz in IKODemo folder

  ```
  cd IKODemo
  tar -xvzf iris_operator-2.0.0.218.0-unix.tar.gz
  ```

- Load the image locally in docker

  ```
  docker load -i iris_operator-2.0.0/image/iris_operator-2.0.0-docker.tgz
  ```

  Check if iris-operator is running:

  ```
  docker images |grep iris-operator
  (or for PoweShell:
  docker images | sls -Pattern iris-operator
  )
  ```

  ```
  docker logout
  docker login
  intersystemsicm / kq2Nh@!]i_jT@#2
  ```

- Verify/Update the following files to match the version of the operator uploaded to our docker repository: "intersystemsicm/iris-operator:v2.0.0"

   * iris-operator_2.0.0/iris-operator_installer.yaml:

     Change the "image" value to reference our repository (intersytstemsicm)
   
     ```
      containers:
           - name: operator
             image: intersystemsicm/iris-operator:v2.0.0
           imagePullPolicy: Always
        
     ```
   
  * iris-operator_1.0.0/chart/iris-operator/values.yaml:
  
    Change the "registry" tag to math "intersystemsicm"
  
    ```
    operator:
      registry: intersystemsicm
    ```
  
    

## Install IKO on Cluster (need to use Helm)

```
helm install intersystems iris_operator-2.0.0/chart/iris-operator
```

And Verify it has been installed correctly:

```
kubectl get pods
```



## Deploying the IRIS Cluster

At this point, we have deployed successfully on our cluster:

* The Kubernetes dashboard
* The InterSystems Kubernetes Operator

In order to deploy IRIS successfully, we need a few more preparational steps, like 

* Store our docker login in a K8s secret so the cluster can access docker and download the images
* Create a hash for the _SYSTEM Login in IRIS and Store this in a K8s secret,
* load the IRIS license in another K8s secret
* prepare some other components like storage classes.

All the items prepared here get used later on by the yaml file describing the iris-cluster.



### Create a Secret for Docker

create the secret, using the username and token provided through the WRC login at:

https://containers.intersystems.com

```
kubectl create secret docker-registry dockerhub-secret --docker-server=https://containers.intersystems.com --docker-username=duquesno --docker-password='REPLACEWITHYOURTOKEN'
```



### Create a Secret for the IRIS License Key

Load the key as a secret:

```
kubectl create secret generic iris-key-secret --from-file=./irislicense/iris.key
```



### Create Password Hash for IRIS Instances

In order to connect securely to the IRIS instances, this step allows to pre-configure the instances with a given password. InterSystems provides a small app (as a docker container) to generate Hashes for the password you choose. Only the Hash gets transferred and loaded to the instance.

Run  PasswordHash-1.0.docker.tar.gz, using password "Roche"

```
docker run --rm -it containers.intersystems.com/intersystems/passwordhash:1.0
Enter password: <password_entry>
Enter password again: <password_entry>
PasswordHash=a299aa705a2d4ff44cace36dff503d8c7056f35c,cilc6a5d
```

Copy this password hash to the cluster definition yaml file to use, for example singleiris4health.yaml:

```
apiVersion: intersystems.com/v1alpha1
kind: IrisCluster
metadata:
  name: iris4hlocal
spec:
  passwordHash: 'a299aa705a2d4ff44cace36dff503d8c7056f35c,cilc6a5d'
(...)
```



### Create the IRIS.CPF

Load a CPF config for compute nodes and data nodes (data.cpf and compute.cpf)

```
[StartUp]
SystemMode=my-iriscluster
[config]
globals= 0,0,512,0,0,0
gmheap= 262144
```

Load this information into a K8s configmap:

```
$ kubectl create cm iris-cpf --from-file data.cpf --from-file compute.cpf
```

### Create a Storage Class if needed

The distributed files use the default "hostpath" storage class available on Docker for Windows, (so nothing needs to be done here) but this would need to be adapted for other environments.  Following is an example of Storage Class for Google Cloud Provider:

iris-ssd-sc-gke.yaml

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: iris-ssd-storageclass
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```

Load the File

```
kubectl create -f iris-ssd-sc-gke.yaml
```



### Finally, deploy the Cluster

As a start, we deploy a simple single-node IRIS instance with the file: **singleiris4health.yaml**

```
kubectl apply -f singleiris4health.yaml
```

Verify that the Pod is running:

```
kubectl get pods
```

this returns the name of our IRIS pod: **iris4hlocal-data-0**

Also, use the Kubernetes Dashboard and take a moment to review everything that was created.

We can also use the command line to inspect some more:

```
kubectl get svc
```



### Connect to our Cluster

The cluster is running locally on our PC, but Kubernetes has created a network and assigned IPs and ports to each pod. The IKO has made sure the ports 52773 and 1972 can be accessed from the outside.

Starting a local port-forward to our **pod** running IRIS allows us to access the cluster from a browser:

```
kubectl port-forward pods/iris4hlocal-data-0 55773:52773
```

 And now, point a browser to the IRIS management portal, and use the password Hashed previously:

```
http://127.0.0.1:55773/csp/sys/UtilHome.csp
```



```
kubectl get svc
```



```
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)           
iris4hlocal        ClusterIP   10.96.54.250    <none>        1972/TCP,52773/TCP 
(...)
```



### Cleanup

Here is how we can cleanup before the next exercise:

One radical way is to restart Kubernetes from the docker-desktop menu. This clears everything, including the secrets, our deployments of IKO and the K8s dashboard....

Here is how to delete just the IRIS cluster:

```
kubectl delete -f singleiris4health.yaml
kubectl delete pvc --all
kubectl delete svc iris-svc
```



## Deploy an IRIS Cluster with Mirroring 

You can simply execute following template, which creates 2 mirrored Data Master Nodes and 1 arbiter node.(Note: The Arbiter is of version 2020.3, as it seems that the arbiter v2020.2 is missing /bin/bash and fails to start):

```
kubectl apply -f ./iris4healthmirror.yaml
```



## Deploy an IRIS Cluster with Application Servers

Following Template creates 1 Data Master node and 2 application servers:

```
kubectl apply -f .\iris4healthcluster.yaml
```



## Debugging

Sometimes a pod may not start up and fail. Here are some commands to investigate:

```
kubectl get pods
```

```
kubectl logs -p <pod-name>
```

```
kubectl describe pod <pod-name>
```

```
kubectl get pods <pod-name> -o yaml
```

