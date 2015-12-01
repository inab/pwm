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

* Access to web interface (https://testcas.rd-connect.eu:9443/pwm/) and start configuration guide:

    Step 1.- Select template OpenLDAP 
		Image1
		
    Step 2.- LDAP>LDAP Directories>Default:
		Image2a
			2a.1.- Change LDAP URL to ldaps://ldap.rd-connect.eu:636
			2a.2.- LDAP Certificates: Import from Server
			2a.3.- LDAP Proxy User: cn=admin,dc=rd-connect,dc=eu
			2a.4.- LDAP Proxy Password: Change it!
			2a.5- LDAP Contextless Login Root: dc=rd-connect,dc=eu
			2a.6.- LDAP Test User: cn=pwmTestUser,ou=people,dc=rd-connect,dc=eu
		Image2b
			2b.1.- Username Search Filter: (&(objectClass=inetOrgPerson)(objectClass=basicRDproperties)(|(uid=%USERNAME%)(mail=%USERNAME%))(disabledAccount=FALSE))
			2b.2.- Attribute to use for Username: uid
			2b.3.- LDAP GUID Attribute: entryUUID
			2b.4.- LDAP Profile Display Name: cn
			
    Step 3.- Modules>Administration:
		Image3
			Administration Permission:
				LDAP Profile: default
				LDAP Search Filter: (&(objectClass=basicRDproperties)(memberOf=cn=pwmAdmin,ou=groups,dc=rd-connect,dc=eu))
				LDAP Base DN: ou=people,dc=rd-connect,dc=eu
	
	Step 4.- Modules>Change Password:
		Image4
			Change Password Permission:
				LDAP Profile: all
				LDAP Search Filter: (&(objectClass=basicRDproperties)(disabledAccount=FALSE))
				LDAP Base DN: ou=people,dc=rd-connect,dc=eu
				
			Require Current Password during change: True
	
	Step 5.- Modules>Forgotten Password>Forgotten Password Profiles>default:
		Image5
			Forgotten Password Password Profile Match
				LDAP Profile: all
				LDAP Search Filter: (&(objectClass=basicRDproperties)(disabledAccount=FALSE))
				LDAP Base DN: ou=people,dc=rd-connect,dc=eu
	
	Step 6.- Modules>Forgotten Password>Forgotten Password Settings
		Image6
				Forgotten Password User Search Form: See image6
				Forgotten Password User Search Filter: (&(objectClass=basicRDproperties)(objectClass=inetOrgPerson)(uid=%uid%)(mail=%mail%)(disabledAccount=FALSE))
				
	Step 7.- Policies>Challenge Policies>default
		Image7a
				Challenge Profile Match:
					LDAP Profile: all
					LDAP Search Filter: (&(objectClass=basicRDproperties)(disabledAccount=FALSE))
				    LDAP Base DN: ou=people,dc=rd-connect,dc=eu
		Image7b
				Minimum Random Required: 1
				Minimum Random Challenges Required During Setup: 3
	
	Step 8.- Policies>Challenge Settings
		Image8
				Save Challenge Permissions:
					LDAP Profile: all
					LDAP Search Filter: (&(objectClass=basicRDproperties)(disabledAccount=FALSE))
					LDAP Base DN: ou=people,dc=rd-connect,dc=eu
				Check Response Match:
					LDAP Profile: all
					LDAP Search Filter: (&(objectClass=basicRDproperties)(disabledAccount=FALSE))
					LDAP Base DN: ou=people,dc=rd-connect,dc=eu
	
	Step 9.- Policies> Password Policies>default
		Image9a
				Password Policy Profile Match
					LDAP Profile: all
					LDAP Search Filter: (&(objectClass=basicRDproperties)(disabledAccount=FALSE))
					LDAP Base DN: ou=people,dc=rd-connect,dc=eu
				Minimum Length: 12
				Maximum sequential Repeat: 3
				Minimum numeric: 1
		Image9b
				Minimum Alphabetic: 1
				Minimum Uppercase: 1
				Minimum Lowercase: 1
		Image9c
				Disallowed Values: Add any amount of disallowed values. See image for an example of them
				Disallowed Attributes: cn, givenName, sn, mail, uid
				Minimum Password Strength: 45
				Maximum Consecutive Characters: 4
	
	Step 10.- Policies>Password Settings:
		Image10
				Password Policy Source: Local
				Password is case sensitive: True
				
	Step 11.- Settings>Application:
		Image11
				Site URL: https://rdconnectcas.rd-connect.eu:9443/pwm
				Enable Version Checking: False
				Enable Anonymous Statistics Publishing: False
				Logout URL: http://rd-connect.eu/
				Home URL: https://platform.rd-connect.eu/
	
	Step 12.- Settings>Captcha:
		Image12
				Captcha Protected Pages:
					Enabled: Login Form, Forgotten Password, New User Registration
	
	Step 13.- Settings>Security>Application Security:
		Image 13
				Security: Changeit!!
					
	Step 14.- Settings>Security>Web Security:
		Image14
				Require HTTPS: True
    
    Step 15.- Save changes 	

* Restart Tomcat
```bash
systemctl restart tomcat7
```
* Login with the user root and in the upper right corner menu click on "Set Configuration Password" (icon with key shape).
Image15
* Set Configuration Password
image16
* Click Save icon in the upper right corner menu to save the configuration password
* Go to Configuration Manager and click on "Restrict Configuration".
