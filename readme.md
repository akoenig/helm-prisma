# Prisma: Helm Chart

A Helm Chart for installing Prisma on a Kubernetes cluster easily.

## Motivation

[Helm](https://helm.sh/) is a package manager for Kubernetes clusters. Think of Homebrew / APT for Kubernetes. It helps to bundle Kubernetes workloads for easy deployment, rollback or removal.

## Prerequisites

* Installed [helm CLI](https://docs.helm.sh/using_helm/#installing-helm)
* Kubernetes cluster (e.g. [minikube](https://github.com/kubernetes/minikube) if you want to give test drive this Chart locally)

## Installing the Prisma server

**Important:** Please note that this Chart doesn't ship with a bundled database system. You have to make sure to provide the database instance you want to use with Prisma manually. For example, if you want to use MySQL, installing it via Helm is pretty straightforward: [Kubeapps Hub: MySQL](https://hub.kubeapps.com/charts/stable/mysql).

First of all, we have to clone this repository:

```sh
git clone git@github.com:akoenig/helm-prisma.git
```

### Configuration

The chart comes with a default configuration you can find in the `values.yaml` file. Please copy that file and name it `values.custom.yaml`. Adjust this template to fit your needs (especially the database section).

### Deployment

After configuring the Chart, you can deploy the Prisma server via:

```sh
helm install --name prisma -f ./values.custom.yaml .
```

Awesome, you have successfully installed a Prisma server via Helm! You can check if everything went smoothly by executing:

```
kubectl get pods,services,configmaps --namespace prisma
```

You should see that the Prisma pod is running, an internal load balancer and the respective config map has been created.

## Accessing the Prisma server

As you have seen in the previous section, we haven't installed an Ingress which would be responsible for exposing the Prisma server to the Internet. The Prisma server is only accessible by applications which are also running on the Kubernetes cluster. You may ask: "Okay, how is it even possible to deploy my Prisma service to it?" - Great question, you can't in fact. But don't panic, `kubectl` comes with a nice feature called `port-forward`. With that it is possible to map a local port to the Prisma server running on the Kubernetes cluster so that you can interact with the Prisma server as if it would be installed on your local machine.

First of all, you have to identify the name of the Pod:

```sh
kubectl get pod --namespace prisma
```

Use the displayed name for establishing the port forwarding:

```sh
kubectl port-forward --namespace prisma <name-of-the-pod> 4467:4466
```

Now, you can use the Prisma CLI to deploy your Prisma service as if it would run locally. An example `prisma.yml`:

```yml
endpoint: http://localhost:4467/myservice/production
datamodel: datamodel.graphql
```

In fact, a `prisma deploy` would deploy the service to the Prisma server which is running on the Kubernetes cluster then.

## Uninstalling the Prisma server

```sh
helm del --purge prisma
```

## Author

MIT © [André König](https://andrekoenig.de)
