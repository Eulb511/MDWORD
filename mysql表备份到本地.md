### 在 Kubernetes（K8s）中，要从 MySQL 容器中提取文件，可以使用以下几种常见的方法：

mysql服务器->k8s集群空间->本地

#### 1.  mysql服务器->k8s集群空间

从mysql中备份表文件,会保存在当前文件夹中 ls

```
mysqldump -uroot -pZeuS@Middleware01 -h127.0.0.1 -P3306  middleware_platform operation_audit --set-gtid-pu
rged=OFF --result-file=/zeus-2024_01_08_11_22_04-dump.sql
```

再将文件移动到k8s集群环境中

方法1  kubectl cp

```
kubectl cp default/mysql-pod:/var/lib/mysql/data/myfile.txt /tmp/myfile.txt -c mysql    #-c mysql指定容器
```

方法2。**使用 `kubectl exec` 命令和 `tar` 命令**

```
kubectl exec -it 命名空间/pod名字 -- tar cf - -C /var/lib/mysql/data . | tar xf - -C /tmp   

#-it终端打开   
#tar xf - -C /local/path: 在本地机器上使用 tar 命令解压标准输入流中的 tar 存档内容，-C 用于指定解压缩目标路径。
```

#### 2.  k8s集群空间->本地

在服务器上下载文件通常可以使用 `curl` 或 `wget` 命令。以下是这两个命令的用法示例：

使用 `curl`

```
curl [options] URL
```

这将从指定的 URL 下载文件并保存到当前工作目录。例如：

```
curl -# -O https://example.com/file.txt  #使用 -O 选项表示将文件保存为与远程文件名相同的本地文件名。-#下载速度 
```

**使用 `wget`**

```
wget URL
```

这也将从指定的 URL 下载文件并保存到当前工作目录。例如：

```
wget https://example.com/file.txt
```

如果你需要指定文件的保存路径，可以在命令后面添加保存路径：

```
wget -O /path/to/save/file.txt https://example.com/file.txt
```

**需要登入用scp**

```
scp username@remote_server:/path/to/remote/file /path/to/local/directory
```

3. ####  数据库表导入本地

```
mysql -u username -p -h hostname database_name < filename.sql
```




## 以下为你详细介绍在 MySQL 数据库中备份所有表的结构（不含数据），删除数据库，重新创建数据库并导入备份的具体步骤，这里会分别给出在本地 MySQL 和 Docker 中 MySQL 容器的操作方法。​
本地 MySQL 操作步骤​
1. 备份数据库表结构（不含数据）​
使用 mysqldump 工具备份数据库中所有表的结构，不包含数据。命令如下：​
​
mysqldump -u <用户名> -p --no-data <数据库名> > backup_structure.sql​
​
示例：​
​
mysqldump -u root -p --no-data mydatabase > backup_structure.sql​
​
执行该命令后，会提示你输入密码，输入正确密码后，会将 mydatabase 数据库的所有表结构备份到 backup_structure.sql 文件中。​
2. 删除数据库​
使用 mysql 命令行工具登录到 MySQL 服务，然后删除指定的数据库：​
​
mysql -u <用户名> -p -e "DROP DATABASE <数据库名>;"​
​
示例：​
​
mysql -u root -p -e "DROP DATABASE mydatabase;"​
​
3. 重新创建数据库​
使用 mysql 命令行工具登录到 MySQL 服务，然后创建新的数据库：​
​
mysql -u <用户名> -p -e "CREATE DATABASE <数据库名>;"​
​
示例：​
​
mysql -u root -p -e "CREATE DATABASE mydatabase;"​
​
4. 导入备份的表结构​
使用 mysql 命令将之前备份的表结构文件导入到新创建的数据库中：​
​
mysql -u <用户名> -p <数据库名> < backup_structure.sql​
​
示例：​
​
mysql -u root -p mydatabase < backup_structure.sql​
​
Docker 中 MySQL 容器操作步骤​
1. 备份数据库表结构（不含数据）​
使用 docker exec 命令在容器内执行 mysqldump 命令进行备份：​
​
docker exec -it <容器名称或 ID> mysqldump -u <用户名> -p --no-data <数据库名> > backup_structure.sql​
​
示例：​
​
docker exec -it my - mysql - container mysqldump -u root -p --no-data mydatabase > backup_structure.sql​
​
同样，执行该命令后会提示输入密码。​
2. 删除数据库​
使用 docker exec 命令在容器内执行删除数据库的 SQL 语句：​
​
docker exec -it <容器名称或 ID> mysql -u <用户名> -p -e "DROP DATABASE <数据库名>;"​
​
示例：​
​
docker exec -it my - mysql - container mysql -u root -p -e "DROP DATABASE mydatabase;"​
​
3. 重新创建数据库​
使用 docker exec 命令在容器内执行创建数据库的 SQL 语句：​
​
docker exec -it <容器名称或 ID> mysql -u <用户名> -p -e "CREATE DATABASE <数据库名>;"​
​
示例：​
​
docker exec -it my - mysql - container mysql -u root -p -e "CREATE DATABASE mydatabase;"​
​
4. 导入备份的表结构​
使用 docker exec 命令将备份文件复制到容器内，然后在容器内执行导入操作：​
​
docker cp backup_structure.sql <容器名称或 ID>:/tmp/backup_structure.sql​
docker exec -it <容器名称或 ID> mysql -u <用户名> -p <数据库名> < /tmp/backup_structure.sql​
​
示例：​
​
docker cp backup_structure.sql my - mysql - container:/tmp/backup_structure.sql​
docker exec -it my - mysql - container mysql -u root -p mydatabase < /tmp/backup_structure.sql​
​
通过以上步骤，你可以完成数据库表结构的备份、数据库的删除和重新创建，以及备份结构的导入。


