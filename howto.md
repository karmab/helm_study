
based on [this](https://docs.helm.sh/using_helm/#quickstart-guide)
and [this](https://www.influxdata.com/blog/packaged-kubernetes-deployments-writing-helm-chart) for charts creation 

1 - client install

client is provided as a go binary
for instance in OSX:

```
wget [AA](https://kubernetes-helm.storage.googleapis.com/helm-v2.6.0-darwin-amd64.tar.gz)
tar zxvf 
```

2 - server install

once the client is installed, run (beeing logged in a cluster)

\# the following should show tiller ( which is the server part )

```
kubectl get pod  --namespace=kube-system
```

3 - installing server side (tiller)

command is normally 

```
helm init
```

but i hit this issue with *helm ls*

``` 
Error: User "system:serviceaccount:kube-system:default" cannot list configmaps in the namespace "kube-system". (get configmaps)
```

as reported here

- https://github.com/kubernetes/helm/issues/2687
- https://github.com/heptio/aws-quickstart/issues/75

fix: use a proper sa and cluster role binding, as kubernetes 1.6 is a more secure cluster by default

````
kubectl create -f helm_sa.yml
helm init --service-account helm
````

4 - deploying a chart

Tiller stores its data in Kubernetes ConfigMap

```
kubectl create namespace prout
helm install stable/mysql
```


client side testing of the release:

```
kubectl get secret --namespace default pouring-mongoose-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo
kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il
apt-get update && apt-get install mysql-client -y
mysql -h pouring-mongoose-mysql -pYUw1b8oQdg
```

5- fun features

- helm install stable/mysql  (use --name to force the name of the "release")
- helm inspect stable/drupal (to find available parameters)
- helm install stable/mysql --set mysqlRootPassword=unix1234
- helm rollback pining-abalone 1
- helm search
- helm get values $RELEASE (to check applied parameters)
- helm repo list
- helm repo add dev https://example.com/dev-charts

6- creating custom charts
helm create kdummy
helm package kdummy
helm install ./kdummy-0.1.0.tar.gz



7- deploy on openshift

we will deploy in a dedicated namespace ( with its own sa)

```
oc new-project helm
oc create serviceaccount helm --namespace=helm
docker exec -it origin oadm policy add-cluster-role-to-user cluster-admin system:serviceaccount:helm:helm
```
note every command should be prefixed with --tiller-namespace but we will rather use the following env variable
export TILLER_NAMESPACE=helm




TODO:

- deploy on openshift
- study correct role to give to helm sa https://github.com/kubernetes/helm/blob/master/docs/service_accounts.md
- deploy helm in a dedicated namespace? DONE
  Tiller can be installed into any namespace. By default, it is installed into kube-system. You can run multiple Tillers provided they each run in their own namespace.
- find why helm deploys replica sets
- patch helm to report info when running in a secure cluster ?
- report when a release is installed but pod not running ?
- leverage the search feature (in openshift templates) and browse repositories
- leverage the info print when deploying a release
- leverage the status
- write a sample chart to deploy later on openshift
- write a helm plugin?
- investigate this error for stateful set example : cannot find volume plugin for alpha provisioning
  https://github.com/openshift/origin/issues/12676
