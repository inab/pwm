Installation instructions for RD-Connect (CentOS)
=====

* Run this
```bash
git clone https://github.com/inab/pwm.git /tmp/pwm
cd /tmp/pwm
mvn clean package
ln -s "$(basename target/pwm-*.war)" target/pwm.war
```
* As we want the pwm-data directory outside the to-be-deployed pwm.war, we first need to create it:
```bash
mkdir -p /etc/cas/pwm-data
chown tomcat: /etc/cas/pwm-data
```

* We need to tell Tomcat and the webapp where the directory is. So, edit `/etc/sysconfig/tomcat7`, and add next declarations to `JAVA_OPTS`

```bash
export JAVA_OPTS=" -Dpwm.applicationPath=/etc/cas/pwm-data"
```

* Deploy the war

cd /tmp 
wget http://www.pwm-project.org/artifacts/pwm/pwm-1.8.0-SNAPSHOT-2015-11-18T23%3A18%3A27Z-pwm-bundle.zip 
mkdir pwm 
mv pwm-1.8.0-SNAPSHOT-2015-11-18T23\:18\:27Z-pwm-bundle.zip pwm 
cd pwm 
unzip pwm-1.8.0-SNAPSHOT-2015-11-18T23\:18\:27Z-pwm-bundle.zip 
systemctl stop tomcat7 
cp /tmp/pwm/pwm.war /var/lib/tomcat7/webapps/ 
systemctl start tomcat7 

New version need to set up the application path. We used the  Java System Property method: 
mkdir -p /etc/cas/pwm-data 
chown tomcat: /etc/cas/pwm-data 

Modify /etc/sysconfig/tomcat7 adding “-Dpwm.applicationPath=/etc/cas/pwm-data” to the JAVA_OPTS variable: 
     vi /etc/sysconfig/tomcat7 
          export JAVA_OPTS=" -Dpwm.applicationPath=/etc/cas/pwm-data -Djavax.net.ssl.keyStore=/etc/tomcat/cas-tomcat-server.jks -Djavax.net.ssl.keyStorePassword=123.qwe -Djavax.net.ssl.trustStore=/etc/tomcat/cas-tomcat-server.jks -Djavax.net.ssl.trustStorePassword=123.qwe" 

Restart Tomcat 
     systemctl restart tomcat7 


Access to web interface (https://testcas.rd-connect.eu:9443/pwm/) and start configuration guide:
     1.- Select template OpenLDAP 
     2.- Hostname: ldap.rd-connect.eu 
     3.- Enable  (Import/remove certificate manually into Java keystore to change)  
     4.- LDAP Proxy Credentials 
          cn=admin,dc=rd-connect,dc=eu 
          password 
     5.-  LDAP Contextless Login Root: User Container DN  
          ou=people,dc=rd-connect,dc=eu 
     6.- Administrators Group: Administrator Group DN  
          cn=pwmAdmin,ou=groups,dc=rd-connect,dc=eu 
     7.- PASSED 
     8.- Select LDAP 
     9.- Site URL: https://www.rd-connect.eu:9443/pwm 
     10.- Select a Configuration Password 
     11.- Confirm configuration 

