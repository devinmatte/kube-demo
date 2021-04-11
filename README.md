# kube-demo

## Install MiniKube

Follow the setup instructions on [MiniKube's Docs for your operating system](https://minikube.sigs.k8s.io/docs/start/)

## Start a cluster

Run

```
minikube start
```

To start your cluster. This mini cluster will allow you to operate as if you're using a full cluster.

## Interacting with Kube

If you already have kubectl on your system, it's recommended to run all commands as `minikube kubectl --` as to not conflict with your existing contexts.

To see the status of pods starting in your cluster
```
minikube kubectl -- get pods -A
```

You should see an output simular to this. This is showing all the systems that Kubernetes needs to run.
What's cool is that, all things that kube needs to operate pods, operates as pods inside kube. Yeah. Kinda cool right?

```
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   coredns-74ff55c5b-sqbtr            1/1     Running   0          86s
kube-system   etcd-minikube                      1/1     Running   0          93s
kube-system   kube-apiserver-minikube            1/1     Running   0          93s
kube-system   kube-controller-manager-minikube   1/1     Running   0          93s
kube-system   kube-proxy-4dt6g                   1/1     Running   0          86s
kube-system   kube-scheduler-minikube            1/1     Running   0          93s
kube-system   storage-provisioner                1/1     Running   0          97s
```

## Creating Pods

A pod is a collection of containers that run in the system. In the case of Minikube, there's only a single node in the cluster, but in a typical Kubernetes setup there would be many nodes that the pod could live on, and move between.

So we're going to create a pod to see kube in action

```
minikube kubectl -- create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4
minikube kubectl -- expose deployment hello-minikube --type=NodePort --port=8080
minikube kubectl -- get services hello-minikube
```

You should have a service available. Run `minikube service hello-minikube` to launch the server in a browser.

You'll see an output like
```
CLIENT VALUES:
client_address=172.17.0.1
command=GET
real path=/
query=nil
request_version=1.1
request_uri=http://127.0.0.1:8080/

SERVER VALUES:
server_version=nginx: 1.10.0 - lua: 10001

HEADERS RECEIVED:
accept=text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
accept-encoding=gzip, deflate, br
accept-language=en-US,en;q=0.9
connection=keep-alive
dnt=1
host=127.0.0.1:52302
sec-ch-ua=" Not A;Brand";v="99", "Chromium";v="90", "Google Chrome";v="90"
sec-ch-ua-mobile=?0
sec-fetch-dest=document
sec-fetch-mode=navigate
sec-fetch-site=none
sec-fetch-user=?1
upgrade-insecure-requests=1
user-agent=Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.61 Safari/537.36
BODY:
-no body in request-
```

This means the application is working.


## Managing Deployments

If we run `minikube kubectl -- get pods` we'll notice that we have 1 pod for `hello-minikube`

We're going to modify the deployment to ensure that the app has redundency.

```
minikube kubectl -- edit deployment hello-minikube
```

The deployment should open a yaml file in an editor, likely vim, vi or nano. You should scroll down to `spec`.

```yaml
spec:
    progressDeadlineSeconds: 600
    replicas: 1
```

Change `replicas` from `1` to `2` or even `3`.

Now run `minikube kubectl -- get pods` again. You'll see 2+ pods now.

```
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-6ddfcc9757-2g5j5   1/1     Running   0          3s
hello-minikube-6ddfcc9757-znksf   1/1     Running   0          6m
```

Now let's see what happens if we delete a pod. Open a second terminal window and run `minikube service hello-minikube`.

Open the page in a browser. Then while looking at the webpage, run `minikube kubectl -- delete pod hello-minikube-pod-name-here`. Replace the pod name with one of your pod names.

As quickly as you can refresh the page in the browser and you'll notice that it's still running. By having two pods we've created reduncency. One pod can die and the app will continue to operate. It also self heals. You told it it needs to always have 2 replicas, so it will ensure that is true by making a new pod.

## Cleanup

If you don't want the cluster to stick around after today, you can either just stop it, or delete it entirely.

```
minikube stop
```

```
minikube delete --all
```