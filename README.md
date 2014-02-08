Solr-on-OpenShift-with-Tomcat
=============================

Apache SOLR on OpenShift with Apache Tomcat7 with basic security


1. If you don't already have an account, create one at http://openshift.redhat.com/   Don't forget to create a namespace and install client tools as well.

2. Create a new tomcat application following the directions at https://github.com/openshift/openshift-tomcat-quickstart

3. Download the latest version of Solr. The current latest version at the time of this writing is 4.6.1. You can get it from http://ftp.jaist.ac.jp/pub/apache/lucene/solr/4.6.1/solr-4.6.1.tgz . Untar it (it will default to a folder called solr-4.6.1).

Assuming your *tomcat* directory from step 2 and *solr-4.6.1* directory from step 3 are in the same directory and your shell is pointed to that directory.

### Add the solr war
    cp solr-4.6.1/example/webapps/solr.war tomcat/diy/tomcat/webapps/


### Because of this http://wiki.apache.org/solr/SolrLogging issue 
    cp -r solr-4.6.1/example/lib/ext/* tomcat/diy/tomcat/lib/
    cp solr-4.6.1/example/resources/log4j.properties tomcat/diy/tomcat/lib/

### Copy the solr config files over
    cp -r solr-4.6.1/example/solr tomcat/diy/tomcat/
    
### Custom configuration of solrconfig.xml and schema.xml files
If you need a custom configuration for your solr set up now is the time to edit the files in tomcat/diy/tomcat/solr/collection1/conf

### Tomcat Security
Edit tomcat/diy/tomcat/conf/tomcat-users.xml. Replace everything within the ``` <tomcat-user>``` element with (using your own username and password) and save:

```
  <role rolename="manager-gui"/>
  <role rolename="solr_admin"/>
  <user username="userxxxx" password="passwordxxx" roles="manager-gui,solr_admin"/>
```
### Optional Solr Security
Edit web.xml by double clicking into tomcat/diy/tomcat/webapps/solr.war and going to WEB-INF/web.xml
Add the following lines within the ```<web-app>``` element and save the war file:

```
  <security-constraint>
    <web-resource-collection>
      <web-resource-name>Solr Lockdown</web-resource-name>
      <url-pattern>/</url-pattern>
    </web-resource-collection>
    <auth-constraint>
      <role-name>solr_admin</role-name>
      <role-name>admin</role-name>
    </auth-constraint>
  </security-constraint>
  <login-config>
    <auth-method>BASIC</auth-method>
    <realm-name>Solr</realm-name>
  </login-config> 
```

### Upload  
    cd tomcat
    git add .
    git commit
    git push


### Try it out replacing xxxxxx with your OpenShift namespace
http://tomcat-xxxxxx.rhcloud.com/

http://tomcat-xxxxxx.rhcloud.com/solr/#/collection1

### Problems
    rhc tail -a tomcat

from ssh shell logged into your openshift tomcat application
```
ctl_all stop
export JAVA_OPTS="-Dsolr.solr.home=${OPENSHIFT_DATA_DIR}solr.home"
ctl_all start
```

###License
This code is dedicated to the public domain to the maximum extent permitted by applicable law, pursuant to CC0 http://creativecommons.org/publicdomain/zero/1.0/
