# Akka Cluster Orchestration Example - DC/OS Deployment

Prerequisites:

* [sbt](https://www.scala-sbt.org/)
* [Docker](https://www.docker.com/)
* [reactive-cli](https://developer.lightbend.com/docs/lightbend-orchestration-kubernetes/latest/cli-installation.html#install-the-cli)
* `dcos` setup to point to your DC/OS cluster
* A Docker registry that you have read access (from DC/OS) to, write access to (locally)

> **You'll need version 1.1.0 (or later) of the CLI.**

1) Verify DC/OS environment

```bash
dcos node
```

2) Install Marathon-lb

Lightbend Orchestration generates configuration for [Marathon-lb](https://github.com/mesosphere/marathon-lb), so you'll need to install it on your DC/OS cluster. The following command will deploy it on your public slave nodes.

```bash

```bash
dcos package install marathon-lb
```

3) Build and publish project

Next, you'll need to build the Docker images and push them to a registry. Be sure to change `REGISTRY` to the location of your actual Docker registry.

```bash
REGISTRY=boot.dcos:5000
sbt docker:publishLocal
docker tag akka-cluster-orchestration-example:0.1.0 "$REGISTRY/akka-cluster-orchestration-example:0.1.0"
docker push "$REGISTRY/akka-cluster-orchestration-example:0.1.0"
```

4) Deploy project

First, designate a host to deploy the application to. For this example, you can simply add an entry to your `/etc/hosts` file, but in a production setting you'd typically create a real DNS record. The following entry is pointing `akka.dcos` to the address of a local public node. Be sure to substitute in addresses of your public nodes.

```
# /etc/hosts

192.168.65.60 akka.dcos
```

Next, actually deploy the application. The following command will generate the Marathon configuration and deploy the application to your DC/OS cluster. Be sure to update `REGISTRY` and `HOST` as applicable.

> Using a registry over plain http (i.e. not https)? Provide the `--registry-disable-https` flag to `rp`.

```bash
REGISTRY=boot.dcos:5000
HOST=akka.dcos
rp generate-marathon-configuration "$REGISTRY/akka-cluster-orchestration-example:0.1.0" \
  --marathon-lb-host "$HOST" \
  --instances 3 | dcos marathon app add
```

5) View Results

This may take 30 seconds or so until all services have been started and the cluster has formed. You should see a member listing.

```bash
dcos marathon app list
```

```
ID                                   MEM   CPUS  TASKS  HEALTH  DEPLOYMENT  WAITING  CONTAINER  CMD
/akka-cluster-orchestration-example  128    1     3/3    3/3       ---      False      DOCKER   N/A
/marathon-lb                         1024   2     1/1    1/1       ---      False      DOCKER   N/A
```

```bash
curl "http://$HOST/cluster-example"
```

```
Akka Cluster Members
====================

akka.tcp://my-system@192.168.65.131:8800
akka.tcp://my-system@192.168.65.111:24565
akka.tcp://my-system@192.168.65.121:18840
```
