# <a name="about"a></a>About

This image contains an installation Redis 3.2.

For more information, see the [Official Image Launcher Page](https://console.cloud.google.com/launcher/details/google/redis3).

Pull command:
```shell
gcloud docker -- pull launcher.gcr.io/google/redis
```

Dockerfile for this image can be found [here](https://github.com/GoogleCloudPlatform/redis-docker/tree/master/3.2.8).

# <a name="table-of-contents"></a>Table of Contents
* [Using Kubernetes](#using-kubernetes)
  * [Running Redis](#running-redis-kubernetes)
    * [Start a Redis Instance](#start-a-redis-instance-kubernetes)
    * [Use a persistent data volume](#use-a-persistent-data-volume-kubernetes)
  * [Configurations](#configurations-kubernetes)
    * [Using configuration volume](#using-configuration-volume-kubernetes)
  * [Redis CLI](#redis-cli-kubernetes)
    * [Connect to a running Redis container](#connect-to-a-running-redis-container-kubernetes)
* [Using Docker](#using-docker)
  * [Running Redis](#running-redis-docker)
    * [Start a Redis Instance](#start-a-redis-instance-docker)
    * [Use a persistent data volume](#use-a-persistent-data-volume-docker)
  * [Configurations](#configurations-docker)
    * [Using configuration volume](#using-configuration-volume-docker)
  * [Redis CLI](#redis-cli-docker)
    * [Connect to a running Redis container](#connect-to-a-running-redis-container-docker)
* [References](#references)
  * [Ports](#references-ports)
  * [Volumes](#references-volumes)

# <a name="using-kubernetes"></a>Using Kubernetes

## <a name="running-redis-kubernetes"></a>Running Redis

### <a name="start-a-redis-instance-kubernetes"></a>Start a Redis Instance

Copy the following content to `pod.yaml` file, and run `kubectl create -f pod.yaml`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: some-redis
  labels:
    name: some-redis
spec:
  containers:
    - image: launcher.gcr.io/google/redis
      name: redis
```

Run the following to expose the port:
```shell
kubectl expose pod some-redis --name some-redis-6379 \
  --type LoadBalancer --port 6379 --protocol TCP
```

### <a name="use-a-persistent-data-volume-kubernetes"></a>Use a persistent data volume

Redis is a in memory database, but does write the data to disk. We need to make sure the data is protected in the event of a container crash or restart. By defining a data volume we can ensure the data is retained.

Copy the following content to `pod.yaml` file, and run `kubectl create -f pod.yaml`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: some-redis
  labels:
    name: some-redis
spec:
  containers:
    - image: launcher.gcr.io/google/redis
      name: redis
      volumeMounts:
        - name: redisdata
          mountPath: /data
          subPath: redisdata
  volumes:
    - name: redisdata
      persistentVolumeClaim:
        claimName: redisdata
---
# Request a persistent volume from the cluster using a Persistent Volume Claim.
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: redisdata
  annotations:
    volume.alpha.kubernetes.io/storage-class: default
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 5Gi
```

Run the following to expose the port:
```shell
kubectl expose pod some-redis --name some-redis-6379 \
  --type LoadBalancer --port 6379 --protocol TCP
```

## <a name="configurations-kubernetes"></a>Configurations

There are several ways to configure your redis cluster.

### <a name="using-configuration-volume-kubernetes"></a>Using configuration volume

You can start redis with a configuration file to customize or tweak how the cluster will run. Create a `redis.conf` file, then start redis with the redis configuration file as the first argument.

Create the following `configmap`:
```shell
kubectl create configmap redisconfig \
  --from-file=/mnt/redisconfig/redis.conf
```

Copy the following content to `pod.yaml` file, and run `kubectl create -f pod.yaml`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: some-redis
  labels:
    name: some-redis
spec:
  containers:
    - image: launcher.gcr.io/google/redis
      name: redis
      args:
        - /etc/redis/redis.conf
      volumeMounts:
        - name: redisconfig
          mountPath: /etc/redis/redis.conf
  volumes:
    - name: redisconfig
      configMap:
        name: redisconfig
```

Run the following to expose the port:
```shell
kubectl expose pod some-redis --name some-redis-6379 \
  --type LoadBalancer --port 6379 --protocol TCP
```

See [Volume reference](#references-volumes) for more details.

## <a name="redis-cli-kubernetes"></a>Redis CLI

Now that it's deployed you can connect to the container.

### <a name="connect-to-a-running-redis-container-kubernetes"></a>Connect to a running Redis container

```shell
kubectl exec -it some-redis -- redis-cli
```

# <a name="using-docker"></a>Using Docker

## <a name="running-redis-docker"></a>Running Redis

### <a name="start-a-redis-instance-docker"></a>Start a Redis Instance

Use the following content for the `docker-compose.yml` file, then run `docker-compose up`.
```yaml
version: '2'
services:
  redis:
    image: launcher.gcr.io/google/redis
```

Or you can use `docker run` directly:

```shell
docker run \
  --name some-redis \
  -d \
  launcher.gcr.io/google/redis
```

### <a name="use-a-persistent-data-volume-docker"></a>Use a persistent data volume

Redis is a in memory database, but does write the data to disk. We need to make sure the data is protected in the event of a container crash or restart. By defining a data volume we can ensure the data is retained.

Use the following content for the `docker-compose.yml` file, then run `docker-compose up`.
```yaml
version: '2'
services:
  redis:
    image: launcher.gcr.io/google/redis
    volumes:
      - /mnt/redisdata:/data
```

Or you can use `docker run` directly:

```shell
docker run \
  --name some-redis \
  -v /mnt/redisdata:/data \
  -d \
  launcher.gcr.io/google/redis
```

## <a name="configurations-docker"></a>Configurations

There are several ways to configure your redis cluster.

### <a name="using-configuration-volume-docker"></a>Using configuration volume

You can start redis with a configuration file to customize or tweak how the cluster will run. Create a `redis.conf` file, then start redis with the redis configuration file as the first argument.

Use the following content for the `docker-compose.yml` file, then run `docker-compose up`.
```yaml
version: '2'
services:
  redis:
    image: launcher.gcr.io/google/redis \
    command:
      - /etc/redis/redis.conf
    volumes:
      - /mnt/redisconfig/redis.conf:/etc/redis/redis.conf
```

Or you can use `docker run` directly:

```shell
docker run \
  --name some-redis \
  -v /mnt/redisconfig/redis.conf:/etc/redis/redis.conf \
  -d \
  launcher.gcr.io/google/redis \
  /etc/redis/redis.conf
```

See [Volume reference](#references-volumes) for more details.

## <a name="redis-cli-docker"></a>Redis CLI

Now that it's deployed you can connect to the container.

### <a name="connect-to-a-running-redis-container-docker"></a>Connect to a running Redis container

```shell
docker exec -it some-redis redis-cli
```

# <a name="references"></a>References

## <a name="references-ports"></a>Ports

These are the ports exposed by the container image.

| **Port** | **Description** |
|:---------|:----------------|
| TCP 6379 | Redis port |

## <a name="references-volumes"></a>Volumes

These are the filesystem paths used by the container image.

| **Path** | **Description** |
|:---------|:----------------|
| /etc/redis/redis.conf | Location to the mounted configmap to configure redis |
| /data | Location where redis will stores the rdb data |
