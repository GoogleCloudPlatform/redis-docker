# <a name="about"></a>About

This image contains an installation Redis 5.x.

For more information, see the [Official Image Marketplace Page](https://console.cloud.google.com/marketplace/details/google/redis5).

Pull command (first install [gcloud](https://cloud.google.com/sdk/downloads)):

```shell
gcloud docker -- pull marketplace.gcr.io/google/redis5
```

Dockerfile for this image can be found [here](https://github.com/GoogleCloudPlatform/redis-docker/tree/master/5).

# <a name="table-of-contents"></a>Table of Contents
* [Using Kubernetes](#using-kubernetes)
  * [Running Redis](#running-redis-kubernetes)
    * [Starting a Redis instance](#starting-a-redis-instance-kubernetes)
    * [Adding persistence](#adding-persistence-kubernetes)
  * [Redis CLI](#redis-cli-kubernetes)
    * [Connecting to a running Redis container](#connecting-to-a-running-redis-container-kubernetes)
  * [Configurations](#configurations-kubernetes)
    * [Using configuration volume](#using-configuration-volume-kubernetes)
* [Using Docker](#using-docker)
  * [Running Redis](#running-redis-docker)
    * [Starting a Redis instance](#starting-a-redis-instance-docker)
    * [Adding persistence](#adding-persistence-docker)
  * [Redis CLI](#redis-cli-docker)
    * [Connecting to a running Redis container](#connecting-to-a-running-redis-container-docker)
  * [Configurations](#configurations-docker)
    * [Using configuration volume](#using-configuration-volume-docker)
* [References](#references)
  * [Ports](#references-ports)
  * [Volumes](#references-volumes)

# <a name="using-kubernetes"></a>Using Kubernetes

Consult [Marketplace container documentation](https://cloud.google.com/marketplace/docs/marketplace-container)
for additional information about setting up your Kubernetes environment.

## <a name="running-redis-kubernetes"></a>Running Redis

### <a name="starting-a-redis-instance-kubernetes"></a>Starting a Redis instance

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
    - image: marketplace.gcr.io/google/redis5
      name: redis
```

Run the following to expose the port.
Depending on your cluster setup, this might expose your service to the
Internet with an external IP address. For more information, consult
[Kubernetes documentation](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/).

```shell
kubectl expose pod some-redis --name some-redis-6379 \
  --type LoadBalancer --port 6379 --protocol TCP
```

### <a name="adding-persistence-kubernetes"></a>Adding persistence

Redis is an in-memory database but does write data to disk periodically for recovery. The location of this dump file can be found at `/data/dump.rdb`. By putting `/data` on to a persistent volume, the data will survive container restarts.

You can find more about redis persistence on the [redis website](https://redis.io/topics/persistence).

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
    - image: marketplace.gcr.io/google/redis5
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

Run the following to expose the port.
Depending on your cluster setup, this might expose your service to the
Internet with an external IP address. For more information, consult
[Kubernetes documentation](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/).

```shell
kubectl expose pod some-redis --name some-redis-6379 \
  --type LoadBalancer --port 6379 --protocol TCP
```

## <a name="redis-cli-kubernetes"></a>Redis CLI

### <a name="connecting-to-a-running-redis-container-kubernetes"></a>Connecting to a running Redis container

```shell
kubectl exec -it some-redis -- redis-cli
```

To test if redis is working we create a key called MY_TEST_KEY. Run the following command to set a test key.

```
SET MY_TEST_KEY pass
```

Run the following command to verify that the set command above succeeded. This should print out "pass".

```
GET MY_TEST_KEY
```

## <a name="configurations-kubernetes"></a>Configurations

### <a name="using-configuration-volume-kubernetes"></a>Using configuration volume

Redis can be started with a configuration file to customize or tweak how the cluster will run. This file is commonly named `redis.conf`.

Assume /path/to/your/redis.conf is the configuration file on your local host.

Create the following `configmap`:

```shell
kubectl create configmap redisconfig \
  --from-file=/path/to/your/redis.conf
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
    - image: marketplace.gcr.io/google/redis5
      name: redis
      args:
        - /etc/redis/redis.conf
      volumeMounts:
        - name: redisconfig
          mountPath: /etc/redis
  volumes:
    - name: redisconfig
      configMap:
        name: redisconfig
```

Run the following to expose the port.
Depending on your cluster setup, this might expose your service to the
Internet with an external IP address. For more information, consult
[Kubernetes documentation](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/).

```shell
kubectl expose pod some-redis --name some-redis-6379 \
  --type LoadBalancer --port 6379 --protocol TCP
```

See [Volume reference](#references-volumes) for more details.

# <a name="using-docker"></a>Using Docker

Consult [Marketplace container documentation](https://cloud.google.com/marketplace/docs/marketplace-container)
for additional information about setting up your Docker environment.

## <a name="running-redis-docker"></a>Running Redis

### <a name="starting-a-redis-instance-docker"></a>Starting a Redis instance

Use the following content for the `docker-compose.yml` file, then run `docker-compose up`.

```yaml
version: '2'
services:
  redis:
    container_name: some-redis
    image: marketplace.gcr.io/google/redis5
    ports:
      - '6379:6379'
```

Or you can use `docker run` directly:

```shell
docker run \
  --name some-redis \
  -p 6379:6379 \
  -d \
  marketplace.gcr.io/google/redis5
```

### <a name="adding-persistence-docker"></a>Adding persistence

Redis is an in-memory database but does write data to disk periodically for recovery. The location of this dump file can be found at `/data/dump.rdb`. By putting `/data` on to a persistent volume, the data will survive container restarts.

You can find more about redis persistence on the [redis website](https://redis.io/topics/persistence).

Use the following content for the `docker-compose.yml` file, then run `docker-compose up`.

```yaml
version: '2'
services:
  redis:
    container_name: some-redis
    image: marketplace.gcr.io/google/redis5
    ports:
      - '6379:6379'
    volumes:
      - /path/to/your/redis/data/directory:/data
```

Or you can use `docker run` directly:

```shell
docker run \
  --name some-redis \
  -p 6379:6379 \
  -v /path/to/your/redis/data/directory:/data \
  -d \
  marketplace.gcr.io/google/redis5
```

## <a name="redis-cli-docker"></a>Redis CLI

### <a name="connecting-to-a-running-redis-container-docker"></a>Connecting to a running Redis container

```shell
docker exec -it some-redis redis-cli
```

To test if redis is working we create a key called MY_TEST_KEY. Run the following command to set a test key.

```
SET MY_TEST_KEY pass
```

Run the following command to verify that the set command above succeeded. This should print out "pass".

```
GET MY_TEST_KEY
```

## <a name="configurations-docker"></a>Configurations

### <a name="using-configuration-volume-docker"></a>Using configuration volume

Redis can be started with a configuration file to customize or tweak how the cluster will run. This file is commonly named `redis.conf`.

Assume /path/to/your/redis.conf is the configuration file on your local host.

Use the following content for the `docker-compose.yml` file, then run `docker-compose up`.

```yaml
version: '2'
services:
  redis:
    container_name: some-redis
    image: marketplace.gcr.io/google/redis5
    command:
      - /etc/redis/redis.conf
    ports:
      - '6379:6379'
    volumes:
      - /path/to/your/redis.conf:/etc/redis/redis.conf
```

Or you can use `docker run` directly:

```shell
docker run \
  --name some-redis \
  -p 6379:6379 \
  -v /path/to/your/redis.conf:/etc/redis/redis.conf \
  -d \
  marketplace.gcr.io/google/redis5 \
  /etc/redis/redis.conf
```

See [Volume reference](#references-volumes) for more details.

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
| /data | Location where redis will stores the rdb data |

