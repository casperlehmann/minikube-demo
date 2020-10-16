# Minikube Demo

Illustration of how to get start and expose a Kubernetes cluster on localhost in 2020.

Recent deprecations in the kubectl api have resulted in some difficulties when searching for information online. Namely, the kubectl run command no longer produces a deployment (it now only creates pods).

The [canonical answer](https://stackoverflow.com/questions/52890718/kubectl-run-is-deprecated-looking-for-alternative) on the topic describes the reason for these changes, but for the majority of newcomers to Kubernetes, that information is not what we need. We'd just like to know what to do instead.

This actually works:

``` bash
eval $(minikube docker-env)
docker build -t hello-py:v1 hello-py
kubectl create deployment hello-py1 --image=hello-py:v1
kubectl expose deployment hello-py1 --type=LoadBalancer --port=5000
minikube service hello-py1
```

In the above, I assume minikube is installed and running (`minikube start`). I also assume that kubectl is installed on the work machine. And of course, the root directory of this repo is your current working directory.

The commands, on by one:

 - `eval $(minikube docker-env)`: For the rest of the active session, all docker workloads will be handled by the docker deamon inside of minikube. Changing docker hosts means that containers running locally will be invisible to the docker command, but we will retain access to our local file system and other functions. In effect, the source code on our work station will be read and deployed inside the minikube container.
- `docker build -t hello-py:v1 hello-py`: Reads and build the content of the `hello-py` directory. The resulting image will be available in the currently active docker host. The image is tagged as `hello-py:v1`.
- `kubectl create deployment hello-py --image=hello-py:v1`: Create a deployment from the created image and name the deployment `hello-py`.
- `kubectl expose deployment hello-py --type=LoadBalancer -port=5000`: Create a `LoadBalancer` `service` for the `hello-py` deployment and expose the internal port 5000. Port 5000 is the port through which Flask is serving our app. This port is also exposed in the app's Dockerfile. We do not get to specify the port on which minikube will eventually host the app, as that happens automatically.
- `minikube service hello-py`: This minikube function automatically opens a browser window and routes us to the app on an automatically assigned port number.

## Start over

If for any reason you need to start over, simply delete the cluster and start again. Don't do this in production. Also, don't use minikube in production, probably.

```
minikube delete
minikube start
eval $(minikube docker-env)
```

Why would you want to start over? Well, one reason would be to work with compose and YAML files. Read on.

## Kubernetes deployment file

I'll have to continue working on this, but the point is to be able to continuously update the app in the docker host.

```
kubectl delete service hello; kubectl delete deployment hello
docker-compose -f docker-compose-dev.yml build
kompose convert
kubectl apply -f hello-deployment.yaml,hello-service.yaml
minikube service hello
```

## Skaffold

The smart way to do this is using the continuous development tool Skaffold.

`brew install skaffold`

```
eval $(minikube docker-env)
> skaffold init
[...]
Configuration skaffold.yaml was written
You can now run [skaffold build] to build the artifacts
or [skaffold run] to build and deploy
or [skaffold dev] to enter development mode, with auto-redeploy
> skaffold dev
Watching for changes...
```

This will make Kubernetes dynamically rebuild every time changes in the code demand it.

Minikube needs to run in the background, so we need a second terminal for that.

```
eval $(minikube docker-env)
minikube service hello
```

The only thing to remember is we are still responsible for supplying up-to-date yaml files, by running kompose when needed to regenerate configuration. So we might need to open a third terminal as well.

```
eval $(minikube docker-env)
kompose convert
```

## What you should not do

So to be clear, `kubectl run` is not what you are looking for, and this is not going to work:

``` bash
$ kubectl run hello-py --image=hello-py:v1 --port=8080
```

## Sources

- https://cloud.google.com/kubernetes-engine/docs/concepts/deployment
- https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/
- https://www.bogotobogo.com/DevOps/DevOps-Kubernetes-1-Running-Kubernetes-Locally-via-Minikube.php (outdated)
- https://www.abhishek-tiwari.com/local-development-environment-for-kubernetes-using-minikube/ (outdated)

- https://stackoverflow.com/questions/48077931/delete-all-the-contents-from-a-kubernetes-node
- https://cloud.google.com/kubernetes-engine/docs/concepts/deployment#:~:text=and%20stable%20hostnames.-,Creating%20Deployments,are%20evicted%20from%20their%20nodes.

- [Walkthrough: Minikube with local build](https://kubernetes.io/blog/2018/05/01/developing-on-kubernetes/)

- https://medium.com/@mukherjee.aniket/continuous-development-using-skaffold-with-local-kubernetes-cluster-with-hot-reload-61009e185258
