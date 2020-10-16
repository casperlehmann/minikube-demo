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
