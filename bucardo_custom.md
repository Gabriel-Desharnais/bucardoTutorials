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
sudo mkdir /var/run/bucardo
sudo mkdir /var/log/bucardo
sudo chmod -R 777 /var/run/bucardo
sudo chmod -R 777 /var/log/bucardo
wget     https://bucardo.org/downloads/Bucardo-5.5.0.tar.gz
tar xvfz Bucardo-5.5.0.tar.gz
cd Bucardo-5.5.0/
perl Makefile.PL
sudo apt install make
make
sudo make install
cd ..
rm -rf Bucardo-5.5.0
rm Bucardo-5.5.0.tar.gz
bucardo install
wget https://bucardo.org/downloads/dbix_safe.tar.gz
tar xvfz dbix_safe.tar.gz
cd DBIx-Safe-1.2.5/
perl Makefile.PL
make
sudo make install
cd ..
rm dbix_safe.tar.gz
rm -rf DBIx-Safe-1.2.5/
wget https://cpan.metacpan.org/authors/id/M/MA/MATTN/Devel-CheckLib-1.14.tar.gz
tar xvfz Devel-CheckLib-1.14.tar.gz
cd Devel-CheckLib-1.14/
perl Makefile.PL
make
sudo make install
cd ..
rm -rf Devel-CheckLib-1.14
rm Devel-CheckLib-1.14.tar.gz
sudo apt-get install gcc
wget https://cpan.metacpan.org/authors/id/T/TU/TURNSTEP/DBD-Pg-3.10.0.tar.gz
tar xvfz DBD-Pg-3.10.0.tar.gz
cd DBD-Pg-3.10.0/
perl Makefile.PL
make
sudo make install
cd ..
rm DBD-Pg-3.10.0.tar.gz
rm -rf DBD-Pg-3.10.0/
```

# Modification to access file

``` bash
#listen_addresses = 'localhost'         # what IP address(es) to listen on;
# to  listen_addresses = '*'
sudo vim /etc/postgresql/10/main/postgresql.conf
# Database administrative login by Unix domain socket
#local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
#host    all             all             127.0.0.1/32            md5
# change to
# "local" is for Unix domain socket connections only
#local   all             all                                     trust
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
