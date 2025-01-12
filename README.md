# PiCluster

Ansible playbook for managing my MicroK8s PiCluster.

# Prerequisites

Each Pi is imaged with Ubuntu Server 64-bit. The playbook is currently confirmed to work on Ubuntu 24.04 LTS.

# Setup

You should ensure that the machine you are running the Playbook from can connect to each Pi in the cluster using ssh keys.

Update `inventory.yaml` with the IP address and sudo password for each Pi in the cluster.
Additional Pi's can be added to hosts:
```yaml
        piXXX:
          ansible_host:
          ansible_user:
          ansible_become: true
          ansible_become_password:
```

# Pre-run

Before running the playbook, ensure you can connect to each Pi by running:
```shell
ansible edge -i inventory.yaml -m ping
```

You should get a `pong` returned for each Pi defined in `hosts`.

# Running the Playbook

```shell
ansible-playbook -i inventory.yaml playbook.yaml
```
