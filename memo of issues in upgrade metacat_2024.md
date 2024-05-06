# Memo of metacat upgrade to Metacat 2.19.0
## 1.Isssue of MetacatUI open
### error message : "The config file or loader.js file failed to load. Check the appConfigPath in index.html and the 'root' attribute in AppConfig." 

### 1) Editing by
  sudo nano /opt/tomcat8/webapps/metacatui/index.html
  chaning the var appConfigPath

var appConfigPath = "/config/config.js"; -> var appConfigPath = "config/config.js";

### 2) Editing by 
sudo nano /opt/tomcat8/webapps/metacatui/config/config.js
  Add the following lines:

MetacatUI.AppConfig = {
  //The path to the root location of MetacatUI, i.e. where index.html is
    root: "/metacatui", 

  //The path to the root location of Metacat, i.e. name of the Metacat Tomcat webapp
    metacatContext: "/metacat",

  // show at title of metacatUI html page
    repositoryName: "TARI Data Catalog",

    defaultSearchFilters: ["all", "attribute", "abstract", "description", "documents", "creator", "dataYear", "pubYear", "id", "taxon", "spatial"]
}

## 2 Issue of restoring old eml documents
### 1) After new installation of Metacat 2.19.0
### 2) stop tomcat
### 3) delete installed postgresql database (the name of database ‘metacat’)
      Important notice: the user name ‘postgres’ must be the same as metacat installation used in global configuration. The scripts are as follow:
      dropdb -U postgres -h localhost metacat
      createdb -U postgres -h localhost metacat
      psql -U postgres -h localhost metacat < mybackup.sql
### 4) copy the old data folders of /var/metacat/data and /var/metacat/documents to new installaion. Need give the writing right to the folders.
### 5) restart tomcat and login to metacat 2.19.0. If there is no problem, Metacat 2.19.0 will show reconfiguration of database. It asks to upgrade database to 2.19.0. after upgrading. You need run solr reindex by http://yourhost:8080/metacat/metacat?action=reindexall
### 6) restart machine (I found this is the best way to make sure everything is ok)
### 7) import ubuntu2204.tar
      wsl --import Ubuntu22-04 c:\your preferred folder c:\anyfolder\ubuntu2204.tar
