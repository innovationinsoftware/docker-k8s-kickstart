# Working with Helm

## Install Helm 

```
wget https://raw.githubusercontent.com/helm/helm/master/scripts/get -O - | bash
```

Now that Helm is installed we need to install the backend 

Create tiller service account
```
kubectl create serviceaccount tiller --namespace kube-system
```

Grant tiller cluster admin role 
```
kubectl create clusterrolebinding tiller-admin-binding --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
```

Initialize Helm to install tiller in your cluster 
```
helm init --service-account=tiller --upgrade
helm repo update
```

## Chart Operations for a demo Chart

**Step 1:** Download dependent charts

```
helm dependency update counter
```

You will now see that the `charts` directory has a new file 

```
ls charts/counter/charts 
```

Output should be similar to:  
```
mariadb-4.4.2.tgz
```

**Step 2:** Now that we have downloaded the dependant chart, create a new release.

`helm install --name counter charts/counter`

**Step 3:** List the release

`helm ls`

**Step 4:** Try the release out. Find the IP address of the service

`kubectl get svc`

**NOTE: The application can take a few minutes to come online, if it doesn't load wait 5 minutes and try again**

Load the 'External-IP' in a browser

You can also test with curl
```
curl <External-IP>
```

**Step 5:** Upgrade application

Let's upgrade our application by deploying a new release.
**NOTE: There is a bug with mariadb chart when using randomly generated passwords. To overcome this
we need to get the current password and save it as a variable to use when upgrading our release. 

Get password
```
kubectl get secret counter-mariadb -o yaml
```

In the `yaml` output you will see two encoded passwords, `mariadb-root` and `mariadb-password`, we need the mariadb-password. 
Decode the hashed password
```
echo '<mariadb-password>' | base64 --decode
```

Create a variable from the output
```
export DB_PASSWORD=<password from above> 
```

Now we can upgrade the release
```
helm upgrade --set image.tag=v0.0.2,mariadb.db.password=$DB_PASSWORD counter charts/counter
```

**Step 6:** Confirm application is being upgraded
```
kubectl get pods 
```
You should see pods being terminated and created
**NOTE: This may take a little while to occur**

**Step 7:** Confirm updated application is available    
Either load in a browser 'External-IP' or use `curl` to hit the ELB

**Step 8:** Rollback a release
Sometimes an update does not go as planned, in these situations you can rollback to an earlier revision.  
First run `helm history counter` to see the available revisions.     
You should see there are two revisions, let's rollback to revision 1. 
```
helm rollback counter 1
```

Now you will see the pods go into a `Terminating` and `ContainerCreating` status as the old `v2` pods are removed and the new `v1` pods are deployed. 

After a few minutes you should be able to run the `curl` command and see it is now displaying version 1.

**Step 9:** Delete a release, but not the history

`helm delete counter`

Now if you run `helm history counter` you will see it still exists. If you want to completely remove a release and it's history you can run
```
helm delete counter --purge
```

## Override Helm chart values at deployment 
Helm allows values to bet set at deployment time which makes charts more portable and customizable.   
In this lab you are going to deploy a simple `todo` app and use Helm to override some default vaules. 

**Step 1:** Let's start by adding a new repository which has a chart for our simple todo app.  
```
helm repo add bitnami https://charts.bitnami.com
```

**Step 2:** Now run `update` to index the charts in that repository 
```
helm repo update 
```

**Step 3:** After this we can install our app. 
```
helm install bitnami/mean --name todo
```

In the output you will see: 
```
1. Get the URL of your MEAN app by running:

  kubectl port-forward --namespace default svc/mean-mean 80:80
  echo "MEAN app URL: http://127.0.0.1:80/"
```

**Step 4:** Access application over ClusterIP
Now that you've deployed the app you can run `kubectl port-forward` from above output to create a tunnel from your local machine
to the cluster. This allows you to load the application on your local machine for testing without making it public available.

**Step 5:** Override `serviceType` to expose application publicly. 
Let's upgrade our application so that it is available externally. 
```
helm install bitnami/mean --name todo --set service.type=LoadBalancer
```

**Step 6:** Confirm application is accessible. 
Run the ouput from the following command to get the URL the application is running on and load in your browser. 

**Step 7:** Deploy a new release 
Now the app is available publicly let's deploy an updated release.   
```
helm upgrade mean --set service.type=LoadBalancer,image.tag=8.16.1-r33
```

**Step 8:** Confirm the new revision has been deployed
```
helm history todo 
``` 


## Congrats!

