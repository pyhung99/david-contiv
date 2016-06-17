---
layout: "install"
page_title: "Storage"
sidebar_current: "storage"
description: |-
  Storage in Contiv
---

# Storage in Contiv
# Contiv Storage: Cluster-Wide Volume Management for Containers

## Introduction 

Contiv Storage controls [Ceph](/install/user_guides/getting_started/storage/ceph.html) Rados block devices (RBDs) in a way that 
is easy for developers to use with Docker and flexible for ops to configure.
Reference volumes with Docker from anywhere Ceph is available; the volumes are located
and mounted. Contiv Storage works well with [Docker Compose](https://github.com/docker/compose) and
[Swarm](https://github.com/docker/swarm).

Contiv Storage profiles make instantiating similar class volumes easy. With profiles, you can:

* Give your development teams full-stack development environments (complete with state) that
  arrive on demand. 
* Scale your stateful containers quickly with our snapshot functionality.
  One CLI command captures the volume, and you can refer to anywhere, immediately. 
* Recover from crashed containers and hosts. Just re-initialize your
  container on another host with the same volume name.

Contiv Storage currently only supports Docker volume plugins. It provides 
first-class scheduler support for [Kubernetes](https://github.com/kubernetes/kubernetes).
Support for [Mesos](http://mesos.apache.org/) will be available before the first stable
release.

Contiv Storage uses a master/slave model to support a number of features, such as:

* On-the-fly image creation and (re)mount from any Ceph source, by referencing
  a policy and volume name.
* Manage many kinds of filesystems, including providing `mkfs` commands.
* Snapshot frequency and pruning. Copying snapshots to new volumes.
* Ephemeral (removed on container teardown) volumes.
* IOPS limiting.

## [Terminology and Concepts](/install/user_guides/storage/concepts/index.html)

## Policies

### Allocation

### IOPS Rate Limiting

### Snapshots

### Filesystems

## Disk Management

## Ceph

## NFS
