---
layout: "docs"
page_title: "Documentation"
sidebar_current: "docs-home"
description: |-
  Welcome to the Contiv documentation!
---

# Contiv Tools

This page describes the command line user interface for various contiv tools. The CLI
uses the REST interface to a cluster-wide master entity that can be accessible to
any automation tool.

## Contiv Networking
[netplugin](https://github.com/contiv/netplugin) allows cluster wide connectivity
and policy controls for container networking across a multi-host cluster. If you are
interested in contributing to the project or learning more about the architecture, please
see the [README](https://github.com/contiv/netplugin/blob/master/README.md) for
more information.

## Contiv Storage
[volplugin](https://github.com/contiv/volplugin) controls
[Ceph](http://ceph.com/) RBD devices with a master/slave model to orchestrate
the mounting and cross-host remounting of volumes scheduled with containers.
**This provides highly available, persistent volumes that can live anywhere on
your network, and be referenced by containers for use.** You can control
docker to mount these volumes with a plugin, or you can (soon) use schedulers,
as well as docker-compose to manage your application alongside Ceph volumes.

This documentation is for the users and administrators of volplugin. If you
wish to learn more about the project for development purposes, please see
[the README](https://github.com/contiv/volplugin/blob/master/README.md)
for more information.
