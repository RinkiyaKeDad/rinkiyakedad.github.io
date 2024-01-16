---
title: "Pod Health With Liveness Probes"
read_time: true
date: 2021-05-13T20:12:05+05:30
draft: false
tags:
 - kubernetes
 - bash
 - devops
---

Kubernetes relies on [Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) to determine the health of a Pod container. A probe can be understood simply as a periodical diagnostic performed by the [kubelet](https://dev.to/rinkiyakedad/introduction-to-kubernetes-55o7) on the container.

In this short article, I'm going to show you a liveness probe in action. Liveness probes are used to check if a Pod is healthy (running as expected) or not. It simply acts as a check for Kubernetes to know when it should restart the container.

## Without Liveness Probe

Let us create a simple nginx pod and see what happens when we mess with it without using a Liveness probe. Use the following configuration to create your pod:

```
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
spec:
  containers:
    - name: my-nginx-container
      image: nginx:alpine
```

If we port-forward this container we will notice that it is serving the nginx starter page. Ideally, for this case, we would want Kubernetes to restart this pod if we are not able to reach this page. Let us see if that is the case currently or not.
 
Launch a shell inside the container using [kubectl exec](http://rinkiyakedad.github.io/kubectl-exec-is-so-cool):

```
kubectl exec my-nginx -it sh
```

Then navigate to the directory which has the HTML file responsible for the nginx starter page:

```
cd /usr/share/nginx/html
```

Do a `ls` here and you'll see the following output:

```
50x.html    index.html
```

Now let us delete this `index.html` file:

```
rm -rf index.html
```

Return to your original shell by typing `exit` in the current one.

Now do `kubectl get pods`. You will see that even though our pod is now not functioning as expected (because we deleted the index.html file), Kubernetes has still not restarted the pod. And to be honest that shouldn't be a surprise. Why would Kubernetes know that we care about that particular `index.html` file unless we explicitly tell it?

## Enter the Liveness Probe

This is where the Liveness Probe comes to the rescue. Update your pod configuration with this:

```
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
spec:
  containers:
    - name: my-nginx-container
      image: nginx:alpine
      livenessProbe:
        httpGet:
            path: /index.html
            port: 80
```

> You'll need to delete your previous pod before creating a new one using this config. 

Now repeat the same steps and delete the `index.html` file again. This time after deleting the index.html file don't immediately type `exit` to leave the shell, instead, wait about 10 secs (which is the default time gap for regular checks) and you automatically exit the terminal. This is because the container inside which we had opened our terminal got restarted.

You can verify this by running `kubectl get pods` and you should see something like:

```
NAME       READY   STATUS    RESTARTS   AGE
my-nginx   1/1     Running   1          4m24s
```

The 1 in the restarts column confirms that the **container** in our pod got restarted.

So what exactly did the liveness probe do here? 
The `httpGet` action sent a GET request to the container. We told the liveness probe to check for `index.html` on port 80 and if we don't get a valid status code for our request then restart the pod. 

This is useful for checking containers running an API or something like nginx serving static files.

This was it for this article. Hope you liked it and thanks for reading :)
