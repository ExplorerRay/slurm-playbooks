# Ansible playbook for building Slurm cluster in Debian

## Usage
1. Revise inventory file to match your environment.
2. Run the playbook depends on the area you want to deploy:
* Test:
   ```bash
   ansible-playbook -i inventory/hosts.yml hpc.yml -e "var_host=all"
   ```
3. ~~DEBUG~~

You can also run the playbook with specific tags for not running all tasks (like CUDA installation):
```bash
ansible-playbook -i inventory/hosts.yml hpc.yml -t <tag_name> -e "var_host=all"
```

## Default settings

* `shared_slurm_config` and `shared_munge` in group_vars determine whether the Slurm configuration files and munge key should be shared via NFS.
* NFS currently supports only one server. To change the accessible subnets (default: `0.0.0.0/0`), edit `roles/nfs/defaults/main.yml`.
* Both login and compute nodes mount NFS exports from the storage node.

## Main roles
### NFS role
* In the `nfs` role, `defaults/main.yml` allows you to define  `nfs_shared_home` option controls whether a separate home directory should be set up on the storage node. If home directory synchronization is already handled or not needed, set this to false.
    * If you don't manage the storage node (i.e., you don’t have permission), you can modify `nfs_admin` in `roles/nfs/defaults/main.yml` and set it to `false`.

### Munge role
* For munge, the first control node is used to initially create the key. If `shared_munge` is not enabled, the key will be distributed to other nodes.

### CUDA role
* This role is used to install CUDA, CUDNN and NVIDIA drivers. The default installed CUDA version is 12.8. If you want to change it, set `cuda_version` in `roles/cuda/defaults/main.yml`.
* Add a service for `/dev/nvidia*` to come out when booting.

### Slurm install role
This role install packages in nodes depend on their node type defined in the inventory file.
* The default Slurm installation version is 24.11.1. To change it, revise `roles/slurm_install/defaults/main.yml`.
* Setting MariaDB configuration such as revising bind address, some innodb settings and galera-related settings (WIP).

### Slurm config role
Do some configurations for Slurm such as DB slurm setting with MySQL module, copying predefined configurations, and so on.
* We use MariaDB for Slurm database. You will prompt to modify `slurm_db_pass` when running.

## Other roles
* The `debian_init` role mainly handles workarounds such as modifying hostname, /etc/hosts, and creating user accounts. If you're using DNS or LDAP and they are properly configured, you can skip this role.

## Notice
* In the inventory, the DB and slurmdbd are separated because slurmdbd supports multiple instances (e.g., for backup), but multiple DBs are not yet supported.
