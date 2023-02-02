Docker CE Install and Configuration
===================================

## Installation notes

**Centos**

Docker requires packages:
```
sudo yum install -y device-mapper-persistent-data lvm2
```
Docker repo adds:
```
sudo yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
```
Docker installation:
```
sudo yum install -y docker-ce-18.09.5 docker-ce-cli-18.09.5 containerd.io
```
service start
```
sudo systemctl start docker
sudo systemctl enable docker
```
Add user, no need `sudo` to run
```
sudo usermod -a -G docker cloud_user
exit
```
log-in again

[**ubuntu**](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

packages:
```
sudo apt-get update
sudo apt-get -y install \
apt-transport-https \
ca-certificates \
curl \
gnupg-agent \
software-properties-common
```
Add the Docker GPG:
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# sudo apt-key fingerprint 0EBFCD88 # not sure where it is from
```
Add repo
```
sudo add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```
Install Docker
```
sudo apt-get install -y docker-ce=5:18.09.5~3-0~ubuntu-bionic docker-ce-cli=5:18.09.5~3-0~ubuntu-bionic containerd.io
```
permission:
```
sudo usermod -a -G docker cloud_user
```
Log out and back in.

## Storage drivers

Docker storage drivers are used by the Docker daemon to manage the storage of images and container data on the host machine. They provide an abstraction layer between the Docker daemon and the underlying storage system, allowing the daemon to use different storage backends without modification.

- `overlay2` is a storage driver for Docker that uses the overlay filesystem to provide a union filesystem for containers. It allows multiple read-write layers to be stacked on top of a read-only base layer, allowing for efficient storage of container data. It also supports other features like copy-on-write, which allows multiple containers to share the same read-only base layer, reducing storage usage.

- `devicemapper` is another storage driver for Docker that uses the device mapper kernel module to provide a thin provisioning system for containers. It allows multiple containers to share the same physical storage, while providing each container with its own virtual device. It also supports features like copy-on-write and snapshots, which allow for efficient storage and management of container data. `devicemapper` uses the device-mapper kernel module to create thin-provisioned storage volumes for containers.

Both `overlay2` and `devicemapper` are supported by Docker and provide similar functionality, but `overlay2` is the recommended storage driver for most systems and is the default on many Linux distributions.

- `aufs`: This is an older storage driver that is based on the Linux kernel's Advanced Multi-Layered Unification Filesystem (AUFS). It is considered to be stable and well-tested, but it is not supported on all platforms.

- `btrfs`: This storage driver is based on the B-tree file system (Btrfs), which is a copy-on-write filesystem that is optimized for use with containers. It is considered to be experimental and is not recommended for production use.

- `zfs`: This storage driver is based on the Zettabyte File System (ZFS), which is a highly scalable and feature-rich filesystem that is optimized for use with containers. It is considered to be experimental and is not recommended for production use.

- `vfs`: This is the simplest storage driver, it uses the host's filesystem and it is not recommended for production use.

**To check the system:**
```
± docker info | grep 'Storage Driver' 
 Storage Driver: overlay2
```
set the storage driver explicitly for Docker daemon:
```
sudo vi /usr/lib/systemd/system/docker.service
```
or by flag on runtime:
```
ExecStart=/usr/bin/dockerd --storage-driver devicemapper ...
```
reload Systemd and restart Docker:
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```
**using the daemon configuration file** (note it may not exists)
```
sudo vi /etc/docker/daemon.json
{
"storage-driver": "devicemapper"
}
```
Restart 
```
sudo systemctl restart docker
sudo systemctl status docker
```

## Running containers

**basic:** `docker run image`

**expanded:** `docker run [OPTIONS] IMAGE[:TAG] [COMMAND] [ARG...]`

example 1:
```
docker run --name my_container busybox echo Hello World
```
- `docker run`
- `--name my_container` -> options
- `busybox` -> image
- `echo` -> command
- `Hello world` -> args to image
example 2:
```
docker run -d --name nginx --restart unless-stopped -p8080:80 nginx
```
- `docker run`
- `-d --name nginx --restart unless-stopped -p 8080:80` -> options
    - `-d` detached
    - `--name nginx` run with name
    - `--restart unless-stopped` restart policy
    - `-p <host_port:<container_port>` port mapping
- `nginx` -> image
```
Options: 
      --add-host list                  Add a custom host-to-IP mapping (host:ip) 
  -a, --attach list                    Attach to STDIN, STDOUT or STDERR
      --blkio-weight uint16            Block IO (relative weight), between 10 and 1000, or 0 to disable (default 0) 
      --blkio-weight-device list       Block IO weight (relative device weight) (default []) 
      --cap-add list                   Add Linux capabilities  
      --cap-drop list                  Drop Linux capabilities    
      --cgroup-parent string           Optional parent cgroup for the container   
      --cgroupns string                Cgroup namespace to use (host|private) 
                                       'host':    Run the container in the Docker host's cgroup namespace 
                                       'private': Run the container in its own private cgroup namespace  
                                       '':        Use the cgroup namespace as configured by the    
                                                  default-cgroupns-mode option on the daemon (default)
      --cidfile string                 Write the container ID to the file 
      --cpu-period int                 Limit CPU CFS (Completely Fair Scheduler) period   
      --cpu-quota int                  Limit CPU CFS (Completely Fair Scheduler) quota
      --cpu-rt-period int              Limit CPU real-time period in microseconds
      --cpu-rt-runtime int             Limit CPU real-time runtime in microseconds 
  -c, --cpu-shares int                 CPU shares (relative weight) 
      --cpus decimal                   Number of CPUs 
      --cpuset-cpus string             CPUs in which to allow execution (0-3, 0,1) 
      --cpuset-mems string             MEMs in which to allow execution (0-3, 0,1) 
  -d, --detach                         Run container in background and print container ID 
      --detach-keys string             Override the key sequence for detaching a container  
      --device list                    Add a host device to the container 
      --device-cgroup-rule list        Add a rule to the cgroup allowed devices list 
      --device-read-bps list           Limit read rate (bytes per second) from a device (default []) 
      --device-read-iops list          Limit read rate (IO per second) from a device (default []) 
      --device-write-bps list          Limit write rate (bytes per second) to a device (default []) 
      --device-write-iops list         Limit write rate (IO per second) to a device (default []) 
      --disable-content-trust          Skip image verification (default true) 
      --dns list                       Set custom DNS servers                                              
      --dns-option list                Set DNS options                                                     
      --dns-search list                Set custom DNS search domains                                       
      --domainname string              Container NIS domain name                                           
      --entrypoint string              Overwrite the default ENTRYPOINT of the image 
  -e, --env list                       Set environment variables
      --env-file list                  Read in a file of environment variables 
      --expose list                    Expose a port or a range of ports 
      --gpus gpu-request               GPU devices to add to the container ('all' to pass all GPUs)
      --group-add list                 Add additional groups to join   
      --health-cmd string              Command to run to check health    
      --health-interval duration       Time between running the check (ms|s|m|h) (default 0s) 
      --health-retries int             Consecutive failures needed to report unhealthy 
      --health-start-period duration   Start period for the container to initialize before starting health-retries countdown (ms|s|m|h) (default 0s)
      --health-timeout duration        Maximum time to allow one check to run (ms|s|m|h) (default 0s)
      --help                           Print usage 
  -h, --hostname string                Container host name
      --init                           Run an init inside the container that forwards signals and reaps processes
  -i, --interactive                    Keep STDIN open even if not attached
      --ip string                      IPv4 address (e.g., 172.30.100.104)
      --ip6 string                     IPv6 address (e.g., 2001:db8::33)
      --ipc string                     IPC mode to use
      --isolation string               Container isolation technology
      --kernel-memory bytes            Kernel memory limit
  -l, --label list                     Set meta data on a container
      --label-file list                Read in a line delimited file of labels
      --link list                      Add link to another container
      --link-local-ip list             Container IPv4/IPv6 link-local addresses
      --log-driver string              Logging driver for the container
      --log-opt list                   Log driver options
      --mac-address string             Container MAC address (e.g.,92:d0:c6:0a:29:33)
  -m, --memory bytes                   Memory limit
      --memory-reservation bytes       Memory soft limit
      --memory-swap bytes              Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --memory-swappiness int          Tune container memory swappiness (0 to 100) (default -1)
      --mount mount                    Attach a filesystem mount to the container
      --name string                    Assign a name to the container
      --network network                Connect a container to a network
      --network-alias list             Add network-scoped alias for the container
      --no-healthcheck                 Disable any container-specified HEALTHCHECK
      --oom-kill-disable               Disable OOM Killer
      --oom-score-adj int              Tune host's OOM preferences (-1000 to 1000)
      --pid string                     PID namespace to use
      --pids-limit int                 Tune container pids limit (set -1 for unlimited)
      --platform string                Set platform if server is multi-platform capable
      --privileged                     Give extended privileges to this container
  -p, --publish list                   Publish a container's port(s) to the host
  -P, --publish-all                    Publish all exposed ports to random ports
      --pull string                    Pull image before running ("always"|"missing"|"never") (default "missing")
      --read-only                      Mount the container's root filesystem as read only
      --restart string                 Restart policy to apply when a container exits (default "no") (no|on-failure|always|unless-stopped)
      --rm                             Automatically remove the container when it exits
      --runtime string                 Runtime to use for this container
      --security-opt list              Security Options
      --shm-size bytes                 Size of /dev/shm
      --sig-proxy                      Proxy received signals to the process (default true)
      --stop-signal string             Signal to stop a container (default "SIGTERM")
      --stop-timeout int               Timeout (in seconds) to stop a container
      --storage-opt list               Storage driver options for the container
      --sysctl map                     Sysctl options (default map[])
      --tmpfs list                     Mount a tmpfs directory
  -t, --tty                            Allocate a pseudo-TTY
      --ulimit ulimit                  Ulimit options (default [])
  -u, --user string                    Username or UID (format: <name|uid>[:<group|gid>])
      --userns string                  User namespace to use
      --uts string                     UTS namespace to use
  -v, --volume list                    Bind mount a volume
      --volume-driver string           Optional volume driver for the container
      --volumes-from list              Mount volumes from the specified container(s)
  -w, --workdir string                 Working directory inside the container
```
## logging

**`Logging drivers`** are various mechanisms/frameworks [supporting logging](https://docs.docker.com/config/containers/logging/configure/) and getting info out of container. They are set in `log-driver` and `log-opts` options in `/etc/docker/daemon.json`. They can be set as an option in `docker run`.

The default setting:
```
$ docker info | grep Logging
 Logging Driver: json-file

$ docker info --format '{{.LoggingDriver}}'
json-file
```
To find on running container:
```
$ docker inspect -f '{{.HostConfig.LogConfig.Type}}' nginx      
json-file
```
It can be changed globally (default) by editing or just adding `/etc/docker/daemon.json`:
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3",
    "labels": "production_status",
    "env": "os,customer"
  }
}
```
after this operation docker daemon needs restart: `sudo systemctl restart docker`

Drivers:
| name | function |
| ---- | -------- |
| **none**| No logs are available for the container and docker logs does not return any output.
| **local**| Logs are stored in a custom format designed for minimal overhead.
| **json-file**| The logs are formatted as JSON. The default logging driver for Docker.
| **syslog**| Writes logging messages to the syslog facility. The syslog daemon must be running on the host machine.
| **journald**| Writes log messages to journald. The journald daemon must be running on the host machine.
| **gelf**| Writes log messages to a Graylog Extended Log Format (GELF) endpoint such as Graylog or Logstash.
| **fluentd**| Writes log messages to fluentd (forward input). The fluentd daemon must be running on the host machine.
| **awslogs**| Writes log messages to Amazon CloudWatch Logs.
| **splunk**| Writes log messages to splunk using the HTTP Event Collector.
| **etwlogs**| Writes log messages as Event Tracing for Windows (ETW) events. Only available on Windows platforms.
| **gcplogs**| Writes log messages to Google Cloud Platform (GCP) Logging.
| **logentries**| Writes log messages to Rapid7 Logentries.

>Note: Docker 20.10 and up introduces `dual logging`, which uses a [local buffer](https://docs.docker.com/config/containers/logging/dual-logging/) that allows you to use the docker logs command for any logging driver.

## Docker Swarm

Build-in feature to stand-up distributed cluster of docker machines to run containers. It is used to run docker app on multiple physical machines.

### Docker Swarm Manager

The manager is stood up during initlisation `docker swarm init`. It may required the network advertisement ex: `docker swarm init --advertise-addr <ip_to_advertise`.

In other words, the fact that cluster is initialised on a particular server, it will make it Swarm Manger.

If this is done on the cloud, use private ip of the server. The main reason is that private IP will have all ports open, and the public may run into firewall problems.

```
$ docker swarm init --advertise-addr 192.168.1.17                                                                                           
Swarm initialized: current node (uwqeswfd0y78s7d4ewbt2rob1) is now a manager.                                                               
                                                                                                                                            
To add a worker to this swarm, run the following command:                                                                                   
                                                                                                                                            
    docker swarm join --token <token> 192.168.1.17:2377       
                                                                                                                                            
To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions. 
```
The swarm can be viewed with:
```
$ docker node ls
ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
uwqeswfd0y78s7d4ewbt2rob1 *   ActEnf     Ready     Active         Leader           20.10.23
```
or with full info:
```
$ docker info 
Client:                      
 Context:    default              
 Debug Mode: false    
 Plugins:                   
  app: Docker App (Docker Inc., v0.9.1-beta3)     
  buildx: Docker Buildx (Docker Inc., v0.10.0-docker)
  compose: Docker Compose (Docker Inc., v2.15.1) 
  scan: Docker Scan (Docker Inc., v0.23.0)
Server:     
 Containers: 0 
  Running: 0 
  Paused: 0
  Stopped: 0 
 Images: 14
 Server Version: 20.10.23
 Storage Driver: overlay2
  Backing Filesystem: extfs 
  Supports d_type: true 
  Native Overlay Diff: true     
  userxattr: false     
 Logging Driver: json-file 
 Cgroup Driver: cgroupfs  
 Cgroup Version: 1 
 Plugins:  
  Volume: local
  Network: bridge host ipvlan macvlan null overlay 
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog 
 Swarm: active
  NodeID: uwqeswfd0y78s7d4ewbt2rob1 
  Is Manager: true
  ClusterID: 3ru64if1du026rcyoukouzuwb 
  Managers: 1      
  Nodes: 1                                      
  Default Address Pool: 10.0.0.0/8
  SubnetSize: 24                                
  Data Path Port: 4789                          
  Orchestration:                                
   Task History Retention Limit: 5              
  Raft:                                         
   Snapshot Interval: 10000                     
   Number of Old Snapshots to Retain: 0                                                          
   Heartbeat Tick: 1                            
   Election Tick: 10                            
  Dispatcher:                                   
   Heartbeat Period: 5 seconds                  
  CA Configuration:                             
   Expiry Duration: 3 months                    
   Force Rotate: 0                              
  Autolock Managers: false                      
  Root Rotation In Progress: false              
  Node Address: 192.168.1.17                    
  Manager Addresses:                            
   192.168.1.17:2377  
```
### Worker nodes

To add worker to existing manger, from the manager server, run:
```
$ docker swarm join-token worker
To add a worker to this swarm, run the following command:

    docker swarm join --token <token> 192.168.1.17:2377
```
It will display full command to run to create a worker

### Backing and restoring

All data are managed on manager server

- Stop the docker service:
  - `sudo systemctl stop docker`
- back-up directory content
  - `sudo tar -zvcf backup.tar.gz /var/lib/docker/swarm`
- Start the docker service:
  - `sudo systemctl start docker`

The process can be automated and to avoid any downtime, multiple managers can be running.

### Restoring the backup

It is simply unpacking context of directory backup
- Stop the docker service:
  - `sudo systemctl stop docker`
- remove directory content to avoid conflicts
  - `sudo rm -rf /var/lib/docker/swarm/*`
- unpack directory content
  - `sudo tar -zxcf backup.tar.gz -C /var/lib/docker/swarm/`
- Start the docker service:
  - `sudo systemctl start docker`

### Managing docker

Docker relies on Unix like Namespaces. They separate the running application, which is not even aware of running inside dockerised system.

- **`Namespaces`** are Linux technology allowing processes to be isolated in terms of resources they see.
> wrap certain global system resources in an abstraction layer. This makes it appear like the processes within a namespace have their own isolated instance of the resource.
  - `pid` - process isolation
  - `net` - network interfaces
  - `ipc` - inter-process communication
  - `mnt` - filesystem mounts
  - `uts` - kernel and version identifiers
  - **`user namespace`** - `cgroups (control groups)` - permission to run resources. Requires special config. Allows mapping root@container to unprivileged_user@host

  The typical namespaces:
  ```
  ls -Gg /proc/self/ns/
total 0
lrwxrwxrwx 1 0 Feb  4 12:08 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 0 Feb  4 12:08 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 0 Feb  4 12:08 mnt -> 'mnt:[4026531841]'
lrwxrwxrwx 1 0 Feb  4 12:08 net -> 'net:[4026531840]'
lrwxrwxrwx 1 0 Feb  4 12:08 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 0 Feb  4 12:08 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 0 Feb  4 12:08 time -> 'time:[4026531834]'
lrwxrwxrwx 1 0 Feb  4 12:08 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx 1 0 Feb  4 12:08 user -> 'user:[4026531837]'
lrwxrwxrwx 1 0 Feb  4 12:08 uts -> 'uts:[4026531838]'
```