# Install Metacat2.19 on Unbuntu20.04/22.04

## tested of the final version of metacat 2.x installation
1.Metacat: version 2.19.0


### Note:metacat 2.19.0 released in Apr 2024 and will be the final version of Metacat 2.x. I tested it under Ubuntu 20.04/22.04. There are several issues when I did. I consulted KNB. But only received partial answers. The useful one is to comfirm metacat only stick on Java 8 no matter what Linux distribution used. In addition, Tomcat8 is preferred.

### If you use apt to install java and tomcat, you need to make sure that Java, Tomcat, and Postgresql version bound in Ubuntu default. Using sudo apt-cache search to check and select the right version.

### metacat 2.19.0 prefers openjdk-8-jdk, tomcat8, PostgreSQL 9.6

### this test found that ubuntu 20.04/22.04 can not use apt to install tomcat8.5 and postgresql9.5. You need manually install tomcat8.5 and create tomcat.service. you also need to add postgresql public key to install postgresql 9.6

### check apt bound these software version
sudo apt-cache search tomcat
sudo apt-cache search jdk
sudo apt-cache search postgresql


# A folder called install has been created under /home/chin directory. All needed files are in the folder. Use your own home directory to store installation files. Then change to your home directiory that all the source can be run.

## 1. install java
sudo apt install openjdk-8-jdk

### if you have other java versions, choose the one you need to change.
sudo update-alternatives --config java

### java version test:
java -version

### JAVA_HOME configuration: Some tools require JAVA_HOME variable. You can set JAVA_HOME in Ubuntu so simple: Edit the file .bashrc under your home directory and add the following lines: (if .bashrc is hidden, use ls -al to show)

### editing .bashrc of user "chin"
gedit /home/chin/.bashrc

### add the follow two lines to the end of .bashrc
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH

### restart .bashrc without logout
source ~/.bashrc

## 2. Install Tomcat8.5
### create tomcat group and user
sudo groupadd tomcat
sudo useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat

### download tomcat8.5.9 from https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.9/bin/ to /home/chin/install, then extract the .tar.gz with tomcat folder
sudo mkdir /opt/tomcat
cd /opt/tomcat
sudo tar xzvf /home/chin/install/apache-tomcat-8.5.9.tar.gz -C /opt/tomcat --strip-components=1
sudo chown -R tomcat:tomcat /opt/tomcat

### the chmod is just for installation.
sudo chmod -R g+r /opt/tomcat/conf 

### editing tomcat.service
sudo gedit /etc/systemd/system/tomcat.service

### the content of tomcat.service as following (Ubuntu 20.04/22.04 using systemd which is different from init. google to find out the explaination of differences between these two)

__________________________________________________

[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64/
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_Home=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment="CATALINA_HOME=/opt/tomcat"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]

WantedBy=multi-user.target
______________________________________________________________

### set tomcat to automatically start at boot
sudo systemctl enable tomcat

### load tomcat 
sudo systemctl daemon-reload

### release firewall
sudo ufw allow 8080

### restart tomcat
sudo systemctl restart tomcat or sudo service tomcat restart

### check if AJP 1.3 is open. You need go to the config file which locates in /opt/tomcat/conf/server.xml. Find out prot 8009 and if it is marked you need uncomment it.
sudo gedit /opt/tomcat/conf/server.xml

### search prot 8009 and unmarked the connector port as following
<!-- Define an AJP 1.3 Connector on port 8009 -->
<!--(take this line out, if marked)
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
-->(take this line out)

## 3. install Apache2 and Tomcat8.5 connector

### install apache2 first and then install connector
sudo apt-get install apache2
sudo apt-get install libapache2-mod-jk

### this module will be located in /etc/apache2/mods-available).start the mod-jk (if it is not enabled)
sudo a2enmod jk

## 4. Install LDAP 

### install the system
sudo apt-get install slapd ldap-utils

### during installation you will be asked to set up administrator's password

### configure the system first you need load the three schemas (this is optional)
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/ldap/schema/cosine.ldif
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/ldap/schema/nis.ldif
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/ldap/schema/inetorgperson.ldif

### check install of LDAP
sudo slapcat

### create your DNS by manual operate. When your LDAP crash, you can recreate the system using this command.
sudo dpkg-reconfigure slapd

#### answer the following question to set up base. the steps 3/4/5 just expamles, you need change to your own
1.Omit OpenLDAP server configuration?--NO
2.DNS domain name: --ecoinformatics.org
3.Organization name:--tari
4.Administrator password:firelab
5.Confirm password:--firelab
6.Database backend to use:--HDB	(ubuntu20.04/22.04 does not have)
7.Do you want the database to be removed when slapd is purged?--yes
8.Move old database?--yes
9.Allow ldapv2 protocol?--no (ubuntu20.04/22.04 does not have)

### the above operate will allow you to create your base as dc=ecoinformatics,dc=org
and has a cn=admin with password "niu"
you can check this with sudo slapcat and you will find the following

sudo slapcat

----------------------------------------------------------------------------------------
dn: dc=ecoinformatics,dc=org
objectClass: top
objectClass: dcObject
objectClass: organization
o: niu
dc: ecoinformatics
structuralObjectClass: organihttp://www.devsniper.com/ubuntu-12-04-install-sun-jdk-6-7/zation
entryUUID: 1d6b1a5e-83c1-1032-8a0a-49168633f472
creatorsName: cn=admin,dc=ecoinformatics,dc=org
createTimestamp: 20130718064335Z
entryCSN: 20130718064335.601780Z#000000#000#000000
modifiersName: cn=admin,dc=ecoinformatics,dc=org
modifyTimestamp: 20130718064335Z

dn: cn=admin,dc=ecoinformatics,dc=org
objectClass: simpleSecurityObject
objectClass: organizationalRole
cn: admin
description: LDAP administrator
userPassword:: e1NTSEF9UUdkOUZSQVdTazNmWTcxZys2S1U4SURTaGxmMkI2K0g=
structuralObjectClass: organizationalRole
entryUUID: 1d6d2ac4-83c1-1032-8a0b-49168633f472
creatorsName: cn=admin,dc=ecoinformatics,dc=org
createTimestamp: 20130718064335Z
entryCSN: 20130718064335.615312Z#000000#000#000000
modifiersName: cn=admin,dc=ecoinformatics,dc=org
modifyTimestamp: 20130718064335Z
-----------------------------------------------------------------------------------------------

### create your layer such as you want have o=TARI (stand for Taiwan Agriculture Research Institute). if you have other organizations, you can change to other o=xxxx)
### notice: the addorg.ldif and adduser.ldif just examples to TARI

sudo ldapadd -x -D cn=admin,dc=ecoinformatics,dc=org -W -f /home/chin/install/addorg.ldif

### content of the file o=tfri.ldif as following;
---------------------------------------------
dn: o=TARI,dc=ecoinformatics,dc=org
o: TARI
objectClass: organization
objectClass: top
---------------------------------------------

### add users you want (this adduser.ldif is for o=TARI.You can change to what you like)

sudo ldapadd -x -D cn=admin,dc=ecoinformatics,dc=org -W -f /home/chin/install/adduser.ldif

### content of adduser.ldif as following;
----------------------------------------------
dn: uid=chin,o=TARI,dc=ecoinformatics,dc=org
cn: chauchin 
sn: Lin
uid: chin
userPassword: firelab
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: uidObject
objectClass: top
----------------------------------------------

### instead of using LDAP commands to create child layers, you can use web page to do it by using phpldapadmin.

### first you need install phpldapmin
sudo apt-get install phpldapadmin

### then modify configure file at /etc/phpldapadmin/config.php
sudo nano /etc/phpldapadmin/config.php 

### modidfy the following lines (line number may be different);
----------------------------------------------------------------------------------------
in line 293 $servers->setValue('server','host','127.0.0.1');
in line 300 $servers->setValue('server','base',array('dc=ecoinformatics,dc=org'));
in line 318 $servers->setValue('login','bind_id','session');
in line 326 $servers->setValue('login','bind_id','cn=admin,dc=ecoinformatics,dc=org');
-----------------------------------------------------------------------------------------

### access LDAP server by http://localhost/phpldapadmin

## 5. Install Postgresql

### install postgresql9.6
### Metcat stick on postgresql 9.x. But using apt install postgresql will install posgresql 14 on Ubuntu 20.04/22.04. Therefore, you need change to 9.6 version by the following steps

wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" | sudo tee /etc/apt/sources.list.d/postgresql-pgdg.list > /dev/null

sudo apt-get update -y
sudo apt-get install postgresql-9.6 

### You can now start the database server using:
sudo /usr/lib/postgresql/9.6/bin/pg_ctl -D /var/lib/postgresql/9.6/main -l logfile start

### config file directory is /var/lib/postgresql/9.6/main/pg_hba.conf

### make Postgresql accessible.By default Postgresql will create a user name 'postgres' in Ubuntu. But you don? know the password. Therefore, you need make user 'postgres' a new password. Then you can use pgadmin3 to manage your database. notice the 'firelab' is just an example.
sudo su postgres -c psql template1
ALTER USER postgres WITH PASSWORD 'firelab';
\q

### you also can use command scripts to create database
sudo su - postgres
createdb metacat

### the database metacat is owned by postgres, you can change the owner
psql metacat
CREATE USER metacat with UNENCRYPTED PASSWORD 'firelab';
\q
exit


## 6.Install solr 

### download solr-8.8.2.tgz to /home/chin/install from https://archive.apache.org/dist/lucene/solr/8.8.2/
cd /home/chin/install
tar xzf solr-8.8.2.tgz solr-8.8.2/bin/install_solr_service.sh --strip-components=2

### Install Solr as the root user:
sudo bash ./install_solr_service.sh solr-8.8.2.tgz

### Ensure the Solr defaults file is group writable:
sudo chmod g+w /etc/default/solr.in.sh

### Check if the Solr service is running:
sudo service solr status

### Make sure the firewall is running and the default port 8983 is not exposed externally (assume you are using ufw):
sudo ufw status

### Add New Allowed Solr Paths when using systemd in Toomcat.
sudo gedit /etc/default/solr.in.sh

### Add two line in the end of the file
SOLR_OPTS="$SOLR_OPTS -Dsolr.allowPaths=*"
SOLR_JAVA_MEM="-Xms2g -Xmx2g"

### Tomcat and Solr User Management.The interaction of the Tomcat and Solr services can cause the file permission issues. Add the tomcat8 **user to the solr group and the solr user to tomcat8 group to fix the problem:

sudo usermod -a -G solr tomcat
sudo usermod -a -G tomcat solr

### Restart the Solr server to make the new group setting effective (Important)
sudo service solr stop
sudo service solr start

### Check that the tomcat8 user and solr user are members of the appropriate groups with:
sudo groups tomcat
sudo groups solr

### create metacat use core on solr engine
sudo su - solr -c "/opt/solr/bin/solr create -c metacat-index -n data_driven_schema_configs"

### summary of stop and start services 

sudo service restart
sudo service tomcat restart
sudo service solr start
sudo systemctl restart postgresql.service

## 7. Install Metacat
### before you install Metacat, you need make Metacat convext(the folder name of metacat files) in Apache to let http can access Tomcat8. if you don't have jk.conf and workers.properties. you need to edit it first. you can just use these two files I provide.

sudo cp /home/chin/install/jk.conf /etc/apache2/mods-available/
sudo cp /home/chin/install/workers.properties /etc/apache2/workers.properties

### The content of jk.conf
---------------------------------------------------------
JkWorkersFile   /etc/apache2/workers.properties
JkLogFile       /var/log/apache2/mod_jk.log
JkShmFile       /var/log/apache2/mod_jk.shm
JkLogLevel      info
---------------------------------------------------------

### The content of workers.properties
---------------------------------------------------------
workers.tomcat_home=/opt/tomcat/
workers.java_home=/usr/lib/jvm/java-1.8.0-openjdk-amd64/
worker.list=ajp13
worker.ajp13.port=8009
worker.ajp13.host=localhost
worker.ajp13.type=ajp13
worker.ajp13.lbfactor=1
worker.loadbalancer.type=lb
----------------------------------------------------------

### Set up a virtual host

sudo gedit /etc/apache2/sites-available/metacat.conf

### The following is the content of httpd.conf (you can add JkMonut context by your own need). Notice ServerName can be changed to fit your own server.

---------------------------------------------------------
<VirtualHost *:80>
        DocumentRoot /var/www/html
        ServerName localhost
        JkMount /metacat-index ajp13
        JkMount /metacat-index/* ajp13
        JkMount /metacat ajp13
        JkMount /metacat/* ajp13
        JkMount /metacatui ajp13
        JkMount /metacatui/* ajp13   
</VirtualHost>
---------------------------------------------------------

### enable metacat.conf and restart apache2 (this step is very important to connect Apache2 and Tomacat8) and restart apache2
sudo a2ensite metacat.conf
sudo systemctl reload apache2

### download metcat2.19.0 from knb and extract the file from download
sudo cp metacat.war metacatui.war metacat-index /opt/tomcat/webapps

### Metacat needs a folder to store files such as raw dataset,logs. So you need create a folder called metacat under /var. During installation this folder need to be able to write (writable). you can change the right of folder back after completing installation.
### metacat 2.19.0 using solr for search engine. the solr core set in solr-home when configureing metacat. It is better to set /var/metacat owner with tomcat

sudo mkdir /var/metacat
sudo chown -R tomcat:tomcat /var/metacat

### chmod of 777 is just for installation. you need to change after installation with proper right.
sudo chmod -R 777 /var/metacat

## install metacat by copying metacat.war, metacat-index.war, and metacatui.war to tomcat
sudo cp /home/chin/install/*.war /opt/tomcat/webapps
sudo service tomcat restart

### type localhost/metacat or localhost:8080/metacat then you can follow the instruction to configure metcat (notice localhost is the local machine. if you gave ServerName in apache2 you need use that for example, I use 10.39.11.20 which is a fix url assigned, therefore I need type http://10.39.11.20/metacat to start insatll metacat)

http://localhost:8080/metacat

### follow the configuration steps to go on.you should see configuration metacat interface. Just follow the instruction, you should be able to install Metacat and run it.

### if you completed the installation without any problem, you can restart tomcat and test if the installation is success. There is a trick issue you might encounter. When you access metacatui, it shows the error message "The config file or loader.js file failed to load. Check the appConfigPath in index.html and the 'root' attribute in AppConfig."

### solution 
1. sudo nano /opt/tomcat/webapps/metacatui/index.html
var appConfigPath = "/config/config.js"; -> var appConfigPath = "config/config.js";

2. sudo nano /opt/tomcat/webapps/metacatui/config/config.js
MetacatUI.AppConfig = {
  //The path to the root location of MetacatUI, i.e. where index.html is
    root: "/metacatui",
  //The path to the root location of Metacat, i.e. name of the Metacat Tomcat webapp
    metacatContext: "/metacat",
  // show at title of metacatUI html page
    repositoryName: "TARI Data Catalog",
    defaultSearchFilters: ["all", "attribute", "abstract", "description", "documents", "creator", "dataYear", "pubYear", "id", "taxon", "spatial"]
}


