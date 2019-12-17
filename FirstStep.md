This tutorial shows how to do a master to master and a master to slave simple installation on ubuntu server.

# The goal
- Running a master to master bucardo install between to dbs inside the same bucardo server on ubuntu
![](https://github.com/Gabriel-Desharnais/bucardoTutorials/blob/master/Untitled%20Diagram.png)
# Requirement
- Fresh install of ubuntu server 18.04.3
- Network connection

# Install dependencies
``` bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install postgresql postgresql-contrib
sudo apt-get install docker.io
sudo usermod -aG docker $USER
sudo apt-get install postgresql-plperl
sudo apt-get install perl
sudo apt-get install libdbi-perl
sudo apt-get install libpq-dev
sudo apt-get install libdbix-safe-perl
sudo apt-get install bucardo
sudo mkdir /var/run/bucardo
sudo mkdir /var/log/bucardo
```

# Modification to access file

``` bash
sudo vim /etc/postgresql/10/main/postgresql.conf # Modify to listen_addresses = '*'
# change to
# "local" is for Unix domain socket connections only
local   all             all                                     trust
# IPv4 local connections:
host    all             all             0.0.0.0/0               trust
sudo vim /etc/postgresql/10/main/pg_hba.conf
sudo /etc/init.d/postgresql restart
```
# bucardo setup

``` bash
exit
```

login again

``` bash
sudo bucardo install
sudo /etc/init.d/postgresql restart
```


``` bash
docker run  --name "pgadmin4" -e "PGADMIN_DEFAULT_EMAIL=gab@gab.gab" -e "PGADMIN_DEFAULT_PASSWORD=gab" -p 5050:80 -d dpage/pgadmin4
```

# sync dbs
- Create a bd with the tables you want
- Replicate it on a new name
- fill the og bd with data

``` bash
sudo -u bucardo bucardo add database dbtest1 dbname=test1
sudo -u bucardo bucardo add database dbtest2 dbname=test2
sudo -u bucardo bucardo add all tables db=dbtest1 --herd=dbtest1_dbtest2
sudo -u bucardo bucardo add all tables db=dbtest2 --herd=dbtest2_dbtest1
sudo -u bucardo bucardo add sync sync_dbpmis1 relgroup=dbtest1_dbtest2 dbs=dbtest1,dbtest2 onetimecopy=2
sudo -u bucardo bucardo add sync sync_dbpmis2 relgroup=dbtest2_dbtest1 dbs=dbtest2,dbtest1
sudo bucardo start
sudo -u bucardo bucardo status
```

- omg it's working
