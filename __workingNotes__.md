**Configure AKS cluster credentials in local system...**

`az aks get-credentials --resource group <resource group name> --name <cluster name>`<br>OR<br>`az aks get-credentials -g <resource group name> -n <cluster name>`

Here, we want to configure our context (i.e. the work environment context) so that implicit references to a Kubernetes cluster resolve with the above given cluster. We give resource name merely to locate and identify the cluster.

I have created the cluster `aks-demo-pg` within the resource group `aks-pg-rg`. I shall be using these names for the configuration.

Results...

az module gives the following warning:

`az : WARNING: Merged "aks-demo-pg" as current context in C:\Users\prana\.kube\config`

This means the `config` file here is currently set to refer to the cluster `aks-demo-pg`.

**View nodes in the cluster...**

`kubectl get nodes`

Results...

```
NAME                                STATUS   ROLES   AGE    VERSION
aks-agentpool-19544217-vmss000003   Ready    agent   136m   v1.25.6
```

`kubectl get nods -o wide` gives additional information about the nodes, such as the container runtim used
