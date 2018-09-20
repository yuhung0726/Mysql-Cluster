環境：AWS EC2  
機器：Amazon Linux
### 系統需求
|   |RAM|HDD|
|:---|:---:|:---:|
|官方QA|64G|512G|
|維基|1G|3G|
|AWS.mgm|4G|8G|
|AWS.ndb+mysql|2G|8G|

### 安裝步驟
1. 下載  
```
wget https://dev.mysql.com/get/Downloads/MySQL-Cluster-7.5/mysql-cluster-gpl-7.5.11-linux-glibc2.12-x86_64.tar.gz  
```
2. 解壓  
```
tar zxvf mysql-cluster-gpl-7.5.11-linux-glibc2.12-x86_64.tar.gz
```
3. 移動兼改名  
```
mv mysql-cluster-gpl-7.5.11-linux-glibc2.12-x86_64/ /usr/local/mysql
```
4. 裡面建一個mysql-cluster資料夾  
```
mkdir /usr/local/mysql/mysql-cluster
```
5. 建立節點ip別名(換自己的私有ip)  
```
vim /etc/hosts
```
> 10.0.34.172     mysqlnodem  
> 10.0.32.18      mysqlnode1  
> 10.0.33.230     mysqlnode2  
6. 建立(new)管理節點的config  
```
vim /usr/local/mysql/mysql-cluster/config.ini  
```
> [NDBD DEFAULT]  
> NoOfReplicas=2  
> DataMemory=128M  
> IndexMemory=128M  
> SharedGlobalMemory=128M  
> DiskPageBufferMemory=128M  
>   
> [MYSQLD DEFAULT]  
>   
> [NDB_MGMD DEFAULT]  
> DataDir=/usr/local/mysql/mysql-cluster  
>   
> [TCP DEFAULT]  
>   
> [NDB_MGMD]  
> NodeId=1  
> HostName=mysqlnodem  
>   
> [MYSQLD]  
> NodeId=2  
> HostName=mysqlnode1  
>   
> [MYSQLD]  
> NodeId=3  
> HostName=mysqlnode2  
>   
> [NDBD]  
> NodeId=4  
> HostName=mysqlnode1  
> DataDir=/usr/local/mysql/mysql-cluster  
> 
> [NDBD]  
> NodeId=5  
> HostName=mysqlnode2  
> DataDir=/usr/local/mysql/mysql-cluster  
7. 建立寫mysql節點&資料節點的cnf
```
vim /etc/my.cnf  
```
> [mysqld]  
> \# enable cluster service  
> ndbcluster  
>   
> \# with connect manage node  
> ndb-connectstring=10.0.34.172:1186 \#管理節點IP  
>   
> \# defaulte database engine  
> default-storage-engine=NDBCLUSTER  
>   
> \# setting character  
> skip-character-set-client-handshake  
> character-set-server = utf8mb4  
> collation-server = utf8mb4_general_ci  
> init-connect = SET NAMES utf8mb4  
>   
> basedir=/usr/local/mysql  
> datadir=/usr/local/mysql/data  
> socket=/usr/local/mysql/data/mysql.sock  
> user=mysql  
> \# Disabling symbolic-links is recommended to prevent assorted security risks  
> symbolic-links=0  
> explicit_defaults_for_timestamp=true  
> innodb_buffer_pool_size=64M  
>   
> [mysql_cluster]  
> \# cluster management node  
> ndb-connectstring=10.0.34.172:1186 \#管理節點IP  
>   
> [mysqld_safe]  
> log-error=/var/log/mysqld.log  
> pid-file=/var/run/mysqld/mysqld.pid  
>   
> [client]  
> socket=/usr/local/mysql/data/mysql.sock  
8. 啟動管理節點  
```
cd /usr/local/mysql/bin  
./ndb_mgmd -v -f /usr/local/mysql/mysql-cluster/config.ini  
```
- 需重新載入可使用  
```
./ndb_mgmd -v -f /usr/local/mysql/mysql-cluster/config.ini --reload
```
- 檢查節點 
```
./ndb_mgm -e show  
```
9. 啟動ndb節點  
- 不需初始化則不用加 -initial 
```
./ndbd --defaults-file=/etc/my.cnf -v -initial 
```
- 重新初始化需砍掉data資料夾  
```
rm -rf /usr/local/mysql/data
```
10. 啟動mysql節點  
- 初始化(需記住暫時密碼)(已有資料可略過此步驟)  
```
./mysqld  --initialize  
```
- 改權限(不改的話一般user無法使用)
```
adduser mysql  
chown -R mysql:mysql /usr/local/mysql  
chmod -R 755 /usr/local/mysql  
```
- 啟動mysql  
```
./mysqld &  
```
11. 登入mysql & 修改密碼  
```
./mysql -uroot -p  
```
> set password=password('密碼')
- 刷新權限
> flush privileges; 
