# Securing and Hardening Docker Containers

- [Securing and Hardening Docker Containers](#securing-and-hardening-docker-containers)
  - [Control Groups](#control-groups)
  - [Kernel Namespaces](#kernel-namespaces)
  - [AppArmor profiles](#apparmor-profiles)
  
## Control Groups

cgroups are a feature of the linux kernel, that allow users to limit the access that processes and containers have to system resources such as CPU, RAM, IOPS and the network. 

Docker uses cgroups to enforce constraints on docker containers to prevent a single container from consuming all resources on the host i.e. to prevent forkbombs etc. This is particularly important in multi-tenant platforms.

## Kernel Namespaces

Namespaces are a feature of the linux kernel that provide isolation for fundamental aspects of containers from the host OS. Namespaces provide the first and most straightforward form of isolation: processes running within a container cannot see processes running in another container, or on the host system.

By default each container also gets its own network stack, meaning that a container doesn’t get privileged access to the sockets or interfaces of another container. From a network architecture point of view, all containers on a given Docker host are sitting on bridge interfaces. This means that they are just like physical machines connected through a common Ethernet switch.

Docker namespaces:
- PID - process isolation
- NET - manages network interfaces
- IPC - access to IPC resources
- MNT - filesystem mount points
- UTS - isolates kernel and version identifiers
- UID - UserID for privilege isolation

Security Note on UID, by default a root user in a docker container has the same priviges as the root user of the host - i.e. if a mount point has been connected to the docker container the container root user can access that mount on the host OS. For this reason it is recommended not to run containers in privileged mode or as the root user.

In some circumstances processes and applications must run as root within the container. To secure the user namespace it is possible to map the container root user to a low privileged user on the host. The mapped user is assigned a range of UIDs which function within the namespace as normal UIDs from 0 to 65536, but have no privileges on the host machine itself.

The remapping itself is handled by two files: /etc/subuid and /etc/subgid. Each file works the same, but one is concerned with the user ID range, and the other with the group ID range. 

For detailed information see https://docs.docker.com/engine/security/userns-remap/

## AppArmor profiles





