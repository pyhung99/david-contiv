---
layout: "install"
page_title: "Networking"
sidebar_current: "networking"
description: |-
  Networking with Contiv
---

# Networking with Contiv

## [Terminology and Concepts](/install/user_guides/networking/concepts.html)

## [Networks](/install/user_guides/networking/networks.html)

## Policies

### Security

### Bandwidth

### Prioritization

## [Load Balancing](/install/user_guides/getting_started/networking/load_balancing.html)

### Services and Service Providers
A service is an abstraction that describes a network connection on one or more ports to a cluster of containers.
The containers are identified by a [label selector] that is unique to the service. 

Collectively, the containers specified by the service's label selector make up a service provider.

Organizing containers into a service provides the following benefits:
- Efficient load balancing.
- Minimal downtime.
- Easy scaling for variable workloads.

### How is a Service Defined

A service must be created on an existing [service network]; therefore, you must first 
create the service network.

In Contiv Network, you define both services and service networks using the [`netctl`] utility. 

## External Connectivity

### Routed Networks

### Bridged Networks

### Overlay or Cloud Networks

### Cisco ACI

## Service Discovery

## IP Address Management

## Hybrid Workloads

### Providing Connectivity

### Applying Policies
