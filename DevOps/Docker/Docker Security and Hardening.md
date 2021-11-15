# Securing and Hardening Docker Containers

- [Securing and Hardening Docker Containers](#securing-and-hardening-docker-containers)
  - [Cgroups](#cgroups)
  - [kernel Namespaces](#kernel-namespaces)

## Cgroups 

cgroups can be used to limit the reources a container can use, ie. to prevent forkbombs etc... e.g. limit pids = pids.max 

## kernel Namespaces

The user namespace is used to create an isolated namespace for UIDs within the container. However by default the root user of a container has similar permissions to the root user of the host machine. For this reason it is recommended not to run containers in privileged mode or as the root user.

However in some circumstances processes and applications must run as root within the container. To secure the user namespace it is possible to map the container root user to a low privileged user on the host. The mapped user is assigned a range of UIDs which function within the namespace as normal UIDs from 0 to 65536, but have no privileges on the host machine itself.

The remapping itself is handled by two files: /etc/subuid and /etc/subgid. Each file works the same, but one is concerned with the user ID range, and the other with the group ID range.

https://docs.docker.com/engine/security/userns-remap/



