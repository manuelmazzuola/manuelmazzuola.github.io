---
layout: post
show_badges: false
title: Deploy Helm Charts on a Kubernetes cluster in Go
description: How to programmatically deploy Helm Charts on a Kubernetes cluster using Go language
summary:  How to write a simple GO application that fetches a Helm chart from a repository and install it on a Kubernetes cluster
tags: [kubernetes, k8s, helm, golang, go, helm-chart]
---

- [What is Helm](#what-is-helm)
- [The Helm Go SDK](#the-helm-go-sdk)
- [Sync the Helm repository](#sync-the-helm-repository)
- [Authenticate to the K8S API server](#authenticate-to-the-k8s-api-server)
- [The install intent](#the-install-intent)
- [Locate and load the chart](#locate-and-load-the-chart)
- [Install the chart](#install-the-chart)
- [Conclusions](#conclusions)

# What is Helm
The basic idea of [Helm](https://helm.sh/) is to facilitate the deployment and manage complex
application workloads in [Kubernetes](https://kubernetes.io/).  

Helm is kinda like a package manager for Kubernetes, and the packages are so-called Helm Charts.
A Helm Chart is essentially the combination of a template file, that describes the Kubernetes
resources to deploy, and a values file used to valorize the former.
The Helm Chart templating system enables the reusability of the packages.  

# The Helm Go SDK
Helm comes with a handy CLI, a powerful command-line client for end-users, responsible
for managing all the lifecycle phases of a Chart.  
But, to programmatically install Helm Charts on a K8S cluster the CLI is not the solution
if the intention is to build a robust and predictable application, luckily, the Helm developers
designed the CLI as an interface for the [Go SDK](https://pkg.go.dev/helm.sh/helm/v3)
that I'll reuse for my purpose.

# Sync the Helm repository
Helm supports Charts storage in remote repositories, so the first thing to do
is to gather from the repository the information about the available charts.

I skip logging, error checking, and methods definitions to keep this blog post brief
and concise.  

Let's start by downloading the repository index file.

```go
import (
  "helm.sh/helm/v3/pkg/repo"
)

// repoEntry contains the name and the url of the repository
// settings contains the path to the directory where to store the repository index file
r, err := repo.NewChartRepository(repoEntry, getter.All(settings))

_, err = r.DownloadIndexFile()
```

Then I fetch the information about the charts stored in the repository
and cache the results into a dedicated folder.

```go
import (
  "helm.sh/helm/v3/pkg/repo"
)

var f repo.File
f.Update(repoEntry)
err := f.WriteFile(CacheFolderPath, 0644)
```

The before pieces of code replicate the Helm CLI commands:
  - `helm repo add repo_name repo_url`
  - `helm repo update`

# Authenticate to the K8S API server
The Helm Go SDK allows building an authenticated client instance by using the path of the
[kubeconfig](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)
file on the filesystem.  

In the following code, the `kube.GetConfig` method returns an implementation
of the `RESTClientGetter` interface, used to obtain a REST client configuration
based on the provided kubeconfig path.  
The REST client configuration contains everything is needed to dialog
with the Kubernetes server API.

```go
restClientGetter, err := kube.GetConfig(KUBECONFIG_PATH_ON_FS)
actionConfig := new(action.Configuration)

err = actionConfig.Init(restClientGetter, ...)
```

However, I can't leverage that method because I don't want to persist on the filesystem
the Kubernetes credentials I got from the outside.

After having done some research and opened an [issue](https://github.com/helm/helm/issues/9473)
on Github (I had to first search for similar issues, my bad), I finally ended up
writing my implementation of the `RESTClientGetter` interface,
[restclient.go](https://gist.github.com/manuelmazzuola/943db0cda93dfc435912968e7cf34126).

Then I build a `RESTClientGetter` by calling the `NewRESTClientGetter` method
passing to it the **kubeconfig content** directly.
```go
import (
  "helm.sh/helm/v3/pkg/action"
)


restClientGetter, err := NewRESTClientGetter(kubeconfigContent, namespace)
actionConfig := new(action.Configuration)

err = actionConfig.Init(restClientGetter, ...)
```

Due to an [issue](https://github.com/helm/helm/issues/9256) with the Helm SDK, I override the
[namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
value in my rest client builder.
The install command, which you'll see later, wrongly uses the `default` namespace,
so I fixed it by forcing the namespace to the value passed in the `NewRESTClientGetter` method.

# The install intent
After that, I instantiate a new Install action and configure it by populating
some of its attributes: the namespace where to create the resources and the release name.

```go
import (
  "helm.sh/helm/v3/pkg/action"
)

client := action.NewInstall(actionConfig)

client.Namespace = namespace
client.ReleaseName = releaseName
```

# Locate and load the chart
The next step is to locate from the repository synced before the chart to deploy and load it.

```go
import (
  "helm.sh/helm/v3/pkg/chart/loader"
)

cp, err := client.ChartPathOptions.LocateChart(chart, settings)
chartReq, err := loader.Load(cp)
```

# Install the chart
Once the chart is loaded, I can execute the deployment by calling the Run method and
passing to it the requested chart.

```go
err := client.Run(chartReq, nil)
```

All the above steps are required to imitate the Helm CLI command
`helm install name --namespace namespace chart`.

# Conclusions
I hope I've given you an idea of ​​how to use the Helm Go SDK to install
Helm Charts on a Kubernetes cluster.  
More can be done, if the deployed Chart exposes a web service,
you can use the K8S Go SDK to create a load balancer Service type to reach
the application from the internet.
