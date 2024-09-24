# 01-ImagePullBackoff

When a kubelet starts creating containers for a Pod using a container runtime, it might be possible the container is in Waiting state because of ImagePullBackOff.

The status ImagePullBackOff means that a container could not start because Kubernetes could not pull a container image for reasons such as 

- Invalid image name or 
- Pulling from a private registry without imagePullSecret. 

The BackOff part indicates that Kubernetes will keep trying to pull the image, with an increasing back-off delay.

Kubernetes raises the delay between each attempt until it reaches a compiled-in limit, which is 300 seconds (5 minutes).

# 02-CrashLoopBackOff

When you see "CrashLoopBackOff," it means that kubelet is trying to run the container, but it keeps failing and crashing. After crashing, Kubernetes tries to restart the container automatically, but if the container keeps failing repeatedly, you end up in a loop of crashes and restarts, thus the term "CrashLoopBackOff." 

This situation indicates that something is wrong with the application or the configuration that needs to be fixed.

## Common Situations of CrashLoopBackOff

The CrashLoopBackOff error in Kubernetes indicates that a container is repeatedly crashing and restarting. Here are explanations of how the CrashLoopBackOff error can occur due to the specific reasons you listed:

### Misconfigurations

Misconfigurations can encompass a wide range of issues, from incorrect environment variables to improper setup of service ports or volumes. These misconfigurations can prevent the application from starting correctly, leading to crashes. For example, if an application expects a certain environment variable to connect to a database and that variable is not set or is incorrect, the application might crash as it cannot establish a database connection.

### Errors in the Liveness Probes

Liveness probes in Kubernetes are used to check the health of a container. If a liveness probe is incorrectly configured, it might falsely report that the container is unhealthy, causing Kubernetes to kill and restart the container repeatedly. For example, if the liveness probe checks a URL or port that the application does not expose or checks too soon before the application is ready, the container will be repeatedly terminated and restarted.

### The Memory Limits Are Too Low

If the memory limits set for a container are too low, the application might exceed this limit, especially under load, leading to the container being killed by Kubernetes. This can happen repeatedly if the workload does not decrease, causing a cycle of crashing and restarting. Kubernetes uses these limits to ensure that containers do not consume all available resources on a node, which can affect other containers.

### Wrong Command Line Arguments

Containers might be configured to start with specific command-line arguments. If these arguments are wrong or lead to the application exiting (for example, passing an invalid option to a command), the container will exit immediately. Kubernetes will then attempt to restart it, leading to the CrashLoopBackOff status. An example would be passing a configuration file path that does not exist or is inaccessible.

### Bugs & Exceptions

Bugs in the application code, such as unhandled exceptions or segmentation faults, can cause the application to crash. For instance, if the application tries to access a null pointer or fails to catch and handle an exception correctly, it might terminate unexpectedly. Kubernetes, detecting the crash, will restart the container, but if the bug is triggered each time the application runs, this leads to a repetitive crash loop.

# 03-pods-not-schedulable

In Kubernetes, the scheduler is responsible for assigning pods to nodes in the cluster based on various criteria. Sometimes, you might encounter situations where pods are not being scheduled as expected. This can happen due to factors such as node constraints, pod requirements, or cluster configurations.

1. Node Selector

Node Selector is a simple way to constrain pods to nodes with specific labels. It allows you to specify a set of key-value pairs that must match the node's labels for a pod to be scheduled on that node.
Usage: Include a nodeSelector field in the pod's YAML definition to specify the required labels.

```
spec:
    containers:
    - name: my-app
    image: my-image
    nodeSelector:
    disktype: ssd
```

2. Node Affinity

Node Affinity is a more expressive way to specify rules about the placement of pods relative to nodes' labels. It allows you to specify rules that apply only if certain conditions are met.
Usage: Define nodeAffinity rules in the pod's YAML definition, specifying required and preferred node selectors.

```
spec:
    containers:
    - name: my-app
    image: my-image
    affinity:
    nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
            - key: disktype
            operator: In
            values:
            - ssd
```

3. Taints

Taints are applied to nodes to repel certain pods. They allow nodes to refuse pods unless the pods have a matching toleration.
Usage: Use kubectl taint command to apply taints to nodes. Include tolerations field in the pod's YAML definition to tolerate specific taints.

```
kubectl taint nodes node1 disktype=ssd:NoSchedule
```

```
spec:
    containers:
    - name: my-app
    image: my-image
    tolerations:
    - key: disktype
      operator: Equal
      value: ssd
      effect: NoSchedule
```

4. Tolerations

Tolerations are applied to pods and allow them to schedule onto nodes with matching taints. They override the effect of taints.

Usage: Include tolerations field in the pod's YAML definition to specify which taints the pod tolerates.

```
spec:
  containers:
  - name: my-app
    image: my-image
  tolerations:
  - key: disktype
    operator: Equal
    value: ssd
    effect: NoSchedule
```
# 04-statefulset-pv

# 05-networkpolicy
