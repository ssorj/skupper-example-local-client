# Skupper local client

[![main](https://github.com/ssorj/skupper-example-local-client/actions/workflows/main.yaml/badge.svg)](https://github.com/ssorj/skupper-example-local-client/actions/workflows/main.yaml)

#### Connect to a remote service from a local process

This example is part of a [suite of examples][examples] showing the
different ways you can use [Skupper][website] to connect services
across cloud providers, data centers, and edge sites.

[website]: https://skupper.io/
[examples]: https://skupper.io/examples/index.html

#### Contents

* [Overview](#overview)
* [Prerequisites](#prerequisites)
* [Step 1: Install the Skupper command-line tool](#step-1-install-the-skupper-command-line-tool)
* [Step 2: Access your Kubernetes cluster](#step-2-access-your-kubernetes-cluster)
* [Step 3: Set up your Kubernetes namespace](#step-3-set-up-your-kubernetes-namespace)
* [Step 4: Install Skupper in your Kubernetes namespace](#step-4-install-skupper-in-your-kubernetes-namespace)
* [Step 5: Deploy and expose the backend on Kubernetes](#step-5-deploy-and-expose-the-backend-on-kubernetes)
* [Step 6: Deploy and expose the frontend on your local machine](#step-6-deploy-and-expose-the-frontend-on-your-local-machine)
* [Step 7: Test the application](#step-7-test-the-application)
* [Accessing the web console](#accessing-the-web-console)
* [Cleaning up](#cleaning-up)
* [About this example](#about-this-example)

## Overview

This example is a multi-service HTTP application deployed across a
Kubernetes cluster and a bare-metal host or VM.

It contains two services:

* A backend service that exposes an `/api/hello` endpoint.  It
  returns greetings of the form `Hi, <your-name>.  I am <my-name>
  (<pod-name>)`.

* A frontend service that sends greetings to the backend and
  fetches new greetings in response.

The backend runs on Kubernetes and the frontend runs on your local
machine.  Skupper enables the frontend to connect to the backend
using a dedicated service network.

<!-- <img src="images/entities.svg" width="640"/> -->

## Prerequisites

* A working installation of Docker ([installation
  guide][install-docker]) or Podman ([installation
  guide][install-podman])

* The `kubectl` command-line tool, version 1.15 or later
  ([installation guide][install-kubectl])

* Access to a Kubernetes cluster, from [any provider you
  choose][kube-providers]

[install-docker]: https://docs.docker.com/engine/install/
[install-podman]: https://podman.io/getting-started/installation
[install-kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
[kube-providers]: https://skupper.io/start/kubernetes.html#providers

## Step 1: Install the Skupper command-line tool

The `skupper` command-line tool is the primary entrypoint for
installing and configuring Skupper.  You need to install the
`skupper` command only once for each development environment.

On Linux or Mac, you can use the install script (inspect it
[here][install-script]) to download and extract the command:

~~~ shell
curl https://skupper.io/install.sh | sh
~~~

The script installs the command under your home directory.  It
prompts you to add the command to your path if necessary.

For Windows and other installation options, see [Installing
Skupper][install-docs].

[install-script]: https://github.com/skupperproject/skupper-website/blob/main/docs/install.sh
[install-docs]: https://skupper.io/install/index.html

## Step 2: Access your Kubernetes cluster

The procedure for accessing a Kubernetes cluster varies by
provider. [Find the instructions for your chosen
provider][kube-providers] and use them to authenticate and
configure access.

[kube-providers]: https://skupper.io/start/kubernetes.html#providers

## Step 3: Set up your Kubernetes namespace

Use `kubectl create namespace` to create the namespace you wish
to use (or use an existing namespace).  Use `kubectl config
set-context` to set the current namespace for your session.

_**Console for hello-world:**_

~~~ shell
kubectl create namespace hello-world
kubectl config set-context --current --namespace hello-world
~~~

_Sample output:_

~~~ console
$ kubectl create namespace hello-world
namespace/hello-world created

$ kubectl config set-context --current --namespace hello-world
Context "minikube" modified.
~~~

## Step 4: Install Skupper in your Kubernetes namespace

The `skupper init` command installs the Skupper router and service
controller in the current namespace.

**Note:** If you are using Minikube, [you need to start `minikube
tunnel`][minikube-tunnel] before you install Skupper.

[minikube-tunnel]: https://skupper.io/start/minikube.html#running-minikube-tunnel

_**Console for hello-world:**_

~~~ shell
skupper init
~~~

_Sample output:_

~~~ console
$ skupper init
Waiting for LoadBalancer IP or hostname...
Skupper is now installed in namespace 'hello-world'.  Use 'skupper status' to get more information.
~~~

## Step 5: Deploy and expose the backend on Kubernetes

Use `kubectl create deployment` to deploy the backend service
in `hello-world`.

_**Console for hello-world:**_

~~~ shell
kubectl create deployment backend --image quay.io/skupper/hello-world-backend
skupper expose deployment/backend --port 8080
~~~

_Sample output:_

~~~ console
$ kubectl create deployment backend --image quay.io/skupper/hello-world-backend
deployment.apps/backend created

$ skupper expose deployment/backend --port 8080
deployment backend exposed as backend
~~~

## Step 6: Deploy and expose the frontend on your local machine

Use the `skupper gateway init` command to set up a gateway
router on your local machine.  Use the `--type docker` option to
run the router in a Docker container.

Use the `skupper gateway forward` command to make the backend
service available on local port 8081.

Run the frontend service on your local machine.  Configure
it to connect to the backend service at local port 8081.

_**Console for hello-world:**_

~~~ shell
skupper gateway init --type docker
skupper gateway forward backend 8081
(cd frontend && python python/main.py --host localhost --port 8080 --backend http://localhost:8081) &
~~~

_Sample output:_

~~~ console
$ skupper gateway init --type docker
Skupper gateway: 'fancypants-jross'. Use 'skupper gateway status' to get more information.

$ skupper gateway forward backend 8081
2022/08/29 16:59:05 CREATE io.skupper.router.tcpListener fancypants-jross-ingress-backend:8080 map[address:backend:8080 name:fancypants-jross-ingress-backend:8080 port:8081 siteId:28b1dbd3-c4cd-4e49-a955-83328155ab53]

$ (cd frontend && python python/main.py --host localhost --port 8080 --backend http://localhost:8081) &
INFO:     Started server process [1710891]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://localhost:8080 (Press CTRL+C to quit)
~~~

## Step 7: Test the application

Now we're ready to try it out.  Use `curl` or a similar tool to
request the frontend `/api/health` endpoint.

_**Console for hello-world:**_

~~~ shell
curl http://localhost:8080/api/health
~~~

_Sample output:_

~~~ console
$ curl http://localhost:8080/api/health
OK
~~~

If everything is in order, you can now access the web interface by
navigating to `http://localhost:8080/` in your browser.

## Accessing the web console

Skupper includes a web console you can use to view the application
network.  To access it, use `skupper status` to look up the URL of
the web console.  Then use `kubectl get
secret/skupper-console-users` to look up the console admin
password.

**Note:** The `<console-url>` and `<password>` fields in the
following output are placeholders.  The actual values are specific
to your environment.

_**Console for hello-world:**_

~~~ shell
skupper status
kubectl get secret/skupper-console-users -o jsonpath={.data.admin} | base64 -d
~~~

_Sample output:_

~~~ console
$ skupper status
Skupper is enabled for namespace "hello-world" in interior mode. It is connected to 1 other site. It has 1 exposed service.
The site console url is: <console-url>
The credentials for internal console-auth mode are held in secret: 'skupper-console-users'

$ kubectl get secret/skupper-console-users -o jsonpath={.data.admin} | base64 -d
<password>
~~~

Navigate to `<console-url>` in your browser.  When prompted, log
in as user `admin` and enter the password.

## Cleaning up

To remove Skupper and the other resources from this exercise, use
the following commands.

_**Console for hello-world:**_

~~~ shell
kill $(ps -ef | grep 'python python/main\.py' | awk '{print $2}') 2> /dev/null
skupper gateway delete
skupper delete
kubectl delete service/backend
kubectl delete deployment/backend
~~~

## Next steps

Check out the other [examples][examples] on the Skupper website.

## About this example

This example was produced using [Skewer][skewer], a library for
documenting and testing Skupper examples.

[skewer]: https://github.com/skupperproject/skewer

Skewer provides some utilities for generating the README and running
the example steps.  Use the `./plano` command in the project root to
see what is available.

To quickly stand up the example using Minikube, try the `./plano demo`
command.
