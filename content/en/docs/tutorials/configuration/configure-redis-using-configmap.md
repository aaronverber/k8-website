---
reviewers:
- eparis
- pmorie
title: Configure Redis using a ConfigMap
content_type: tutorial
weight: 30
---

<!-- overview -->

In this tutorial, you will create a basic Redis cache using a ConfigMap. A ConfigMap enables your Redis configuration data to be independent of your instance. 


## {{% heading "objectives" %}}

At the end of this tutorial, you should be able to:

* Create and update a ConfigMap with Redis configuration values.
* Create a Redis Pod that mounts and uses the ConfigMap.
* Verify that the configuration was correctly applied.


## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

This tutorial builds on concepts described in [Configure a Pod to Use a ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/). You should understand the basic functions of a ConfigMap before using this tutorial. Also, this tutorial requires `kubectl` 1.14 or above.


<!-- lessoncontent -->


## Step 1: Create a new ConfigMap

In a terminal, run the following command to create your ConfigMap's configuration block:

```shell
cat <<EOF >./example-redis-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-redis-config
data:
  redis-config: ""
EOF
```

## Step 2: Create a new Redis instance

Run the following commands to apply the example ConfigMap and build a Redis pod from a sample manifest.

```shell
kubectl apply -f example-redis-config.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml
```

## Step 3: Verify the configuration

1. Run the following command in your terminal to inspect the Redis pod and verify that `kubectl` applied the ConfigMap correctly.

```shell
kubectl get pod my-redis-pod -o yaml > redis-pod.yaml
cat redis-pod.yaml
```

The following code sample shows what the full YAML output should look like:

{{% code_sample file="pods/config/redis-pod.yaml" %}}

2. Examine the contents of the Redis pod manifest and note the following:

* A volume named `config` is created by `spec.volumes[1]`
* The `key` and `path` under `spec.volumes[1].configMap.items[0]` exposes the `redis-config` key from the 
  `example-redis-config` ConfigMap as a file named `redis.conf` on the `config` volume.
* The `config` volume is then mounted at `/redis-master` by `spec.containers[0].volumeMounts[1]`.

This has the net effect of exposing the data in `data.redis-config` from the `example-redis-config`
ConfigMap above as `/redis-master/redis.conf` inside the pod.

3. Run the following command to examine the created objects:

```shell
kubectl get pod/redis configmap/example-redis-config 
```

You should see the following output:

```
NAME        READY   STATUS    RESTARTS   AGE
pod/redis   1/1     Running   0          8s

NAME                             DATA   AGE
configmap/example-redis-config   1      14s
```

4. Remember that we left the `redis-config` key blank in the example configuration. To view the redis-config, run the following command:

```shell
kubectl describe configmap/example-redis-config
```

Because redis-config is blank, you should see an empty `redis-config` key:

```shell
Name:         example-redis-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
redis-config:
```

## Step 4:  Additional verification

1. Run the following command to open the Redis CLI:

```shell
kubectl exec -it redis -- redis-cli
```

2. In the Redis CLI, run the following command to check `maxmemory`:

```shell
127.0.0.1:6379> CONFIG GET maxmemory
```

You should see the default value of 0:

```shell
1) "maxmemory"
2) "0"
```

3. You can also check `maxmemory-policy` using a similar method:

```shell
127.0.0.1:6379> CONFIG GET maxmemory-policy
```

You should see the default value of `noeviction`:

```shell
1) "maxmemory-policy"
2) "noeviction"
```

## Step 5: Update the ConfigMap

In this step, you will use the following example values to update your ConfigMap with new configurations.

{{% code_sample file="pods/config/example-redis-config.yaml" %}}

Run the following command to update the ConfigMap:

```shell
kubectl apply -f example-redis-config.yaml
```

## Step 6: Recreate the Redis pod to apply the values

1. Restart the Redis pod to apply the updated ConfigMap:

```shell
kubectl delete pod redis
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml
```

2. Run the following command to check the configuration values:

```shell
kubectl exec -it redis -- redis-cli
```

3. Run the following command to check `maxmemory`:

```shell
127.0.0.1:6379> CONFIG GET maxmemory
```

It should return the updated value of 2097152:

```shell
1) "maxmemory"
2) "2097152"
```

4. Run the following command to check `maxmemory-policy`:

```shell
127.0.0.1:6379> CONFIG GET maxmemory-policy
```

It now reflects the desired value of `allkeys-lru`:

```shell
1) "maxmemory-policy"
2) "allkeys-lru"
```

5. To clean up your work, run the following command to delete the Redis pod:

```shell
kubectl delete pod/redis configmap/example-redis-config
```

## {{% heading "whatsnext" %}}


* [Theoretical link to Redis configuration topics]
* [Theoretical link to additional Kubernetes configuration topics]
