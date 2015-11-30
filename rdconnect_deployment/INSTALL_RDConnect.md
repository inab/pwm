Installation instructions for RD-Connect (CentOS)
=====

* First, create a user `pwmTestUser` in the LDAP directory, which will be used by PWM periodic checks

```bash
domainDN='dc=rd-connect,dc=eu'
adminName='admin'
adminDN="cn=$adminName,$domainDN"
pwmTestUserHashPass="$(slappasswd -s 'PWMCHANGEIT')"
cat > /tmp/pwm-test-user.ldif <<EOF
dn: cn=pwmTestUser,ou=people,$domainDN
objectClass: inetOrgPerson
objectClass: basicRDproperties
uid: root
disabledAccount: FALSE
userPassword: $pwmTestUserHashPass
cn: pwmTestUser
sn: pwmTestUser
description: A user named pwmTestUser
EOF
ldapadd -x -D "$adminDN" -W -f /tmp/pwm-test-user.ldif
```

* Run this
```bash
git clone https://github.com/inab/pwm.git /tmp/pwm
cd /tmp/pwm
mvn clean package
ln -s "$(basename target/pwm-*.war)" target/pwm.war
```
* As we want the pwm-data directory outside the to-be-deployed pwm.war, we first need to create it:
```bash
mkdir -p /var/lib/pwm-data
chown tomcat: /var/lib/pwm-data
```

* We need to tell Tomcat and the webapp where the directory is. So, edit `/etc/sysconfig/tomcat7`, and add next declarations to `JAVA_OPTS`
```bash
export JAVA_OPTS=" -Dpwm.applicationPath=/var/lib/pwm-data"
```

* Restart Tomcat and deploy the war:
```bash
systemctl restart tomcat7
```

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

