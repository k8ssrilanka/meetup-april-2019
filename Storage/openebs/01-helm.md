### Helm

#### Install Helm Client

Helm comes preconfigured with an installer script that automatically grabs the latest version of the Helm client and installs it locally. Fetch the script by running the following command:
```curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh```

Next, run the following commands to get the Helm client installed:
```chmod 700 get_helm.sh```
```./get_helm.sh```

Now initialize helm:
```helm init```

#### Install Tiller

```kubectl create serviceaccount --namespace kube-system tiller```

```kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller```

```kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'```

```helm init --service-account tiller --upgrade```

##### Verify the installation

```kubectl get deployments -n kube-system```

you should see a tiller deployment running.

Additionally, you can verify the installation by running

```helm version```