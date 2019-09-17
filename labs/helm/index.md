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

You will now see that `charts` has a new file 

```
ls charts/counter/charts 
```

Output should be similar to:  
```
mariadb-4.4.2.tgz
```

** Step 2:** Now that we have downloaded the dependent chart, create a new release.

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

**Step 8:** Delete a release

`helm delete counter --purge`

## Congrats!

