
# Monitoring Rook with Prometheus and Grafana on GKE

This project is to easily bring up Prometheus and Grafana to monitor Rook.

## Getting Started and Documentation

This guide will guide you to install Kubernetes using GKE, deploy Rook and monitoring using Prometheus and Grafana.

We will be using [gcloud](https://cloud.google.com/sdk/gcloud/) to access the Google Cloud API. Please install it on your machine if you havent set it up.

## Installing Kubernetes with GKE

#### Login to Google Cloud

To login to Google Cloud, you can use `gcloud auth login --no-launch-browser`. This will provide a link that you can go to in order to authenticate to Google. Once authenticated, you will be provided with a verification code which will need to be provided.

```console
$ gcloud auth login --no-launch-browser
Go to the following link in your browser:

    https://accounts.google.com/o/oauth2/auth?redirect_uri=...

Enter verification code:
```

#### Set project ID and availability zone

Once authenticated, you should choose the project where your Kubernetes will be deployed on. You can list all available projects to choose or you can create a new one.

```console
$ gcloud projects list
PROJECT_ID            NAME              PROJECT_NUMBER
named-inn-167720      My First Project  663519429439
symbolic-wind-171217  gce-rook          460990813846
```

In my case, I would select the `gce-rook` project, whose ID is `symbolic-wind-171217`.

```console
$ gcloud config set project symbolic-wind-171217
Updated property [core/project].
```

Next you will select the availability zone to deploys your Kubernetes cluster resources on. I will select `us-west1-a`

```console
$ gcloud config set compute/zone us-west1-a
Updated property [compute/zone].
```

#### Create Kubernetes Cluster

Now we are ready to create our Kubernetes cluster. Here is how I created mine.

```console
$ gcloud container clusters create rook-k8s --cluster-version=1.8.4-gke.0 --image-type=UBUNTU -m n1-standard-4 --local-ssd-count 1
Creating cluster rook-k8s...done.
Created [https://container.googleapis.com/v1/projects/symbolic-wind-171217/zones/us-west1-a/clusters/rook-k8s].
kubeconfig entry generated for rook-k8s.
NAME      LOCATION    MASTER_VERSION  MASTER_IP       MACHINE_TYPE   NODE_VERSION  NUM_NODES  STATUS
rook-k8s  us-west1-a  1.8.4-gke.0     35.197.126.217  n1-standard-4  1.8.4-gke.0   3          RUNNING
```

As you can see, you can use `gcloud` to create your Kubernetes cluster. There are a different way to configure your cluster. Here is what each arguments I used means:

- `rook-k8s`: The name of the Kubernetes cluster. This is a positional argument
- `cluster-version`: The version of Kubernetes to deploy. You can get a list of supported Kubernetes versions by running `gcloud container get-server-config`.
- `image-type`: The node image to use. Rook can only be deployed on `UBUNTU` image due to kernel module limitation on the other image type.
- `machine-type`: Also provided with the `-m` argument. This specify the flavor of node to use. To list all available machine types, run `gcloud compute machine-types list`.
- `local-ssd-count`: Adds a number of local ssd volumes for each node.

Once deployed, `gcloud` will automatically configure your `kubeconfig` options so that you can start accessing your Kubernetes cluster using `kubectl`.

#### Create `clusterrolebinding` for your user

Once the cluster is created, your user will only have limited priviledges. To give admin privildge, create a `clusterrolebinding` as follow. Replace the provide email with yours:

```console
$ kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=user@gmail.com
```

## Rook

To deploy Rook, run the following:

```console
$ kubectl create -f manifests/rook-operator/rook-operator.yaml
namespace "rook-system" created
serviceaccount "rook-operator" created
clusterrolebinding "rook-operator" created
deployment "rook-operator" created
clusterrole "rook-operator" created
```

```console
$ kubectl create -f manifests/rook/rook-cluster.yaml
namespace "rook" created
cluster "rook" created
```

```console
$ kubectl create -f manifests/rook/rook-storageclass.yaml
pool "replicapool" created
storageclass "rook-block" created
```

To get more information about [Rook](https://github.com/rook/rook), see the [Rook Documentation](https://rook.github.io/docs/rook/master).

## Prometheus and Grafana

Now that Rook is deployed, you can deploy Prometheus and Grafana to monitor it.
The `deploy-monitoring` script is provided to deploy both Prometheus and Grafana and have it monitor Rook.

After you have deployed it, you can run the following to get the URL to the grafana dashboard:

```console
$ echo "http://$(kubectl -n monitoring  get svc grafana -o jsonpath={.status.loadBalancer.ingress[0].ip}):3000"
```

# Contributors

Special thanks to [@blingwang](https://github.com/blingwang) for providing the manifests for both Prometheus and Grafana and as well as the `deploy-monitor` and `teardown-monitoring` scripts.
