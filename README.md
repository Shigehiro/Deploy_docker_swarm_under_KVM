Set up Docker Swarm Cluster under LXD(KVM) environment
---

- [Set up Docker Swarm Cluster under LXD(KVM) environment](#set-up-docker-swarm-cluster-under-lxdkvm-environment)
- [About this](#about-this)
- [Tested environment](#tested-environment)
- [Walkthrough](#walkthrough)
- [Add additional manager nodes](#add-additional-manager-nodes)
- [Add additional worker nodes](#add-additional-worker-nodes)

## About this

This Ansible playbook sets up Docker swarm cluster under KVM.

- launch virtual machines managed by LXD
- install docker packages via apt
- configure docker swarm
- add swarm manager nodes
- add swarm worker nodes

## Tested environment

- KVM host: Ubuntu 20.04
- VM: Ubuntu 20.04 ( Other Linux distribtions are not supported. )
- VMs are managed by LXD, like `lxc launch --vm ...`
- Note that this playbook deploys docker swarm cluster on **Virtual Machines** managed by **LXD**, this playbook does not deploy swarm cluster under **LXC containers**.

## Walkthrough

There are no VMs.
```shell
$ lxc list 
+------+-------+------+------+------+-----------+
| NAME | STATE | IPV4 | IPV6 | TYPE | SNAPSHOTS |
+------+-------+------+------+------+-----------+
```

prepare an inventory file.
```shell
$ cat inventory.ini 
[localhost]
127.0.0.1

[instances]
docker01 ansible_connection=lxd
docker02 ansible_connection=lxd
docker03 ansible_connection=lxd

[swarm_managers]
docker01 ansible_connection=lxd

[swarm_workers]
docker02 ansible_connection=lxd
docker03 ansible_connection=lxd
```

add the above nodes in host_vars/localhost.yml like below.

```yaml
$ grep -A3 instance_name host_vars/localhost.yml 
instance_name:
  - docker01
  - docker02
  - docker03
```

run the playbook. 
deploy one manager and two worker nodes.
```shell
$ ansible-playbook -i ./inventory.ini main.yml 
```

```shell
$ lxc list -cn4
+----------+------------------------------+
|   NAME   |             IPV4             |
+----------+------------------------------+
| docker01 | 172.18.0.1 (docker_gwbridge) |
|          | 172.17.0.1 (docker0)         |
|          | 10.9.109.3 (enp5s0)          |
+----------+------------------------------+
| docker02 | 172.18.0.1 (docker_gwbridge) |
|          | 172.17.0.1 (docker0)         |
|          | 10.9.109.149 (enp5s0)        |
+----------+------------------------------+
| docker03 | 172.18.0.1 (docker_gwbridge) |
|          | 172.17.0.1 (docker0)         |
|          | 10.9.109.48 (enp5s0)         |
+----------+------------------------------+
```

```shell
$ lxc exec docker01 -- docker node ls
ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
ig4uvqjgym98b4qtubpq0f4m6 *   docker01   Ready     Active         Leader           20.10.7
glf1i9cew5kbycbuysr5fnsry     docker02   Ready     Active                          20.10.7
58ojbzvmulmh92ui0g7ksjuwz     docker03   Ready     Active                          20.10.7
```
## Add additional manager nodes

add two additional manager nodes(docker04, docker05)

```shell
$ grep ^"\[swarm_managers" inventory.ini -A3
[swarm_managers]
docker01 ansible_connection=lxd
docker04 ansible_connection=lxd
docker05 ansible_connection=lxd
```

```shell
$ grep -A5 instance_name host_vars/localhost.yml 
instance_name:
  - docker01
  - docker02
  - docker03
  - docker04
  - docker05
```

then run the playbook.

```shell
$ ansible-playbook -i ./inventory.ini main.yml 
```

## Add additional worker nodes

You can also deploy additional worker nodes.

```text
$ grep '^\[swarm_worker' inventory.ini -A1
[swarm_workers]
docker02 ansible_connection=lxd
```