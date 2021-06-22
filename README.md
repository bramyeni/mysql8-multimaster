# MySQL 8.0 Multimaster Replication
The concept is the same as Mariadb multimaster replication which can be found here https://github.com/bramyeni/Mariadb-Multimaster
However there few differences:
1. MySQL 8.0 default authentication using sha2 or sha256, in this case i will force to use native (old style)
2. MySQL 8.0 database is initialized by default using secured password for root, this is annoying for testing purpose, therefore i enforced to use --initialize-insecure when initializing a new MySQL 8 database, it is very important to set the root password securely after that 
3. Galera replication on Mariadb uses mariabackup (by default), with MySQL 8 i found out using rsync is a lot easier and faster (by default it uses mysqldump, this is slow and again requires SSL connection for securing communication between donor and joiner nodes)

## How to build the docker image
Since there is no Galera 4 for mysql 8.0 running on alpine linux, therefore the docker image will use debian 10 (buster)

cd /opt
docker build --tag galera-mysql8 -f ./Dockerfile-galera-mysql8 /mnt

## How to Run
You can find few "docker run" samples within the Dockerfile, i will re-write those in here


Rebuild DBCONFIG and DATABASE for Single Node server
<pre>
docker run -it --name mysql8 -p 3306:3306 -v /opt/mysql:/var/lib/mysql -v /opt/mysql/conf:/etc/mysql.conf.d -e MYSQL_DBCONFIG_REBUILD=yes -e MYSQL_DATABASE_REBUILD=yes galera-mysql8

docker run -it --name mysql8 --network host -v /opt/mysql:/var/lib/mysql -v /opt/mysql/conf:/etc/mysql.conf.d -e MYSQL_DBCONFIG_REBUILD=yes -e MYSQL_DATABASE_REBUILD=yes galera-mysql8
</pre>

On Donor node (galera active)
Rebuild DBCONFIG with HOST network
<pre>
docker run -it --name mysql8  --network host -v /opt/mysql:/var/lib/mysql -v /opt/mysql/conf:/etc/mysql.conf.d -e GALERA_CLUSTER_BOOTSTRAP=yes -e MYSQL_DBCONFIG_REBUILD=yes galera-mysql8
</pre>

Run without Rebuild with HOST network
<pre>
docker run -it --name mysql8  --network host -v /opt/mysql:/var/lib/mysql -v /opt/mysql/conf:/etc/mysql.conf.d galera-mysql8
</pre>

For Joiner node:
Rebuild DBCONFIG and DATABASE with HOST network
<pre>
docker run -it --name mysql8  --network host -v /opt/mysql:/var/lib/mysql -v /opt/mysql/conf:/etc/mysql/mysql.conf.d -e MYSQL_DBCONFIG_REBUILD=yes -e MYSQL_DATABASE_REBUILD=yes -e GALERA_CLUSTER_ADDRESS="gcomm://192.168.0.178" galera-mysql8
</pre>

Run without Rebuild DBCONFIG and DATABASE with HOST network
<pre>
docker run -it --name mysql8  --network host -v /opt/mysql:/var/lib/mysql -v /opt/mysql/conf:/etc/mysql/mysql.conf.d -e GALERA_CLUSTER_ADDRESS="gcomm://192.168.0.158" galera-mysql8
</pre>
