# Supported tags and respective `Dockerfile` links

* `0.9.7`, `0.9` [(0.9.7/Dockerfile)](https://github.com/31z4/storm-docker/blob/master/0.9.7/Dockerfile)
* `0.10.2`, `0.10` [(0.10.2/Dockerfile)](https://github.com/31z4/storm-docker/blob/master/0.10.2/Dockerfile)
* `1.0.2`, `1.0`, `latest` [(1.0.2/Dockerfile)](https://github.com/31z4/storm-docker/blob/master/1.0.2/Dockerfile)

[![](https://images.microbadger.com/badges/image/31z4/storm.svg)](http://microbadger.com/images/31z4/storm)

# What is Apache Storm?

Apache Storm is a distributed computation framework written predominantly in the Clojure programming language. Originally created by Nathan Marz and team at BackType, the project was open sourced after being acquired by Twitter. It uses custom created "spouts" and "bolts" to define information sources and manipulations to allow batch, distributed processing of streaming data. The initial release was on 17 September 2011.

> [wikipedia.org/wiki/Storm_(event_processor)](https://en.wikipedia.org/wiki/Storm_(event_processor))

![](https://upload.wikimedia.org/wikipedia/commons/7/70/Storm_logo.png)

# How to use this image

## Running topologies in local mode

Assuming you have `storm-starter-topologies-1.0.2.jar` in the current directory.

	$ docker run -it -v $(pwd)/storm-starter-topologies-1.0.2.jar:/topology.jar 31z4/storm:1.0.2 storm jar /topology.jar org.apache.storm.starter.ExclamationTopology

## Setting up a minimal Storm cluster

1.	[Apache Zookeeper](https://zookeeper.apache.org/) is a must for running a Storm cluster. Start it first. Since the Zookeeper "fails fast" it's better to always restart it.

		$ docker run -d --restart always --name some-zookeeper zookeeper:3.4

2.	The Nimbus daemon has to be connected with the Zookeeper. It's also a "fail fast" system.

		$ docker run -d --restart always --name some-nimbus --link some-zookeeper:zookeeper 31z4/storm:1.0.2 storm nimbus

3.	Finally start a single Supervisor node. It will talk to the Nimbus and Zookeeper.

		$ docker run -d --restart always --name supervisor --link some-zookeeper:zookeeper --link some-nimbus:nimbus 31z4/storm:1.0.2 storm supervisor

4.	Now you can submit a topology to our cluster.

		$ docker run --link some-nimbus:nimbus -it --rm -v $(pwd)/storm-starter-topologies-1.0.2.jar:/topology.jar 31z4/storm:1.0.2 storm jar /topology.jar org.apache.storm.starter.WordCountTopology topology

5.	Optionally, you can start the Storm UI.

		$ docker run -d -p 8080:8080 --restart always --name ui --link some-nimbus:nimbus 31z4/storm:1.0.2 storm ui

## ... via [`docker-compose`](https://github.com/docker/compose)

Example `docker-compose.yml` for `storm`:

```yaml
version: '2'
services:
    zookeeper:
        image: zookeeper:3.4
        container_name: zookeeper
        restart: always

    nimbus:
        image: 31z4/storm:1.0.2
        container_name: nimbus
        command: storm nimbus
        depends_on:
            - zookeeper
        links:
            - zookeeper
        restart: always
        ports:
            - 6627:6627

    supervisor:
        image: 31z4/storm:1.0.2
        container_name: supervisor
        command: storm supervisor
        depends_on:
            - nimbus
            - zookeeper
        links:
            - nimbus
            - zookeeper
        restart: always
```

Run `docker-compose up` and wait for it to initialize completely. The Nimbus will be available at your host and port `6627`.

## Configuration

This image uses [default configuration](https://github.com/apache/storm/blob/v1.0.2/conf/defaults.yaml) of the Apache Storm. There are two main ways to change it.

1.	Using command line arguments.

		$ docker run -d --restart always --name nimbus 31z4/storm:1.0.2 storm nimbus -c storm.zookeeper.servers="[zookeeper]"

2.	Assuming you have `storm.yaml` in the current directory you can mount it as a volume.

		$ docker run -it -v $(pwd)/storm.yaml:/conf/storm.yaml 31z4/storm:1.0.2 storm nimbus

## Logging

This image uses [default logging configuration](https://github.com/apache/storm/tree/v1.0.2/log4j2). All logs go to the `/logs` directory by default.

## Data persistence

No data are persisted by default. For convenience there are `/data` and `/logs` directories in the image owned by `storm` user. Use them accordingly to persist data and logs using volumes.

	$ docker run -it -v /logs -v /data 31z4/storm:1.0.2 storm nimbus

*Please be noticed that using paths other than those predefined is likely to cause permission denied errors. It's because for [security reasons](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#user) the Storm is running under the non-root `storm` user.*

# License

View [license information](http://storm.apache.org/about/free-and-open-source.html) for the software contained in this image.
