# 1. Securing and Hardening Docker Containers

- [1. Securing and Hardening Docker Containers](#1-securing-and-hardening-docker-containers)
  - [1.1. Control Groups](#11-control-groups)
  - [1.2. Kernel Namespaces](#12-kernel-namespaces)
    - [1.2.1. Secure the User Namespace](#121-secure-the-user-namespace)
  - [1.3. AppArmor](#13-apparmor)
  - [1.4. Seccomp profiles](#14-seccomp-profiles)
  - [1.5. Linux Capabilities](#15-linux-capabilities)
  
## 1.1. Control Groups

cgroups are a feature of the linux kernel, that allow users to limit the access that processes and containers have to system resources such as CPU, RAM, IOPS and the network. 

Docker uses cgroups to enforce constraints on docker containers to prevent a single container from consuming all resources on the host i.e. to prevent forkbombs etc. This is particularly important in multi-tenant platforms.

## 1.2. Kernel Namespaces

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

### 1.2.1. Secure the User Namespace
In some circumstances, certain processes and applications may be required to run as root within the container. To secure the user namespace it is possible to map the container root user to a low privileged user on the host. The mapped user is assigned a range of UIDs which function within the namespace as normal UIDs from 0 to 65536, but have no privileges on the host machine itself.

The remapping itself is handled by two files: /etc/subuid and /etc/subgid. Each file works the same, but one is concerned with the user ID range, and the other with the group ID range. 

For detailed information see https://docs.docker.com/engine/security/userns-remap/

## 1.3. AppArmor
Application Armor is a linux security module that protects an operating system and its applications from security threats. As AppArmor allows us to restrict program capabilities it can be used to protect docker from some security threats by restricting network access control, file permissions and mount rules etc.

To use AppArmor, a system administrator must associate a security profile with each application. Docker expects to find an AppArmor policy loaded and enforced, by default Docker automatically generates and loads a profile for containers named `docker-default`. The Docker binary generates this profile in tmpfs and then loads it into the kernel.

To use a new profile, the profile must be created and then loaded into AppArmor before it can be specified with a container. For detailed information regarding creating AppArmor profiles see https://doc.opensuse.org/documentation/leap/security/html/book-security/cha-AppArmor-profiles.html#sec-AppArmor-profiles-mount

- Load a new profile into AppArmor:
```
apparmor_parser -r -W /path/to/profile
```

- To overide the default security policy you can pass a new apparmor profile using the `security-opt` flag:
```
docker run --rm -ti --security-opt apparmor=<newprofile> alpine sh
```
- Unload a profile from AppArmor
```
apparmor_parser -R /path/to/profile
```
## 1.4. Seccomp profiles
Secure Computing mode (seccomp) is another Linux kernel feature. It can be used to restrict the actions available within the container. The seccomp() system call operates on the seccomp state of the calling process. You can use this feature to restrict your application’s access. Note - This feature is available only if Docker has been built with seccomp and the kernel is configured with CONFIG_SECCOMP enabled.

Seccomp can act like a firewall for system calls by filtering the system calls issued by a program. Like AppArmor Secomp profiles can be configured to filter which system calls can be run from within a container and must be loaded per container. By default Docker attaches the docker seccomp profile to containers which blocks several service calls such as `insmod` to reject loading kernel modules. The default profile can be changed with the `--security-opt seccomp=` flag.

Note - running containers as `--privileged` disables the AppArmor and Seccomp protections even if specifying a security profile.

## 1.5. Linux Capabilities