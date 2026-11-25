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

## Stretch tasks

### Batch Job and where its files go
The Job generator-batch completed (1/1) and logged: "Wrote 7 days to /data/: 3192 rates, 284 schedules, 259 bookings". /data is an emptyDir volume, which lives only as long as the pod. Once the Job's pod finished, the files were gone. To keep them, the Job should write to a PersistentVolume or Cloud Storage.

### Nonexistent image tag (v2)
kubectl set image deployment/generator generator=generator:v2 created a new pod stuck in ErrImagePull / ImagePullBackOff ("repository does not exist or may require docker login"). The old pod kept running, because the rolling update only removes old pods once new ones are ready. kubectl rollout undo fixed it.

### OOMKilled
With requests and limits set to 8Mi, the container was killed by the kernel: Last State Terminated, Reason OOMKilled, Exit Code 137, and it restarted repeatedly. Unlike the v2 case, the old pod was replaced, because the new pod briefly started before running out of memory and there is no readiness probe. kubectl rollout undo restored the original 64Mi/128Mi values. Lesson: limits that are too low crash the app, and a readiness probe would have protected the running version.

### Standard or Autopilot?
For this course I would choose GKE Standard. We run one small zonal cluster for the whole semester and scale it to 0 nodes between sessions, so we pay nothing while idle, and the free tier covers the fee for one zonal cluster. Session 5 adds Kafka, which needs stable nodes and persistent disks that we control. Autopilot would remove node management and bill per pod request, which suits many small or bursty workloads, but it gives less control over node size and is not the best fit for a long-running stateful system like Kafka.
