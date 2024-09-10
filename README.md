為了整理 HPC 服務相關會用到的腳本或 playbook，所以創建此 project。

## Usage
* slurm 預設安裝版本為 24.05.2，若想修改，請 `-e "slurm_version=xxx"`
* Lmod 預設安裝版本為 8.7
* `group_vars`中可定義 `shared_config` 及 `shared_munge`，決定 slurm 設定檔和 munge key 要不要放在 NFS 中共用。還有`shared_home`可決定是否要再 storage 中再自己弄一個家目錄，若已經處理好家目錄同步，或是不需要的請改成`false`

## Notice
* 環境為 Debian 12
* `debian_init` 這個 role 主要在處理修改 hostname, `/etc/hosts`、創建帳號等 workaround，若有 DNS 或 LDAP 並設定好，這部分是不需要跑的。
* spack 環境，可以在`roles/spack_env/templates/`底下增加課程環境，到時候會一起創建及安裝相關套件
* login node, compute node 的 NFS 會掛 storage export 出來的
* storage 若不是自己管理(沒權限)，可以修改`roles/nfs/defaults/main.yml`的`nfs_admin`，將值改為 `False`
* 關於 inventory 的部分，將 DB 跟 slurmdbd 分隔開來，因為 slurmdbd 可以有多台(backup)，但 **DB 目前未處理**
* munge 部分，需要修改`roles/munge/defaults/main.yml`來改變可 munge 一開始 create key 的 node (預設為 `hpc051`)
* NFS 部分，目前僅有一個 server，需要修改`roles/nfs/defaults/main.yml`來改變可 access 的 subnets (預設為 `0.0.0.0/0`)
    * 共用的資料夾為 `/opt/spack` 及 `/home`
* 此 playbook 有處理 login node 及 slurmdbd node，但目前先將 control node 當 login node 及 slurmdbd node 用
    * DB 可用 mariadb server (default) 或 mysql server，並請記得修改`roles/slurm_config/defaults/main.yml`裡面的`slurm_db_pass`
    * 若選擇 mysql server，mysql 的 apt config .deb 檔需要自行下載，並放入`roles/slurm_install/files/`
        * 因為用 ansible 的 get_url 載 mysql 的檔案會無法正常下載，所以才改用 copy 的方式
