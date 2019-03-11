# fluent-bit log collector service

We deploy fluent-bit on each node to read production jupyter container logs
then send filtered output to a single fluent-bit service with an attached disk.

Based on https://github.com/fluent/fluent-bit-kubernetes-logging

## Setup

1. Create the namespace, service account and role setup:

```
kubectl create namespace logging
kubectl -n logging create -f fluent-bit-service-account.yaml
kubectl -n logging create -f fluent-bit-role.yaml
kubectl -n logging create -f fluent-bit-role-binding.yaml
```

2. Create the collector service
	- PersistentVolumeClaim: to save everything to a file
	- Deployment: runs a fluent-bit pod that mounts that volume
	- Service: listens on a service endpoint named "collector"
	- PodDisruptionBudget: make sure one pod is running

```
kubectl -n logging create collector.yaml
```

3. Create a ConfigMap that will be used by the DaemonSet.

	- specifying container log to read
	- filters input (removes "/metrics")
	- forwards output to a "collector" service.

```
kubectl -n logging create cm-fluent-bit-file.yaml
```

4. Create a DaemonSet

	- Runs fluent-bit on each node
	- hostpath volumes to mount within the fluent-bit pod

```
kubectl -n logging create ds-fluent-bit-file.yaml
```


## Access the logs

1. Find the pod with the attached logging disk
```
collector_pod=`kubectl -n logging get pod -l app=collector-app -o name | cut -d/ -f2`
```

2. View the live log
```
kubectl -n logging exec -it ${collector_pod} -- /usr/bin/tail -f /srv/events.log
```

3. Save the log locally
```
kubectl -n logging cp ${collector_pod}:/srv/events.log .
```
