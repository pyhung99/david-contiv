# Contiv Storage
[Contiv Storage](https://github.com/contiv/volplugin "Title")
is a Ceph volume driver and policy system that works with Docker. 
Rather than pinning containers to specific hosts to consume the
host's storage, Contiv Storage automatically moves your storage 
with your containers. 

## Getting Started

The following procedure describes setting up a test environment with three VMs. Once
the test environment is set up see the [**Configure Services**](#Configure Services).

### Prerequisites

You must have the following to build the standalone test environment:
- A Linux or OS X machine with at least 8 GB RAM. You will need to install the following:
  - :q


### Clone and build the project

### On Linux (without a VM)

Clone the project:

```
git clone https://github.com/contiv/volplugin
```

Build the project:

```
make run-build
```


The command `make run-build` installs utilities for building the software in
the `$GOPATH`, as well as the `volmaster`, `volplugin` and `volcli` utilities.

### Everywhere else (with a VM):

Clone the project:

```
git clone https://github.com/contiv/volplugin
```


Build the project:

```
make start
```


The build and binaries will be on the VM in the following directory `/opt/golang/bin`.

## Do it yourself

### Installing Dependencies

Use the Contiv [nightly releases](https://github.com/contiv/volplugin/releases)
when following these steps:

**Note:** Using the nightly builds is simpler than building the applications.

Install the dependencies in the following order:

1. Follow the [Getting Started](https://github.com/coreos/etcd/releases/tag/v2.2.0) to install [Etcd](https://coreos.com/etcd/docs/latest/getting-started-with-etcd.html).
  * Currently versions 2.0 and later are supported.

2. Follow the [Ceph Installation Guide](http://docs.ceph.com/docs/master/install/) to install [Ceph](http://ceph.com).
3. Configure Ceph with [Ansible](https://github.com/ceph/ceph-ansible).

  **Note**: See the [README](https://github.com/contiv/volplugin/blob/master/README.md#running-the-processes)
  for pre-configured VMs that work on any UNIX operating system to simplify
    Ceph installation.

4. Upload a global configuration. You can find an example one [here](https://github.com/contiv/volplugin/blob/master/systemtests/testdata/global1.json)

5. Start volmaster in debug mode (as root):

```
volmaster &
```

**Note**: volmaster debug mode is very noisy and is not recommended. Therefore,
avoid using it with background processes. volplugin currently connects to
volmaster using port 9005, however in the future it is variable.

6. Start volsupervisor (as root):

```
volsupervisor &
```


**Note**: volsupervisor debug mode is very noisy and is not recommended.

7.  Start volplugin in debug mode (as root):

```
volplugin &
```

If running volplugin on multiple hosts, use the `--master` flag to
provide a ip:port pair to connect to over http. By default it connects to
`127.0.0.1:9005`.

## Configure Services

Ensure Ceph is fully operational, and that the `rbd` tool works as root.

Upload a policy:

```
volcli policy upload policy1
```


**Note**: It accepts the policy from stdin, e.g.: `volcli policy upload policy1 < mypolicy.json`
Examples of a policy are in [systemtests/testdata](https://github.com/contiv/volplugin/tree/master/systemtests/testdata).

### Creating a container with a volume


Create a volume that refers to the volplugin driver:

```
docker volume create -d volplugin --name policy1/test
```

**Notes**:

* `test` is the name of the volume, and is located under policy `policy1`,
 which is uploaded with `volcli policy upload.`
* The volume will inherit the properties of the policy. Therefore, the
volume will be of appropriate size, iops, etc.
* There are numerous options (see below) to declare overrides of most parameters in the policy configuration.
* Run a container that uses the policy:

```
docker run -it -v policy1/test:/mnt ubuntu bash
```
* Run `mount | grep /mnt` in the container.


**Note**: `/dev/rbd#`should be attached to that directory.

* Once a multi-host system is setup, anytime the volume is not mounted, it
can be mounted on any host that has a connected rbd client available, and
volplugin running.
## Try it out

Contiv Storage currently does not run in a container. The other components (Network, Cluster) do, but
`volplugin` does not. The `volplugin' service must be run on the host where the volumes are
to be mounted.

### Prerequisites:

To set up a single small VM (4096 MB RAM) for running just the tools and trying it out,
- Install Ubuntu 15 or Centos 7 on each of your servers used for the Contiv cluster.
- If your servers are behind an http proxy, set the following environment variables to expose the proxy:

```
export http_proxy=<proxy url>
export https_proxy=<proxy_url>
```

```
$ make demo
```

Note that you will still need ansible, virtualbox, and vagrant.

#### Development/Mock Production env

For a more comprehensive version of the system including swarm support across
several hosts, see below:

On the host, equivalent or greater:

* 12GB of free RAM. Ceph likes RAM.
* [VirtualBox](https://virtualbox.org) 5.0.2 or greater
* [Vagrant](https://vagrantup.com) 1.8.x
* [Ansible](https://ansible.com) 2.0+
  * install with pip; you'll want to install `python-pip` and `python-dev` on
    ubuntu machines, then `sudo pip install ansible`. `brew install ansible`
    should do the right thing on OS X.
  * The make tooling in this repository will install it for you if it is not
    already installed, which requires `pip`. If you are not root, it may fail
    to perform this operation. The solution to this problem is to install
    ansible independently as described above.
* [Go](https://golang.org) 1.6 to run the system tests.

Your guests will configure themselves.

### Running the processes

Be sure to start and run the environment with `make start` before you
continue with these steps. You must have working vagrant, virtualbox, and
ansible. If you are behind a proxy server, set the `https_proxy` same as the
`http_proxy`. Ansible has a current limitation (https://github.com/ansible/ansible/issues/10941), 
that it only supports `http://` proxy. So, `https_proxy` should be set to
`"http://<proxyserver>:<port>"`

These instructions ssh you into the `mon0` vm. If you wish to test the
cross-host functionality, ssh into `mon1` or `mon2` with `vagrant ssh`.

1. Run the suite: `make run`.
1. SSH into the host: `make ssh`.
1. Upload policy information: `volcli policy upload policy1 < /testdata/ceph/policy1.json`
1. Add a docker volume with `policy/name` syntax:
  * `docker volume create -d volplugin --name policy1/foo`
1. Run a container with the volume attached:
  * `docker run -it -v policy1/foo:/mnt ubuntu bash`
1. You should have a volume mounted at `/mnt`, pointing at a `/dev/rbd#`
   device. Exit the shell to unmount the device.

To use the volume again, either `docker volume create` it on another host and
start a container, or just do it again with a different container on the same
host. Your data will be there!

`volcli` has many applications including volume and mount management. Check it
out!

## Development Instructions 

See our [CONTRIBUTING](https://github.com/contiv/volplugin/blob/master/CONTRIBUTING.md)
document as well.

Please read the `Makefile` for most targets. If you `make build` you will get
volmaster/volplugin/volcli installed on the guests, so `make run-build` if you
want a `go install`'d version of these programs on your host.
volmaster/volplugin **do not** run on anything but linux (you can use volcli,
however, on other platforms).

If you wish to run the tests, `make test`. The unit tests (`make unit-test`)
live throughout the codebase as `*_test` files. The system tests / integration
tests (`make system-test`) live in the `systemtests` directory.

[ReportCard-URL]: https://goreportcard.com/report/github.com/contiv/volplugin
[ReportCard-Image]: http://goreportcard.com/badge/contiv/volplugin
[Build-Status-URL]: http://contiv.ngrok.io/job/Volplugin%20Push%20Build%20Master
[Build-Status-Image]: http://contiv.ngrok.io/buildStatus/icon?job=Volplugin%20Push%20Build%20Master


# volplugin
Contiv Volume plugin [volplugin](https://github.com/contiv/volplugin "Title")
is a Ceph volume driver, and policy system that works well within a Docker
ecosystem. It can automatically move your storage with your containers, rather
than pinning containers to specific hosts to take advantage of their storage.

## Getting started

Getting started describes setting up a test environment with three VMs. Once
the test environment is setup see the [**Configure Services**](#Configure Services).

#### Prerequisites

Please read and follow the instructions in the prerequisites section of the
volplugin
[README](https://github.com/contiv/volplugin/blob/master/README.md#prerequisites)
before completing the following:

### Clone and build the project

### On Linux (without a VM)

Clone the project:

```
git clone https://github.com/contiv/volplugin
```

Build the project:

```
make run-build
```


The command `make run-build` installs utilities for building the software in
the `$GOPATH`, as well as the `volmaster`, `volplugin` and `volcli` utilities.

### Everywhere else (with a VM):

Clone the project:

```
git clone https://github.com/contiv/volplugin
```


Build the project:

```
make start
```


The build and binaries will be on the VM in the following directory `/opt/golang/bin`.

## Do it yourself

### Installing Dependencies

Use the Contiv [nightly releases](https://github.com/contiv/volplugin/releases)
when following these steps:

**Note:** Using the nightly builds is simpler than building the applications.

Install the dependencies in the following order:

1. Follow the [Getting Started](https://github.com/coreos/etcd/releases/tag/v2.2.0) to install [Etcd](https://coreos.com/etcd/docs/latest/getting-started-with-etcd.html).
  * Currently versions 2.0 and later are supported.

2. Follow the [Ceph Installation Guide](http://docs.ceph.com/docs/master/install/) to install [Ceph](http://ceph.com).
3. Configure Ceph with [Ansible](https://github.com/ceph/ceph-ansible).

  **Note**: See the [README](https://github.com/contiv/volplugin/blob/master/README.md#running-the-processes)
  for pre-configured VMs that work on any UNIX operating system to simplify
    Ceph installation.

4. Upload a global configuration. You can find an example one [here](https://github.com/contiv/volplugin/blob/master/systemtests/testdata/global1.json)

5. Start volmaster in debug mode (as root):

```
volmaster &
```

**Note**: volmaster debug mode is very noisy and is not recommended. Therefore,
avoid using it with background processes. volplugin currently connects to
volmaster using port 9005, however in the future it is variable.

6. Start volsupervisor (as root):

```
volsupervisor &
```


**Note**: volsupervisor debug mode is very noisy and is not recommended.

7.  Start volplugin in debug mode (as root):

```
volplugin &
```

If running volplugin on multiple hosts, use the `--master` flag to
provide a ip:port pair to connect to over http. By default it connects to
`127.0.0.1:9005`.

## Configure Services

Ensure Ceph is fully operational, and that the `rbd` tool works as root.

Upload a policy:

```
volcli policy upload policy1
```


**Note**: It accepts the policy from stdin, e.g.: `volcli policy upload policy1 < mypolicy.json`
Examples of a policy are in [systemtests/testdata](https://github.com/contiv/volplugin/tree/master/systemtests/testdata).

### Creating a container with a volume


Create a volume that refers to the volplugin driver:

```
docker volume create -d volplugin --name policy1/test
```

**Notes**:

* `test` is the name of the volume, and is located under policy `policy1`,
 which is uploaded with `volcli policy upload.`
* The volume will inherit the properties of the policy. Therefore, the
volume will be of appropriate size, iops, etc.
* There are numerous options (see below) to declare overrides of most parameters in the policy configuration.
* Run a container that uses the policy:

```
docker run -it -v policy1/test:/mnt ubuntu bash
```
* Run `mount | grep /mnt` in the container.


**Note**: `/dev/rbd#`should be attached to that directory.

* Once a multi-host system is setup, anytime the volume is not mounted, it
can be mounted on any host that has a connected rbd client available, and
volplugin running.
