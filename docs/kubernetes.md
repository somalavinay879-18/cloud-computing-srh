# Session 4: Docker and Kubernetes

**Platform note:** My GCP free trial billing account was closed, so with my lecturer's approval I ran this workshop on minikube in Cloud Shell instead of GKE. The image was built inside minikube (eval $(minikube docker-env)) and referenced as generator:v1 instead of the Artifact Registry path. All kubectl steps are the same as on GKE.

## kubectl get pods -o wide
~~~
NAME                        READY   STATUS    RESTARTS   AGE    IP            NODE       NOMINATED NODE   READINESS GATES
generator-5cb89966b4-7grd5  1/1     Running   0          2m4s   10.244.0.8    minikube   <none>           <none>
~~~

## kubectl rollout history deployment/generator
~~~
deployment.apps/generator
REVISION  CHANGE-CAUSE
2         <none>
3         <none>

~~~

## What happened when I deleted the pod
When I deleted the generator pod, Kubernetes immediately created a new pod with a different name. The Deployment declares that 1 replica should always run, and its controller constantly compares the desired state with the actual state. When the pod disappeared, the actual count dropped to 0, so it started a replacement. This is self-healing: I describe the desired state, and Kubernetes keeps it true.

## Rollout and rollback
Changing MODE to air and RATE to 5 created revision 2 with a rolling update (the new pod started before the old one was terminated). kubectl rollout undo brought back the original settings as revision 3. I then restored the YAML file so the committed file matches the running state.
