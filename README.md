# How To Install and Configure a Basic LDAP Server

The following parameters are used to configure the ldap server. <br />

**Domain**:	vinil.com<br />
**Domain DN**:	dc=vinil,dc=com<br />
**LDAP Server**:	ldapserver.vinil.com<br />
**OS Version** : Red Hat Enterprise Linux Server release 7.9 (Maipo)<br />

**Installing packages required to configure Ldap server:<br />**

[root@ldapserver ~]# yum -y install openldap-servers openldap-clients<br />

**copying the sample config file and starting the service<br />**

[root@ldapserver ~]# cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG<br />
[root@ldapserver ~]# chown ldap. /var/lib/ldap/DB_CONFIG<br />
[root@ldapserver ~]# systemctl start slapd<br />
[root@ldapserver ~]# systemctl enable slapd<br />


**Set OpenLDAP admin password.<br />**
	
[root@ldapserver ~]# slappasswd<br />
New password:<br />
Re-enter new password:<br />
**{SSHA}lQiyFGZXw4Uk3F2Bic74EbShG3Fl6C57<br />**

specify the password generated above for "olcRootPW" section <br />

[root@ldapserver ~]# vi chrootpw.ldif<br />
dn: olcDatabase={0}config,cn=config<br />
changetype: modify<br />
add: olcRootPW<br />
olcRootPW: **{SSHA}lQiyFGZXw4Uk3F2Bic74EbShG3Fl6C57<br />**

[root@ldapserver ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f chrootpw.ldif<br />
SASL/EXTERNAL authentication started<br />
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth<br />
SASL SSF: 0<br />
modifying entry "olcDatabase={0}config,cn=config"<br />

**Import basic schemas<br />**

[root@ldapserver ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif<br />
SASL/EXTERNAL authentication started<br />
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth<br />
SASL SSF: 0<br />
adding new entry "cn=cosine,cn=schema,cn=config"<br />

[root@ldapserver ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif<br />
SASL/EXTERNAL authentication started<br />
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth<br />
SASL SSF: 0<br />
adding new entry "cn=nis,cn=schema,cn=config"<br />

[root@ldapserver ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif<br />
SASL/EXTERNAL authentication started<br />
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth<br />
SASL SSF: 0<br />
adding new entry "cn=inetorgperson,cn=schema,cn=config"<br />

**Set your domain name on LDAP DB.**<br />

[root@ldapserver ~]# slappasswd<br />
New password:<br />
Re-enter new password:<br />
**{SSHA}E1rizZbX2PijOZrh0JYb0E8VwZF+jshy<br />**

[root@ldapserver ~]# vi chdomain.ldif<br />
dn: olcDatabase={1}monitor,cn=config<br />
changetype: modify<br />
replace: olcAccess<br />
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth"<br />
  read by dn.base="cn=Manager,dc=vinil,dc=com" read by * none<br />

dn: olcDatabase={2}hdb,cn=config<br />
changetype: modify<br />
replace: olcSuffix<br />
olcSuffix: dc=vinil,dc=com<br />

dn: olcDatabase={2}hdb,cn=config<br />
changetype: modify<br />
replace: olcRootDN<br />
olcRootDN: cn=Manager,dc=vinil,dc=com<br />

dn: olcDatabase={2}hdb,cn=config<br />
changetype: modify<br />
add: olcRootPW<br />
olcRootPW: **{SSHA}E1rizZbX2PijOZrh0JYb0E8VwZF+jshy<br />**

dn: olcDatabase={2}hdb,cn=config<br />
changetype: modify<br />
add: olcAccess<br />
olcAccess: {0}to attrs=userPassword,shadowLastChange by<br />
  dn="cn=Manager,dc=vinil,dc=com" write by anonymous auth by self write by * none<br />
olcAccess: {1}to dn.base="" by * read<br />
olcAccess: {2}to * by dn="cn=Manager,dc=vinil,dc=com" write by * read<br />

[root@ldapserver ~]# ldapmodify -Y EXTERNAL -H ldapi:/// -f chdomain.ldif<br />
SASL/EXTERNAL authentication started<br />
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth<br />
SASL SSF: 0<br />
modifying entry "olcDatabase={1}monitor,cn=config"<br />

modifying entry "olcDatabase={2}hdb,cn=config"<br />

modifying entry "olcDatabase={2}hdb,cn=config"<br />

modifying entry "olcDatabase={2}hdb,cn=config"<br />

modifying entry "olcDatabase={2}hdb,cn=config"<br />


[root@ldapserver ~]# vi basedomain.ldif<br />
dn: dc=vinil,dc=com<br />
objectClass: top<br />
objectClass: dcObject<br />
objectclass: organization<br />
o: Vinil Com<br />
dc: vinil<br />

dn: cn=Manager,dc=vinil,dc=com<br />
objectClass: organizationalRole<br />
cn: Manager<br />
description: Directory Manager<br />

dn: ou=People,dc=vinil,dc=com<br />
objectClass: organizationalUnit<br />
ou: People<br />

dn: ou=Group,dc=vinil,dc=com<br />
objectClass: organizationalUnit<br />
ou: Group<br />

[root@ldapserver ~]# ldapadd -x -D cn=Manager,dc=vinil,dc=com -W -f basedomain.ldif<br />
Enter LDAP Password:<br />
adding new entry "dc=vinil,dc=com"<br />

adding new entry "cn=Manager,dc=vinil,dc=com"<br />

adding new entry "ou=People,dc=vinil,dc=com"<br />

adding new entry "ou=Group,dc=vinil,dc=com"<br />

**Checking the ldap server setup:<br />**

[root@ldapserver ~]# ss -tlpnu | grep slapd<br />
tcp    LISTEN     0      128       *:389                   *:*                   users:(("slapd",pid=1731,fd=8))<br />
tcp    LISTEN     0      128    [::]:389                [::]:*                   users:(("slapd",pid=1731,fd=9))<br />

**verifying the config file <br />**

[root@ldapserver ~]# slaptest<br />
config file testing succeeded<br />

**running ldapsearch<br />**

[root@ldapserver ~]# ldapsearch -x -b "dc=vinil,dc=com" -H ldap://ldapserver.vinil.com

**Adding some ldap users for testing<br />**
run useradd to add some users in ldapserver and set the password.<br />

[root@ldapserver ~]# useradd user1<br />
[root@ldapserver ~]# echo "vinilv123" | passwd --stdin user1<br />
Changing password for user user1.<br />
passwd: all authentication tokens updated successfully.<br />

**create ldapuser.ldif file**
dn: uid=user1,ou=People,dc=vinil,dc=com
uid: user1
cn: user1
sn: user1
mail: user1@vinil.com
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
userPassword: $6$ZCeX8kYx$0GIARL9NBsuQcJLTi2jNOOzsaDJbey.4.ElK06bX15QBQxPWeGZi8sTwOFEtl2kbA4Q3fnYXhm7WkZtvww9ip1
shadowLastChange: 17449
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 1001
gidNumber: 1001
homeDirectory: /home/user1

**Add the users to ldap server using ldapadd<br />**

[root@ldapserver ~]# ldapadd -x -D cn=Manager,dc=vinil,dc=com -W -f ldapuser.ldif<br />
Enter LDAP Password:<br />
adding new entry "uid=user1,ou=People,dc=vinil,dc=com"<br />
**set the password for user1**
ldappasswd -s user1@123 -W -D "cn=Manager,dc=vinil,dc=com" -x "uid=user1,ou=People,dc=vinil,dc=com"

