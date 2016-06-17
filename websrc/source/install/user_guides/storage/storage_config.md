## Contiv Storage Configuration

This section describes ways to manipulate 'volplugin' using configuration and options.

### Volume Formatting

Because of limitations in the docker volume implementation, we use a *pattern* to describe volumes to docker. This pattern is `policy-name/volume-name`. The pattern is supplied to `docker volume create --name` and transfers to `docker run -v`.

For example, a typical use of `volplugin` follows. For this example, assume a policy named `policy1` has been uploaded:

```
$ docker volume create -d volplugin --name policy1/new_volume_1
$ docker run -it -v policy1/new_volume_1:/mnt ubuntu bash
```

This pattern creates a volume called `new_volume_1` in `policy1`'s default Ceph pool. To change the pool (or other options), see "Driver Options" below.

### JSON Global Configuration

Global configuration modifies the whole system through the `volmaster`, `volplugin`, and `volsupervisor` services. You manipulate global configuration settins with the `volcli global` command set.

A global configuration looks like this:

```javascript
{
  "TTL": 60,
  "Debug": true,
  "Timeout": 5,
  "MountPath": "/mnt/ceph"
}
```

Options:

- TTL: Time (in seconds) for a mount record to timeout if a volplugin dies
- Debug: A Boolean value indicating whether or not to enable debug traps/logging
- Timeout: Time limit (in minutes) after which a command is terminated 
- MountPath: The base path used for mount directories. Directories are structured beneath this root in `policy/volume` format.

### JSON Tenant Configuration

Default volume parameters are specified in a tenant configuration written in JSON. The configuration is uploaded to 'etcd' by the `volcli` tool.

Here is an example of a tenant configuration:

```javascript
{
  "backends": {
    "crud": "ceph",
    "mount": "ceph",
    "snapshot": "ceph"
  },
  "driver": {
    "pool": "rbd"
  },
  "create": {
    "size": "10MB",
    "filesystem": "btrfs"
  },
  "runtime": {
    "snapshots": true,
    "snapshot": {
      "frequency": "30m",
      "keep": 20
    },
    "rate-limit": {
      "write-iops": 1000,
      "read-iops": 1000,
      "write-bps": 100000000,
      "read-bps": 100000000
    }
  },
  "filesystems": {
    "ext4": "mkfs.ext4 -m0 %",
    "btrfs": "mkfs.btrfs %",
    "falsefs": "/bin/false"
  }
}
```

The parameters are:

- `filesystems`: A policy-level map of the filesystem and `mkfs` command.
  - Commands are run when the filesystem is specified and the volume does not already exist.
  - Commands run in a POSIX (not bash, zsh) shell.
  - If the `filesystems` block is omitted, `mkfs.ext4 -m0 %` is applied to all volumes within this policy. This command is used by the creation-time volume parameter `filesystem`. The `%` symbol is replaced with the device to format.
- `backends`: The storage backends to use for different operations. Not all drivers are compatible with each other. Currently only the `ceph` driver is supported.
  - `crud`: Create/delete operations driver name.
  - `mount`: Mount operations driver name.
  - `snapshot`: Snapshot operations driver name.
- `driver`: Driver-specific options.
	- `pool`: the Ceph pool to use.
- `create`: creation-time options.
	- `size`: The size of the volume.
  - `filesystem`: The filesystem to use. See the `filesystems` parameter.
- `runtime`: Runtime options. These options can be changed; the changes are applied to mounted volumes almost immediately.
  - `snapshots`: Use the snapshots feature.
  - `snapshot`: Map of the following parameters:
    - `frequency`: The time between snapshots.
    - `keep`: The number of snapshots to keep. The oldest ones are deleted first.
  - `rate-limit`: Map of the following rate-limiting parameters:
    - `write-iops`: Write I/O weight.
    - `read-iops`: Read I/O weight.
    - `write-bps`: Write bytes per second.
    - `read-bps`: Read bytes per second.

You supply the tenant configuration by using `volcli policy upload <policy name>`. Provide the JSON as standard input. For example, if your file is `policy2.json`:

```
$ volcli policy upload myTenant < policy2.json
```

### Driver Options

Driver options are passed at `docker volume create` time with the `--opt` flag.  Driver options are `key=value` pairs. For example:

```
docker volume create -d volplugin \
  --name policy2/image \
  --opt size=1000
```

The options are as follows:

- `size`: The size (in MB) for the volume.
- `snapshots`: Take snapshots or not. Affects future options with `snapshot` in the key name.
  - The value must satisfy [this specification](https://golang.org/pkg/strconv/#ParseBool).
- `snapshots.frequency`: The frequency at which snapshots are taken.
- `snapshots.keep`: The number of snapshots to keep.
- `filesystem`: The named filesystem to create. See the JSON Configuration section for more information on this.
- `rate-limit.write.iops`: Write IOPS.
- `rate-limit.read.iops`: Read IOPS.
- `rate-limit.read.bps`: Read bytes per second.
- `rate-limit.write.bps`: Write bytes per second.
