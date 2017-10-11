============= Ansible playbook,roles for WEB01,APP01,DB01 ========================

Ansible Roles: 

  a)Opencms -Istall and configure  Opencms app on App instance deploying on tomcat server

  b)webserver -Install and configure MySQL on DB instance.

  c)tomcat -Install and configure Tomcat on App instance

  d)mysqldb - install and configure MySQL on DB instance.

Route 53 :

  a)create a recode set for www with EIP of WEB01 (this should be configured as per DNS redirecting URL )

  b)accessign the app www.opencmsapp.tk

Ansible Control server:
-----------------------------------------------------------------------------------------------

1)Installation of git and check git version

ex:
[root@ip-10-0-0-250 ec2-user]# yum install -y git
[root@ip-10-0-0-250 ec2-user]# git --version
git version 1.8.3.1

2)Install and configure latest version of Ansible on controle instance with  RHEL

#yum install -y ansible
https://developers.redhat.com/blog/2016/09/02/how-to-install-and-configure-ansible-on-rhel/

[root@ip-10-0-0-250 ec2-user]# rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
[root@ip-10-0-0-250 ec2-user]# yum install -y ansible
[root@ip-10-0-0-250 ec2-user]# ansible --version
ansible 2.3.1.0

3)must be install telnet on all the machines to test the connectivity 

Project installation work :
------------------------------------------------------------------------------------------------

1)First Clone the aws_opencms repo, and start up Ansible:

https://github.com/rajaprojects/aws_ansible_templates.git

2)On Ansible Master server :

[root@ip-10-0-0-250 ansible]# useradd -d /home/ansadm -m ansadm
[root@ip-10-0-0-250 ansible]# passwd ansadm
Changing password for user ansadm.
New password:
BAD PASSWORD: The password is shorter than 8 characters
Retype new password:
passwd: all authentication tokens updated successfully.
[root@ip-10-0-0-250 ansible]# passwd -x -1 ansadm
Adjusting aging data for user ansadm.
passwd: Success
[root@ip-10-0-0-250 ansible]#

3)installed ansible in control server and generated ssh-key gen in control server under "ansadm" user home 
and later copied the id_rsa.pub content into all nodes server by creating folder under /home/ansadm/.ssh/

[root@ip-10-0-0-250 ansadm]# su ansadm
[ansadm@ip-10-0-0-250 ~]$ pwd
/home/ansadm
[ansadm@ip-10-0-0-250 ~]$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ansadm/.ssh/id_rsa):
Created directory '/home/ansadm/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ansadm/.ssh/id_rsa.
Your public key has been saved in /home/ansadm/.ssh/id_rsa.pub.
#

4)Please make sure that give the appropriate permissions and verify the connecting from ansible server using SSH agent

vi authorized_keys 
#chomod 700 .ssh 
#chmod 600 authorized_keys


5)add the hosts names groups in /etc/ansible in "hosts" but make sure "/etc/ansible" mount point should mapped to user:ansadm and groupid:ansadm"
for WEWB01 ,APP01 and DB01 instances 

[webserver]
[appserver]
[dbserver]

6)On App01 server :

[root@ip-10-0-2-122 ec2-user]# useradd -d /home/ansadm -m ansadm
[root@ip-10-0-2-122 ec2-user]# passwd ansadm
Changing password for user ansadm.
New password:
BAD PASSWORD: The password is shorter than 8 characters
Retype new password:
passwd: all authentication tokens updated successfully.
[root@ip-10-0-2-122 ec2-user]# passwd -x -1 ansadm
Adjusting aging data for user ansadm.
passwd: Success
[root@ip-10-0-2-122 ec2-user]#
[ansadm@ip-10-0-2-122 ~]$ mkdir .ssh

add id_ras.pub file of Ansible master server into the below file:

[ansadm@ip-10-0-2-122 ~]$ vi .ssh/authorized_keys

then change the permissions
[ansadm@ip-10-0-2-122 ~]$ chmod 700 .ssh/
[ansadm@ip-10-0-2-122 .ssh]$ chmod 600 authorized_keys

Then do ssh from ansible control server check all servers able to connect without password

[ansadm@ip-10-0-0-250 ~]$ ssh ansadm@ip-10-0-2-122.ec2.internal

7)run ping module to check connectivity with passowrdless -SSH agen from ansible control server

[ansadm@ip-10-0-0-250 ansible]$ ansible appserver -m ping
10.0.2.122 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
ip-10-0-2-122.ec2.internal | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

8) we should add ansadm user in to /etc/sudoers then only package yum installtion should success into all the nodes 

##ansible admin user
ansadm ALL=NOPASSWD: ALL

[root@ip-10-0-0-250 ansible]# vi /etc/sudoers


================== WEB01 installation ====================
1)

Through Ansible role 
i changed in default server web01 
[ansadm@ip-10-0-0-250 defaults]$ ls -lrt
total 4
-rw-rw-r--. 1 ansadm ansadm 1774 Sep 24 15:58 main.yml
[ansadm@ip-10-0-0-250 defaults]$ vi main.yml
[ansadm@ip-10-0-0-250 defaults]$

2) i changed 
[ansadm@ip-10-0-0-250 templates]$ vi vhosts.conf.j2

for this error fatal: [ip-10-0-0-30.ec2.internal]: FAILED! => {"changed": false, "failed": true, "msg": "AnsibleUndefinedVariable: 'appserver' is undefined"}
so i added the below tomcat server name into templated/vhosts.conf.j2
ip-10-0-2-194.ec2.internal

i disabled selinux 
https://www.tecmint.com/disable-selinux-temporarily-permanently-in-centos-rhel-fedora/
[root@ip-10-0-0-30 targeted]# echo 0 > /etc//selinux/enforce
[root@ip-10-0-0-30 targeted]# setenforce 0
[root@ip-10-0-0-30 targeted]#


===================== 2) tomcat and app ====================

TASK [opencmsapp : Configure Tomcat server env file] ****************************************************************************************************************************
changed: [ip-10-0-2-194.ec2.internal]

TASK [opencmsapp : Configure Auto Setup Script config] **************************************************************************************************************************
fatal: [ip-10-0-2-194.ec2.internal]: FAILED! => {"changed": false, "failed": true, "msg": "AnsibleUndefinedVariable: 'db_user' is undefined"}
        to retry, use: --limit @/etc/ansible/tomcat_app.retry

PLAY RECAP **********************************************************************************************************************************************************************
ip-10-0-2-194.ec2.internal : ok=20   changed=16   unreachable=0    failed=1

[ansadm@ip-10-0-0-250 ansible]$ ansible-playbook tomcat_app.yml -i hosts -l appserver


=============== 3) DB server ========================

1)To check MySql verion:

Installign Mysql using mysqldb role from ansible control server 

[root@ip-10-0-26-9 ec2-user]# mysqladmin version
mysqladmin  Ver 8.42 Distrib 5.6.37, for Linux on x86_64
Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Server version          5.6.37
Protocol version        10
Connection              Localhost via UNIX socket
UNIX socket             /var/run/mysqld/mysqld.sock
Uptime:                 29 min 33 sec

Threads: 1  Questions: 4  Slow queries: 0  Opens: 67  Flush tables: 1  Open tables: 60  Queries per second avg: 0.002
[root@ip-10-0-26-9 ec2-user]#


or 

[root@ip-10-0-26-9 ec2-user]# mysql -version

2)How to start ,stop,restart and status of MySql ?

[root@ip-10-0-26-9 mysql]# service mysql start
[root@ip-10-0-26-9 mysql]# service mysql stop
[root@ip-10-0-26-9 mysql]# service mysql restart

or 
#systemctl enable mysqld.service 
#systemctl stop mysqld.service 
#systemctl start mysqld.service 
#systemctl restart mysqld.service 
#systemctl status mysqld.service 



https://serverfault.com/questions/520751/installing-mysql5-6-12-in-ubuntu-12-10-error-etc-mysql-conf-d-is-missing

[root@ip-10-0-26-9 mysql]# service mysql start


issue :
[ansadm@ip-10-0-26-9 mysql]$ mysql -u root -p
mysql: Can't read dir of '/etc/mysql/conf.d/' (Errcode: 2 - No such file or directory)
Fatal error in defaults handling. Program aborted
[ansadm@ip-10-0-26-9 mysql]$

solution :
https://serverfault.com/questions/520751/installing-mysql5-6-12-in-ubuntu-12-10-error-etc-mysql-conf-d-is-missing


[root@ip-10-0-26-9 mysql]# mkdir /etc/mysql/conf.d
[root@ip-10-0-26-9 mysql]# mysql -u root -p
Enter password:
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (2)
[root@ip-10-0-26-9 mysql]#

and just stop and start 

[root@ip-10-0-26-9 mysql]# service mysqld stop
[root@ip-10-0-26-9 mysql]# service mysqld start
[root@ip-10-0-26-9 mysql]# service mysqld status
Redirecting to /bin/systemctl status mysqld.service
● mysqld.service - MySQL Community Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2017-10-10 12:25:50 UTC; 3s ago
  Process: 9114 ExecStartPost=/usr/bin/mysql-systemd-start post (code=exited, status=0/SUCCESS)
  Process: 9099 ExecStartPre=/usr/bin/mysql-systemd-start pre (code=exited, status=0/SUCCESS)
 Main PID: 9113 (mysqld_safe)
   CGroup: /system.slice/mysqld.service
           ├─9113 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
           └─9538 /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log/mysqld.log --pid-file=/var/run/mysqld/mys...

Oct 10 12:25:49 ip-10-0-26-9.ec2.internal systemd[1]: Starting MySQL Community Server...
Oct 10 12:25:49 ip-10-0-26-9.ec2.internal mysqld_safe[9113]: 171010 12:25:49 mysqld_safe Logging to '/var/log/mysqld.log'.
Oct 10 12:25:49 ip-10-0-26-9.ec2.internal mysqld_safe[9113]: 171010 12:25:49 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
Oct 10 12:25:50 ip-10-0-26-9.ec2.internal systemd[1]: Started MySQL Community Server.
[root@ip-10-0-26-9 mysql]#


2nd time connecting :

ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
[ansadm@ip-10-0-26-9 mysql]$ mysql -u root -p

Solution : jsut run this and try 
mysql -u root -S /var/run/mysqld/mysqld.sock
https://serverfault.com/questions/516934/after-installing-mysql-5-6-the-server-cant-start


[ansadm@ip-10-0-26-9 conf.d]$ exit
exit
[root@ip-10-0-26-9 mysql]# mysql -u root -S /var/run/mysqld/mysqld.sock
or 
[root@ip-10-0-26-9 mysql]# mysql -h ip-10-0-26-9.ec2.internal -u root

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.6.37 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>


3rd)

created table and insert values :

https://www.cyberciti.biz/faq/howto-linux-unix-creating-database-and-table/

[root@ip-10-0-26-9 mysql]# mysql -u root -S /var/run/mysqld/mysqld.sock
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 5.6.37 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CREATE DATABASE books;
Query OK, 1 row affected (0.00 sec)

mysql> USE books;
Database changed
mysql> CREATE TABLE authors (id INT, name VARCHAR(20), email VARCHAR(20));
Query OK, 0 rows affected (0.02 sec)

mysql> SHOW TABLES;
+-----------------+
| Tables_in_books |
+-----------------+
| authors         |
+-----------------+
1 row in set (0.00 sec)

mysql> INSERT INTO authors (id,name,email) VALUES(1,"raja","raja@abc.com");
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO authors (id,name,email) VALUES(2,"Priya","p@gmail.com");
Query OK, 1 row affected (0.01 sec)

mysql> INSERT INTO authors (id,name,email) VALUES(3,"Tiger","ten@yahoo.com");
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM authors;
+------+-------+---------------+
| id   | name  | email         |
+------+-------+---------------+
|    1 | raja  | raja@abc.com  |
|    2 | Priya | p@gmail.com   |
|    3 | Tiger | ten@yahoo.com |
+------+-------+---------------+
3 rows in set (0.00 sec)

mysql>


3)Inserting the mysql use and password :

Followed this link :

https://www.liquidweb.com/kb/change-a-password-for-mysql-on-linux-via-command-line/

update user set password=password where User='root';

mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> update user set password=password where User='root';
Query OK, 0 rows affected (0.01 sec)
Rows matched: 4  Changed: 0  Warnings: 0

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql>


https://www.liquidweb.com/kb/create-a-mysql-user-on-linux-via-command-line/

4)
use mysql:
so here using mysql databse as table "user"
https://stackoverflow.com/questions/1526688/get-table-column-names-in-mysql


insert into user values ('localhost', 'opencmsuser', password('XXXXX'),\ 
   'N','N','N','N','N','N','N','N','N','N','N','N','N','N'); 
------------------ added into user ----------------------------------
Correct :
insert into user values ('localhost', 'opencmsuser', 'password', 'N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N', 'N','N','N','N','N','', 'NULL', 'NULL', 'NULL','0','0','0','0', 'mysql_native_password','NULL','N');

insert into user values ('ip-10-0-26-9.ec2.internal', 'opencmsuser', 'password', 'N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N', 'N','N','N','N','N','', 'NULL', 'NULL', 'NULL','0','0','0','0', 'mysql_native_password','NULL','N');

flush privileges;
------------------------------------------------------
   
insert ignore into user values ('localhost', 'root', password('password'), 'N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N', 'N','N','N','N','N','', 'NULL', 'NULL', 'NULL','0','0','0','0', 'mysql_native_password','NULL','N'); ON DUPLICATE KEY UPDATE host=localhost and user=root'

mysql> insert into user values ('ip-10-0-26-9.ec2.internal', 'root', 'password', 'N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N', 'N','N','N','N','N','', 'NULL', 'NULL', 'NULL','0','0','0','0', 'mysql_native_password','NULL','N');
ERROR 1062 (23000): Duplicate entry 'localhost-root' for key 'PRIMARY'
mysql>



insert into user values ('localhost', 'root', password('password'), 'Y','Y','Y','','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y');

list out column names :
SELECT COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_SCHEMA = 'mysql' AND TABLE_NAME = 'user';
or 

SHOW COLUMNS FROM user;


insert into user values ('localhost', 'root', password('password'),\ 
   'N','N','N','N','N','N','N','N','N','N','N','N','N','N');
   
insert into user values ('ip-10-0-26-9.ec2.internal', 'root', password('password'), 'N','N','N','N','N','N','N','N','N','N','N','N','N','N'); 
 

SHOW COLUMNS FROM db;


------------------ added into user ----------------------------------
Correct :
insert into user values ('localhost', 'opencmsuser', 'password', 'N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N', 'N','N','N','N','N','', 'NULL', 'NULL', 'NULL','0','0','0','0', 'mysql_native_password','NULL','N');

insert into user values ('ip-10-0-26-9.ec2.internal', 'opencmsuser', 'password', 'N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N', 'N','N','N','N','N','', 'NULL', 'NULL', 'NULL','0','0','0','0', 'mysql_native_password','NULL','N');
flush privileges;

------------------ using db table inserted ---------------------------
insert into db values ('localhost', 'test', 'root', 'Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y'); 
insert into db values ('ip-10-0-26-9.ec2.internal', 'test', 'root', 'Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y'); 
flush privileges;


insert into db values ('localhost', 'opencms', 'opencmsuser', 'Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y'); 
insert into db values ('ip-10-0-26-9.ec2.internal', 'opencms', 'opencmsuser', 'Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y'); 
flush privileges;



use mysql; 
5) if want to know more Mysql commands 

https://gist.github.com/hofmannsven/9164408



------------- to test the opscms url -------------

http://34.230.6.255:8080/opencms/opencms/system/login/

http://localhost:8080/opencms/opencms/system/login/.

http://ip-10-0-2-194.ec2.internal:8080/opencms/setup/

http://34.230.6.255:8080/opencms/setup/

Finally URL through accessing Route 53 :
www.opencmsapp.tk
