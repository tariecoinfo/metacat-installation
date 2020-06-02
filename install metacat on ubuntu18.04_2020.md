# **Install Metacat on Unbuntu18.04**

> tested of the following version of metacat and morpho
   -Metacat: version 2.13.0
   -Morpho: version 1.9.1 and 1.11.0

> I create a folder called install under home which is /home/chin. The install folder includes:
   -source: all programs needed 
   -conf: configuration files for installation
   -doc: all scripts like this guide
   -eml: exmpales using eml 1.1.0


> Installation start from here.

## Install Java
> For Metacat 2.13.0 you need JDK7 or JDK8. I used open-jdk-8

sudo apt install openjdk-8-jdk

### Check Java Installation.

java -version

### If you have serveral Java versions, you can choose default java using the following commands

sudo update-alternatives --config java

### JAVA_HOME configuration
> Some tools require JAVA_HOME or JRE_HOME variables. You can set JAVA_HOME by editing the file called .bashrc under your home directory.
> add the following lines: (if .bashrc is hidden, click in Nautilus Menu View > Show Hidden Files). You need change directory to your home directory. For example. I do this. cd /home/chin, gedit .bashrc.

export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
export JRE_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre
export PATH=$JAVA_HOME/bin:$PATH

## 2. **Install Tomcat8**

> apt-get install tomcat8 usuall includes the default java. If you install openjdk first, apt install will just use it. But check after installlation and  use reconfig java to choose any other version. However, if you just use Metacat, then you need only java 8.

sudo apt-get install tomcat8

> the apt-get install of tomcat8 will create folder /var/lib/tomcat8. under tomcat8 there is a folder called conf.
> you need check server.xml and enable port 8009 to connect apache2 and tomcat8.
 >> First to check if AJP 1.3 is open.
 >> You need go to the config file locates in /var/lib/tomcat8/conf/server.xml.
 >> Find out prot 8009 and uncomment it.

sudo gedit /var/lib/tomcat8/conf/server.xml

> search prot 8009 and unmarked the connector port as following
  <!-- Define an AJP 1.3 Connector on port 8009 -->
  <!--(take this line out, if marked)
  <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
  -->(take this line out)

> then restart tomcat

sudo /etc/init.d/tomcat8 restart

## 3. **Install Apache and Tomcat connector**

sudo apt-get install apache2
sudo apt-get install libapache2-mod-jk

> this module will be located in /etc/apache2/mods-available)

> start the mod-jk (if it is not enabled)

sudo a2enmod jk

## **4. Install LDAP**

### install the system

sudo apt-get install slapd ldap-utils

> during installation you will be asked to set up administrator's password

> configure the system first you need load the three schemas (no need to do during installation time)

sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/ldap/schema/cosine.ldif
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/ldap/schema/nis.ldif
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/ldap/schema/inetorgperson.ldif

### check install

sudo slapcat

### configure the ldap system

sudo dpkg-reconfigure slapd

>answer the following question to set up base

###### 1. Omit OpenLDAP server configuration?--NO
###### 2. DNS domain name: --ecoinformatics.org
###### 3. Organization name:--tern
###### 4. Administrator passwod
###### 5. Confirm password:--firelab
###### 6. Database backend to use:--MDB
###### 7. Do you want the database to be removed when slapd is purged?--yes
###### 8. Move old database?--yes
###### 9. Allow ldapv2 protocol?--no

> the above operate will allow you to create your base as dc=ecoinformatics,dc=org.
> and has a cn=admin with password "firelab"

### check your coinfuguration.you will see the result.

sudo slapcat

### create layer
> such as you want have o=TERN (this file is for Taiwan Ecological Research Netowrk. I edited. you can change to other o=xxxx)

sudo ldapadd -x -D cn=admin,dc=ecoinformatics,dc=org -W -f /home/chin/o=tern.ldif

> content of the file o=eapilter.ldif as following;

------------------------------------------------------------------------------------
dn: o=TERN,dc=ecoinformatics,dc=org
o: TERN
objectClass: organization
objectClass: top

------------------------------------------------------------------------------------

### add users you want (this adduser.ldif is for o=TERN, I edited. you can change to what you like)

sudo ldapadd -x -D cn=admin,dc=ecoinformatics,dc=org -W -f /home/chin/adduser_tern.ldif

> content of adduser_tern.ldif as following;

------------------------------------------------------------------
dn: uid=chin,o=TERN,dc=ecoinformatics,dc=org
cn: cc
sn: lin
uid: chin
userPassword: firelab
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: uidObject
objectClass: top

-------------------------------------------------------------------


# install phpldapadmin for managing ldap, if you like gui infterfact to manage ldap server.

sudo apt-get install phpldapadmin

### modify configure file at /etc/phpldapadmin/config.php

sudo cp /etc/phpldapadmin/config.php /etc/phpldapadmin/config.php.bak
sudo gedit /etc/phpldapadmin/config.php

>modidfy the following lines (line number may be different);

-------------------------------------------------------------------------------
in line 293 $servers->setValue('server','host','127.0.0.1');
in line 300 $servers->setValue('server','base',array('dc=ecoinformatics,dc=org'));
in line 318 $servers->setValue('login','bind_id','session');
in 326$servers->setValue('login','bind_id','cn=admin,dc=ecoinformatics,dc=org');

-------------------------------------------------------------------------------

### access LDAP server by http://localhost/phpldapadmin

## **5. Install Postgresql**

### Install the server

sudo apt-get install postgresql pgadmin3

> config file directory is /var/lib/postgresql/9.1/main/pg_hba.conf

> notice the version of postgresql might be different. You need chech it.

### make Postgresql accessible

> By default Postgresql will create a user name: postgres in Ubuntu. But you don’t know the password. Therefore, you need make user: postgres a new password. Then you can use pgadmin to manage your database. The scripts (commands) are as following:

sudo su postgres -c psql template1
ALTER USER postgres WITH PASSWORD 'tari';
\q

> you also can use command scripts to create database

sudo su - postgres
createdb metacat
psql metacat
CREATE USER metacat with UNENCRYPTED PASSWORD 'tari';
\q
exit

### access posgresql server for gui interface that you can create users too.

sudo pgadmin3

## **6. Install Solr server**
> Metacat 2.13.0 needs a seperate Solr server outside Metacat. You need install it before installing Metacat.

> Download and 
cd /opt 
sudo wget https://archive.apache.org/dist/lucene/solr/8.4.1/
sudo tar xzf solr-8.4.1.tgz solr-8.4.1/bin/install_solr_service.sh --strip-components=2
sudo bash ./install_solr_service.sh solr-8.4.1.tgz
sudo chmod g+w /etc/default/solr.in.sh
sudo usermod -a -G solr tomcat8
sudo usermod -a -G tomcat8 solr

> You can start or stop solr by sudo service solr stop or sudo service solr start
> You can check if Solr is running by typing http://localhost:8983/solr

## **7. Install Metacat**
>before you install Metacat, you need make Metacat convext in Apache to let http can access Tomcat6.
>if you don't have jk.conf and workers.properties. you need to edit it first. you can just use these two files I provide.

### make apache2 and tomcat connected

sudo cp /home/chin/jk.conf /etc/apache2/mods-available/
sudo cp /home/chin/workers.properties /etc/apache2/workers.properties

> The content of jk.conf

---------------------------------------------------------
JkWorkersFile   /etc/apache2/workers.properties
JkLogFile       /var/log/apache2/mod_jk.log
JkShmFile       /var/log/apache2/mod_jk.shm

---------------------------------------------------------

> The content of workers.properties

---------------------------------------------------------
workers.tomcat_home=/var/lib/tomcat7
workers.java_home=/usr/lib/jvm/jdk1.7.0_80
worker.list=ajp13
worker.ajp13.port=8009
worker.ajp13.host=localhost
worker.ajp13.type=ajp13
worker.ajp13.lbfactor=1
worker.loadbalancer.type=lb

----------------------------------------------------------

### Set up an apache2 virtual host

sudo gedit /etc/apache2/sites-available/metacat.conf

> The following is the content of httpd.conf (you can add JkMonut context by your own need). Notice ServerName can be changed to fit your own server.

---------------------------------------------------------

<VirtualHost *:80>

        DocumentRoot /var/www/html
        ServerName localhost
        
        JkMount /metacat ajp13
        JkMount /metacat/* ajp13
        JkMount /metacatui ajp13
        JkMount /metacatui/* ajp13

</VirtualHost>


---------------------------------------------------------

### enable metacat.conf and restart apache2

### create metacat data folder
> Metacat needs a folder to store files such as raw dataset,logs. So you need create a folder called metacat under /var. During installation this folder need to be able to write (writable). you can change the right of folder after completing installation.

sudo mkdir /var/metacat
sudo chown -R tomcat8:tomcat8 /var/metacat

sudo chmod -R 775 /var/metacat

### enable virtual host and restart apache2 and tomcat8

sudo a2ensite metacat.conf
sudo /etc/init.d/tomcat7 restart
sudo /etc/init.d/apache2 restart

### copy .war files to /var/lib/tomcat7/webapps

> metacat bin .war files can be downloaded from https://knb.ecoinformatics.org. the latest version of this writing is 12.12.3.
> the version 2.13.0 you need metacat.war, metacatui.war, and metacat-index.war.

sudo cp *.war /var/lib/tomcat7/webapps

### start to configure Metacat

> configure metacat by tpying http://localhost/metacat and following the screen to finish the settings.
> you should see configuration metacat interface. Just follow the instruction, you should be able to install Metacat and run it.

1. first screen shows up is Authentication configuration

 1) Authentication URL: ldap://localhost:389/

 2) Authentication Secure URL: ldap://localhost:389/

 3) Metacat Administrators: uid=chin,o=TERN,dc=ecoinformatics,dc=org

 4) after save the first setp. the administroator login shows up, you need type ldap password of metacat administrator that you define in the first step.

 5) if the password is correct, then you will access Metacat Configuration.
   -Metacat Global Properties   
   - Authentication Configuration   
   - Skins specfic properties   
   - Database Installation/Upgrade   
   - Solr serach index (choose create)
   - Geoserver Configuration (choose bypass)  
   - DataOne (choose bypass)

2. Metacat Global Properties
   - Database Username: metacat
   - Database password: firelab
   - JDBC connection string: you can change to what the database name  h
   - Serve Name : you can use something like 192.168.10 or others
   - Metacat context: here is knb (this must match with knb.war
   - Data file path: you can change
   - Inline Data file path:you can change
   - Document file path: this is metacat documents path
   - Temporary file path:for installation backup path

3. Skins Specific Propterties    
   - 2.13.0 default skin is metacatui

4. Database Install/Upgrade utility
   - you need to make sure database name that JDBC will connect
   - also make shure the path /usr/share/tomcat6/webapps/knb/WEB-INF/sql/xmltables-postgres.sql is correct.

5. Geosever configuration
   - you can choose bypass

6. DataOne configuration
   - you can just choose bypass

7. you should see all the Metacat configuration items show green and tell you the configuration of Metacat is complete

8. you can go to metacat by typing http://localhost/metacat

9. if you can access metacat interface (skin), you should test if it is really success by searching % on the "search data catalog".

# tomcat7 has cookie rules which will conflict to morpho, solution as the follows

### first enable headers module

sudo  a2enmod headers

### then add the lines to apache2.conf

Header edit* Set-Cookie "(JSESSIONID=.*)(; Secure)" "$1"

Header edit* Set-Cookie "(JSESSIONID=.*)(; HttpOnly)" "$1"

Header edit* Set-Cookie "(JSESSIONID=.*)(; No \'=\')" "$1"   

# Enjoy your installation!!
