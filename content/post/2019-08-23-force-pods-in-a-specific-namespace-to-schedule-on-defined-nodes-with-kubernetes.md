---
title: "Force all pods in a specific namespace to schedule on defined hosts with Kubernetes"
date: 2019-08-23T12:37:00+01:00
draft: false
author: VaLouille
type: post
categories:
  - kubernetes
tags:
  - namespace
  - ns
  - pods
  - kops
  - PodNodeSelector
  - node-selector
  - taints
  - tolerations
  - kubernetes
---

Sometimes, we need to stick pods to specific hosts to isolate business critical workloads from others, or take advantage of different server types. We can do it by assigning a `critical` [taint](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/) on a node as follows (`critical` can be changed to whatever you like):

```
kubectl taint nodes node1 role=critical:NoSchedule
```
or using kops, on a newly created `critical` InstanceGroup :

```
spec:
  taints:
  - role=critical:NoSchedule
```

Nodes having this taint won't schedule any pods that don't have a toleration assigned to them.

To distinguish these nodes from the others, we need to assign them a label (if using another InstanceGroup in kops, this is unnecessary since we can use `kops.k8s.io/instancegroup` label which is automatically created by kops) :

```
kubectl label node node1 role=critical
```

Then, to schedule a pod on a node having this taint, we need to specify the label of the host, and we must add a [toleration](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/) to its deployment :

```
spec:
  template:
    spec:
      tolerations:
      - key: "role"
        operator: "Equal"
        value: "critical"
        effect: "NoSchedule"
      nodeSelector:
        role: critical
```

If using kops, we can use `kops.k8s.io/instancegroup` label:

```
spec:
  template:
    spec:
      tolerations:
      - key: "role"
        operator: "Equal"
        value: "critical"
        effect: "NoSchedule"
      nodeSelector:
        kops.k8s.io/instancegroup: critical
```

While this is nice and can be sufficient, it still requires adding specific configuration to the deployments, and can be hard to systemize. 

A good way to ensure that critical pods are always running on tainted nodes is to use a namespace that has the ability to schedule them on the correct hosts by automatically adding the `nodeSelector` and `toleration` configuration. This can be done by enabling two admission controllers : [PodNodeSelector](https://kubernetes.io/docs/admin/admission-controllers/#podnodeselector) & [PodTolerationRestriction](https://kubernetes.io/docs/admin/admission-controllers/#podtolerationrestriction).

This can be done by adding to `--enable-admission-plugins` flag in api-server configuration file located at `/etc/kubernetes/manifests/kube-apiserver.manifest` the two admission controllers:

```
--enable-admission-plugins=[...],PodNodeSelector,PodTolerationRestriction
```
> The flag is named `--admission-control` before Kubernetes 1.10

> Don't forget to apply the new configuration

or using Kops by adding in the cluster configuration file :

```
spec:
  kubeAPIServer:
    enableAdmissionPlugins:
    - Initializers
    - NamespaceLifecycle
    - LimitRanger
    - ServiceAccount
    - PersistentVolumeLabel
    - DefaultStorageClass
    - DefaultTolerationSeconds
    - MutatingAdmissionWebhook
    - ValidatingAdmissionWebhook
    - NodeRestriction
    - ResourceQuota
    # added admission controllers
    - PodNodeSelector
    - PodTolerationRestriction
```

> `enableAdmissionPlugins` parameter is called `admissionControl` before Kubernetes 1.10

> Don't forget to perform a rolling-update to enable the modifications

Then, we need to add to the namespace the following annotations so that new pods created inside of it will be assigned the correct nodeSelector and tolerations configuration :

```
apiVersion: v1
kind: Namespace
metadata:
  name: critical
  annotations:
    scheduler.alpha.kubernetes.io/defaultTolerations: '[{"Key": "role", "Operator": "Equal", "Value": "critical", "Effect": "NoSchedule"}]'
    scheduler.alpha.kubernetes.io/node-selector: "role=critical"
```

Or is using Kops with a `critical` InstanceGroup
```
apiVersion: v1
kind: Namespace
metadata:
  name: critical
  annotations:
    scheduler.alpha.kubernetes.io/defaultTolerations: '[{"Key": "role", "Operator": "Equal", "Value": "critical", "Effect": "NoSchedule"}]'
    scheduler.alpha.kubernetes.io/node-selector: "kops.k8s.io/instancegroup=critical"
```

Then, to test, we can schedule a pod :

```
kubectl run -n critical nginx --image=nginx --port=80
```

Then we check it has the correct configurations :
```
 » kubectl get pods -n critical
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6f858d5d45-8zgjc   1/1     Running   0          3m
```

```
 » kubectl describe pod nginx-6f858d5d45-8zgjc -n critical
 [...]
  nodeSelector:
    kops.k8s.io/instancegroup: critical
  tolerations:
  - effect: NoSchedule
    key: role
    operator: Equal
    value: critical
```

To find on which node the pod is running, we can use the following command :
```
» kubectl get pods nginx-6f858d5d45-8zgjc -o "jsonpath={.spec.nodeName}"
node1
```

To verify that this is a critical workloads dedicated node, we can use the following command :
```
» kubectl get node node1 -L role
NAME    STATUS   ROLES   AGE   VERSION    ROLE
node1   Ready    node    1h    v1.14.2    critical
```

Or if using Kops :
```
» kubectl get node node1 -L kops.k8s.io/instancegroup
NAME    STATUS   ROLES   AGE   VERSION    INSTANCEGROUP
node1   Ready    node    1h    v1.14.2    critical
```

And that's a node dedicated to business critical pods !