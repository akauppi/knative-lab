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
- Google CodeLabs > [Using Knative to deploy serverless applications to Kubernetes](https://codelabs.developers.google.com/codelabs/knative-intro/#0) <font size="+2">🥇</font>
  
## Requirements

You have:

- a Google Cloud account
  - GKE enabled within that project
  - Istio enabled
- and [`gcloud` installed](https://cloud.google.com/sdk/install)

**GKE settings (at creation of the cluster):**

In "advanced settings":

- `Networking` > `VPC-native` > `[x] Enable VPC native (using alias IP)` <sub>[details](https://cloud.google.com/kubernetes-engine/docs/how-to/alias-ips?hl=en_US&_ga=2.193848357.-966989269.1542020118&_gac=1.155356873.1547648787.CjwKCAiAyfvhBRBsEiwAe2t_i2Sx9xEckJFlzuzmzZYqnJwTjM2cwjU3AlxjppH9MPEtQa5w7QnXvhoCU_EQAvD_BwE)</sub>
  - this is "soon going to be default" so no reason not to
- `Load balancing` > `[ ] Enable HTTP load balancing`; you can likely switch this off since we're using Istio. <font color=red>tbd. try out</font>
- `Additional features` > `[x] Try the new Stackdriver beta Monitoring and Logging experience`
- `[x] Enable Istio`

Kubernetes 1.11 is required for Knative 0.3.0 and above (I chose latest, `1.11.6-gke.2`). If you miss the Kubernetes version, the cluster is upgradable (see later).

**Permissive or strict mTLS mode?**

See [here](https://istio.io/docs/reference/config/istio.authentication.v1alpha1/#MutualTls-Mode):

- "permissive" allows http as well as https (right?), and does not require requests to be carrying a certificate

><font color=red>Confirm the above, once more experienced.</font>


### Command line tools

Have [gcloud installed](https://cloud.google.com/sdk/install) (Google Cloud docs). 

```
$ gcloud init     # select right GCP account and project
```

Install `kubectl` (note: this likely means you shouldn't have a non-GCP `kubectl` installed): 

```
$ gcloud components install kubectl
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

## Installing Knative

With `kubectl` pointing to a cluster with Istio installed, we can install Knative on it.[^1]

[^1]: Google has something called [GKE Serverless addon](https://docs.google.com/forms/d/e/1FAIpQLSdG5cCIiHhkW7srw9MWvdiLEsLXwJES1R3lnKgAn-opy3_iuQ/viewform) in the works; it may be a better way to get started, but requires opt-in approvement for now (Jan-19).

### Prime the cluster

Give your account `cluster-admin` role on the cluster (necessary to install Knative):<sub>[source](https://codelabs.developers.google.com/codelabs/knative-intro/#2)</sub>

```
$ kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole=cluster-admin \
  --user=$(gcloud config get-value core/account)
...
clusterrolebinding.rbac.authorization.k8s.io "cluster-admin-binding" created
```

<!-- disabled (we don't use it in install)
### Pick the version

Let's see the latest version from [releases](https://github.com/knative/serving/releases) (GitHub).

>0.3 is the first release of our new schedule of releasing every 6 weeks.

>Kubernetes 1.11 is now required
-->

### Install

```
$ kubectl apply --filename https://storage.googleapis.com/knative-releases/build/latest/release.yaml
```
<sub>[source](https://github.com/knative/docs/blob/master/build/installing-build-component.md#adding-the-knative-build-component)</sub>

Check that Knative stuff is up:

```
$ kubectl get pods --namespace=knative-serving
NAME                          READY     STATUS    RESTARTS   AGE
activator-598b4b7787-42lrg    2/2       Running   0          4m
autoscaler-5cf5cfb4dc-vqllk   2/2       Running   0          4m
controller-7fc84c6584-4bh4q   1/1       Running   0          4m
webhook-7797ffb6bf-qv7fr      1/1       Running   0          4m
```

:)

### Which version did we get?

We installed `latest` - what did we end up with?

```
$ kubectl describe deploy controller --namespace knative-serving
...
    Image:      gcr.io/knative-releases/github.com/knative/serving/cmd/controller@sha256:5a5a0d5fffe839c99fc8f18ba028375467fdcd83cbee9c7015c1a58d01ca6929
...
```

Now we're going a bit to the ditch. Hang on. 🧗‍♀️

Pick the image URL. Cut it at the `@sha256:`.

```
$ gcloud container images list-tags gcr.io/knative-releases/github.com/knative/serving/cmd/controller | grep 5a5a0d
5a5a0d5fffe8  gke-1.11.6,latest,v0.3.0                                1970-01-01T02:00:00
```

Here, we caught the image is tagged `latest` and `v0.3.0` so that's what we got. :)

---

>Note: This was inspired by [these instructions](https://github.com/knative/docs/blob/master/install/check-install-version.md).

---

## Then what?

We'll get back to the [Knative codelabs #6](https://codelabs.developers.google.com/codelabs/knative-intro/#6) and follow their sample.

```
$ kubectl apply -f helloworld.yaml
service.serving.knative.dev "helloworld" created
```

*(you should now follow the codelabs page)*

Didn't get a response to the `curl` command?

```
$ curl -H "Host: helloworld.default.example.com" http://35.228.134.203 
curl: (7) Failed to connect to 35.228.134.203 port 80: Connection refused
```

><font color=red>Fix.</font>




## Appendices

### Upgrading Kubernetes

GKE tries to make upgrading Kubernetes easy. Let's collect here experiences about doing so in practise.

#### 22-Jan-2019 (1.10.x -> 1.11.6-gke.2)

- took ~15min but really smooth

