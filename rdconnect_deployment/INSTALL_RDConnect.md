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
=====
* Step 1.- Select template OpenLDAP 
 ![](https://github.com/inab/pwm/blob/master/rdconnect_deployment/images/step1.png)  
		
* Step 2.- LDAP>LDAP Directories>Default:
 ![](https://github.com/inab/pwm/blob/master/rdconnect_deployment/images/step2a.png)  
 - LDAP URL: ldaps://ldap.rd-connect.eu:636.  
 - LDAP Certificates: Import from Server.   
 - LDAP Proxy User: cn=admin,dc=rd-connect,dc=eu  
 - LDAP Proxy Password: Change it!  
 - LDAP Contextless Login Root: dc=rd-connect,dc=eu  
 - LDAP Test User: cn=pwmTestUser,ou=people,dc=rd-connect,dc=eu  
 ![](https://github.com/inab/pwm/blob/master/rdconnect_deployment/images/step2b.png)  
 - Username Search Filter: (&(objectClass=inetOrgPerson)(objectClass=basicRDproperties)(|(uid=%USERNAME%)(mail=%USERNAME%))(disabledAccount=FALSE))  
 - Attribute to use for Username: uid  
 - LDAP GUID Attribute: entryUUID  
 - LDAP Profile Display Name: cn  
			
* Step 3.- Modules>Administration:
 ![](https://github.com/inab/pwm/blob/master/rdconnect_deployment/images/step3.png)  
 - Administration Permission:  
 	- LDAP Profile: default  
	- LDAP Search Filter: (&(objectClass=basicRDproperties)(memberOf=cn=pwmAdmin,ou=groups,dc=rd-connect,dc=eu))  
	- LDAP Base DN: ou=people,dc=rd-connect,dc=eu  
	
* Step 4.- Modules>Change Password:
 ![](https://github.com/inab/pwm/blob/master/rdconnect_deployment/images/step4.png)  
 - Change Password Permission:
 	- LDAP Profile: all  
	- LDAP Search Filter: (&(objectClass=basicRDproperties)(disabledAccount=FALSE))  
	- LDAP Base DN: ou=people,dc=rd-connect,dc=eu  
		
 - Require Current Password during change: True  
	
* Step 5.- Modules>Forgotten Password>Forgotten Password Profiles>default:
 ![](https://github.com/inab/pwm/blob/master/rdconnect_deployment/images/step5.png)  
 - Forgotten Password Password Profile Match  
	- LDAP Profile: all  
	- LDAP Search Filter: (&(objectClass=basicRDproperties)(disabledAccount=FALSE))  
	- LDAP Base DN: ou=people,dc=rd-connect,dc=eu  
	
* Step 6.- Modules>Forgotten Password>Forgotten Password Settings
 ![](https://github.com/inab/pwm/blob/master/rdconnect_deployment/images/step6.png)  
 - Forgotten Password User Search Form: See image6  
 - Forgotten Password User Search Filter: (&(objectClass=basicRDproperties)(objectClass=inetOrgPerson)(uid=%uid%)(mail=%mail%)(disabledAccount=FALSE))  
				
* Step 7.- Policies>Challenge Policies>default
 ![](https://github.com/inab/pwm/blob/master/rdconnect_deployment/images/step7a.png)  
 - Challenge Profile Match:  
	- LDAP Profile: all  
	- LDAP Search Filter: (&(objectClass=basicRDproperties)(disabledAccount=FALSE))  
	- LDAP Base DN: ou=people,dc=rd-connect,dc=eu  
 ![](https://github.com/inab/pwm/blob/master/rdconnect_deployment/images/step7b.png)  
 - Minimum Random Required: 1  
 - Minimum Random Challenges Required During Setup: 3  
	
* Step 8.- Policies>Challenge Settings
 ![](https://github.com/inab/pwm/blob/master/rdconnect_deployment/images/step8.png)  
 - Save Challenge Permissions:  
	- LDAP Profile: all  
	- LDAP Search Filter: (&(objectClass=basicRDproperties)(disabledAccount=FALSE))  
	- LDAP Base DN: ou=people,dc=rd-connect,dc=eu  
 - Check Response Match:
	- LDAP Profile: all  
	- LDAP Search Filter: (&(objectClass=basicRDproperties)(disabledAccount=FALSE))  
	- LDAP Base DN: ou=people,dc=rd-connect,dc=eu  
	
* Step 9.- Policies> Password Policies>default
 ![](https://github.com/inab/pwm/blob/master/rdconnect_deployment/images/step9a.png)  
 - Password Policy Profile Match  
	- LDAP Profile: all  
	- LDAP Search Filter: (&(objectClass=basicRDproperties)(disabledAccount=FALSE))  
	- LDAP Base DN: ou=people,dc=rd-connect,dc=eu  
	- Minimum Length: 12  
	- Maximum sequential Repeat: 3  
	- Minimum numeric: 1  
![](https://github.com/inab/pwm/blob/master/rdconnect_deployment/images/step9b.png)  
 - Minimum Alphabetic: 1  
 - Minimum Uppercase: 1  
	Minimum Lowercase: 1  
![](https://github.com/inab/pwm/blob/master/rdconnect_deployment/images/step9c.png)  
 - Disallowed Values: Add any amount of disallowed values. See image for an example of them  
 - Disallowed Attributes: cn, givenName, sn, mail, uid  
 - Minimum Password Strength: 45  
 - Maximum Consecutive Characters: 4  
	
* Step 10.- Policies>Password Settings:
 ![](https://github.com/inab/pwm/blob/master/rdconnect_deployment/images/step10.png)  
 - Password Policy Source: Local  
 - Password is case sensitive: True  
				
* Step 11.- Settings>Application:
 ![](https://github.com/inab/pwm/blob/master/rdconnect_deployment/images/step11.png)  
 - Site URL: https://rdconnectcas.rd-connect.eu:9443/pwm  
 - Enable Version Checking: False  
 - Enable Anonymous Statistics Publishing: False  
 - Logout URL: http://rd-connect.eu/  
 - Home URL: https://platform.rd-connect.eu/  
	
* Step 12.- Settings>Captcha:
 ![](https://github.com/inab/pwm/blob/master/rdconnect_deployment/images/step12.png)  
 - Captcha Protected Pages:  
 	- Enabled: Login Form, Forgotten Password, New User Registration  
	
* Step 13.- Settings>Security>Application Security:
 ![](https://github.com/inab/pwm/blob/master/rdconnect_deployment/images/step13.png)  
 - Security: Changeit!!  
					
* Step 14.- Settings>Security>Web Security:
 ![](https://github.com/inab/pwm/blob/master/rdconnect_deployment/images/step14.png)  
 - Require HTTPS: True  
    
* Step 15.- Save changes 	

* Restart Tomcat
```bash
systemctl restart tomcat7
```
* Login with the user root and in the upper right corner menu click on "Set Configuration Password" (icon with key shape).
 ![](https://github.com/inab/pwm/blob/master/rdconnect_deployment/images/step15.png)  
* Set Configuration Password  
 ![](https://github.com/inab/pwm/blob/master/rdconnect_deployment/images/step16.png)  
* Click Save icon in the upper right corner menu to save the configuration password  
* Go to Configuration Manager and click on "Restrict Configuration".  
