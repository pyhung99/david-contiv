# Contiv Storage

[Contiv Storage](https://github.com/contiv/volplugin "Title") is a storage policy system designed to work within a Docker application environment. Contiv Storage can automatically move your storage with your containers, rather than requiring storage to be on specific hosts to take advantage of their storage.

**Note**: The Contiv Github site may refer to Contiv Storage as *volplugin*. This was a former working name for both the Contiv Storage project and product. 

## Architecture

Contiv Storage is a suite of components:

The `volsupervisor` process 

The `volmaster` service is the master process. It coordinates the storage clients (`volplugin`) safely manage container volumes. It records its state with `etcd`.

The `volplugin` process runs on each storage server. It mediates state management between `volmaster` and `docker`, and mounts volumes on specific hosts.

The `volcli` command is a utility for managing `volmaster`. It works by making REST API calls to `volmaster` and in some cases writes directly to etcd.

### Organizational Architecture

The `volmaster` process is completely stateless, and can be run multi-host for redundancy. The `volmaster` process requires root permissions and must be able to manipulate RBD images with the `rbd` tool.

The `volsupervisor` process executes scheduled and supervised tasks such as snapshotting. It may only be deployed on one host at a time.

The `volplugin` process runs on every host that is running containers. On startup, `volplugin` creates a UNIX socket in the appropriate plugin path so that Docker discovers it. This creates a driver named `volplugin`.

The management tool `volcli` can reside anywhere that it can gain access to the `etcd` cluster and to `volmaster`.

### Security Architecture

Security is a feature targeted for beta release. It is still under development.

### Network Architecture

By default, `volmaster` listens on IP and port `0.0.0.0:9005`. It exposes all of its operations using a RESTful interface. The REST APIs are used both by `volplugin` and `volcli`. The `volmaster` process connects to `etcd` at `127.0.0.1:2379` You can change the IP and port by supplying  the `--etcd` argument one or more times.

The `volsupervisor` process requires root access, connections to `etcd`, and administrator access to the Ceph `rbd` tools as.

The `volplugin` process contacts `volmaster` but does not listen on any network ports (it uses a UNIX socket as described in [Organizational Architecture] to communicate with Docker). It by default connects to `volmaster` at `127.0.0.1:9005` and must be supplied the `--master` switch to talk to a remote `volmaster`.

`volcli` communicates with both `volmaster` and `etcd` to pass state and operations to the system.
