# Knative Lab

Getting started with Knative on Google Kubernetes Engine.

<font size="+5">🎯</font> Aim:

- Able to serve an API end point, using Knative (dynamic scaling up/down)
- Able to authenticate access to such end point
- Logging
- Accessing the end point via a DNS name (e.g. `something.i.hold`)
- `https://` access, only
- "GitOps" approach, i.e. changes to the end point come via changes to this repo

## Sources

- knative docs > [Installing Knative](https://github.com/knative/docs/blob/master/install/README.md)


## Requirements

You have:

- a Google Cloud account
  - GKE enabled within that project
  - Istio enabled
- and `gcloud` installed

**GKE settings (at creation of the cluster):**

In "advanced settings":

- Networking > VPC-native > [x] Enable VPC native (using alias IP) <sub>[details](https://cloud.google.com/kubernetes-engine/docs/how-to/alias-ips?hl=en_US&_ga=2.193848357.-966989269.1542020118&_gac=1.155356873.1547648787.CjwKCAiAyfvhBRBsEiwAe2t_i2Sx9xEckJFlzuzmzZYqnJwTjM2cwjU3AlxjppH9MPEtQa5w7QnXvhoCU_EQAvD_BwE)</sub>
  - this is "soon going to be default" so no reason not to
- Load balancing > [ ] Enable HTTP load balancing; you can likely switch this off since we're using Istio. <font color=red>tbd. try out</font>
- Additional features > [x] Try the new Stackdriver beta Monitoring and Logging experience
- [x] Enable Istio

### Command line tools

Have [gcloud installed](https://cloud.google.com/sdk/install) (Google Cloud docs). 

```
$ gcloud init     # select right GCP account and project
```

Configure cluster access for kubectl <sub>[source](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl)</sub>:

```
$ gcloud container clusters get-credentials cluster-1
...
```

You should now be able to reach the cluster:

```
$ kubectl get nodes
NAME                                       STATUS    ROLES     AGE       VERSION
gke-cluster-1-default-pool-835ba868-bmss   Ready     <none>    7m        v1.10.9-gke.5
gke-cluster-1-default-pool-835ba868-rh6c   Ready     <none>    7m        v1.10.9-gke.5
gke-cluster-1-default-pool-835ba868-v30c   Ready     <none>    7m        v1.10.9-gke.5
```


