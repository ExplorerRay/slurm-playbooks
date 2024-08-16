為了整理 HPC 服務相關會用到的腳本或 playbook，所以創建此 project。

## Usage
* slurm 預設安裝版本為 24.05.2，若想修改，請 `-e "slurm_version=xxx"`

## Notice
* 環境為 Debian 12
* compute node 的 NFS 會掛 login node export 出來的
* 關於 inventory 的部分，將 DB 跟 slurmdbd 分隔開來，因為 slurmdbd 可以有多台(backup)，但 DB 目前沒處理
* munge 部分，需要修改`roles/munge/defaults/main.yml`來改變可 munge 一開始 create key 的 node (預設為 `hpc051`)
* NFS 部分，目前僅有一個 server，需要修改`roles/nfs/defaults/main.yml`來改變可 access 的 subnets (預設為 `0.0.0.0/0`)
    * 共用的資料夾為 `/opt/spack` 及 `/home`
* **(尚未處理多個 DB 的情形)**
* 此 playbook 有處理 login node 及 slurmdbd node，但目前先將 control node 當 login node 及 slurmdbd node 用
    * DB 可用 mariadb server (default) 或 mysql server，並請記得修改`roles/slurm_config/defaults/main.yml`裡面的`slurm_db_pass`
    * 若選擇 mysql server，mysql 的 apt config .deb 檔需要自行下載，並放入`roles/slurm_install/files/`
        * 因為用 ansible 的 get_url 載 mysql 的檔案會無法正常下載，所以才改用 copy 的方式
