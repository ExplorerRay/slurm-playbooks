# Ansible playbook for building Slurm cluster in Debian

## Usage
1. Revise inventory file to match your environment.
2. Revise `group_vars` files to match your environment.
3. Run the playbook with your desired inventory file:
   ```bash
   ansible-playbook -i inventory/hosts.yml hpc.yml
   ```
    * When running slurm_config role, you will be prompted to enter the Slurm database password (`slurm_db_pass`).

### Run specific role
You can also run the playbook with specific tags for not running all tasks:
```bash
ansible-playbook -i inventory/hosts.yml hpc.yml -t <tag_name>
```

## Group variables
We use group variables to define some settings that may vary in Production (`hpc_cluster.yml`) and Test (`hpc_test.yml`) environments. You can find them in `group_vars` directory.

* `shared_slurm_config` and `shared_munge` in group_vars determine whether the Slurm configuration files and munge key should be shared via NFS.
* `nfs_prefix` is the NFS export prefix for config (slurm & munge), which is used to mount the NFS share on all Slurm nodes.

## Default settings

* The default Lmod version is 8.7.
* Both login and compute nodes mount NFS exports from the storage node.

## Main roles
### NFS role
* In the `nfs` role, `defaults/main.yml` allows you to define  `nfs_shared_home` option controls whether a separate home directory should be set up on the storage node. If home directory synchronization is already handled or not needed, set this to `false`.
    * If you don't manage the storage node (i.e., you don’t have permission), you can modify `nfs_admin` in `roles/nfs/defaults/main.yml` and set it to `false`.
    * NFS currently supports only one server. To change the accessible subnets (default: `0.0.0.0/0`), edit `roles/nfs/defaults/main.yml`.

### Munge role
* For munge, the first control node is used to initially create the key. If `shared_munge` is not enabled, the key will be distributed to other nodes manually.

### Slurm install role
This role install packages in nodes depend on their node type defined in the inventory file.
* The default Slurm installation version is 24.11.1. To change it, revise `roles/slurm_install/defaults/main.yml`.
* Setting MariaDB configuration such as revising bind address, some innodb settings and galera-related settings.
* Galera part is only enabled when `hpc_galera` (virtual IP) is set and `hpc_db` has more than 3 nodes in inventory file.
* If your Galera cluster interface name is not `ens192`, you can change it in `roles/slurm_install/defaults/main.yml`.

### Slurm config role
Do some configurations for Slurm such as DB slurm setting with MySQL module, copying predefined configurations, and so on.
* We use MariaDB for Slurm database. You will prompt to modify `slurm_db_pass` when running.

## Other roles
* The `debian_init` role mainly handles workarounds such as modifying hostname, `/etc/hosts`. If you're using DNS or LDAP and they are properly configured, you can skip this role.

## Notice
* In the inventory, the DB and slurmdbd are separated because slurmdbd supports multiple instances (e.g., for backup). Now only support **single** DB node or **3** DB nodes for Galera.
