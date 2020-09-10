Update host operating system 

# yum update

Install a read file software ie nano

# yum install nano

Install all OpenLDAP packages through

# yum -y install openldap*

Run following commands to start the openldap server daemon

# sudo systemctl start slapd
# sudo systemctl enable slapd
# sudo systemctl status slapd

Next, allow requests to the LDAP server daemon through the firewall

# firewall-cmd --add-service=ldap

OpenLDAP administrative user and assign a password for that user,
a hashed value is created for the given password, take note of it

# slappasswd
ie {SSHA}FODUmmM/koJIlP6pMpwTt17y/tWxsLp5


create an LDIF file (db.ldif) which is used to add an entry to the LDAP directory

# sudo nano db.ldif

    Add the following contents in it:

        dn: olcDatabase={0}config,cn=config
        changetype: modify
        add: olcRootPW
        olcRootPW: {SSHA}FODUmmM/kgJItP6pHpwTt17y/tWxsdK6


Add the corresponding LDAP entry by specifying the URI referring to the ldap server and the file

# sudo ldapadd -Y EXTERNAL -H ldapi:/// -f db.ldif

        #--db.ldif---!
        dn: olcDatabase={2}hdb,cn=config
        changetype: modify
        replace: olcSuffix
        olcSuffix: dc=hadoop,dc=com

        dn: olcDatabase={2}hdb,cn=config
        changetype: modify
        replace: olcRootDN
        olcRootDN: cn=ldapadm,dc=hadoop,dc=com

        dn: olcDatabase={2}hdb,cn=config
        changetype: modify
        replace: olcRootPW
# Add the hash password at the end        olcRootPW: {SSHA}FODUmmM/kgJItP6pHpwTt17y/tWxsdK6

To view contents open 
#       nano olcDatabase\=\{2\}hdb.ldif
    # AUTO-GENERATED FILE - DO NOT EDIT!! Use ldapmodify.
    # CRC32 c268f85d
    dn: olcDatabase={2}hdb
    objectClass: olcDatabaseConfig
    objectClass: olcHdbConfig
    olcDatabase: {2}hdb
    olcDbDirectory: /var/lib/ldap
    olcDbIndex: objectClass eq,pres
    olcDbIndex: ou,cn,mail,surname,givenname eq,pres,sub
    structuralObjectClass: olcHdbConfig
    entryUUID: e3ac0b9c-8783-103a-825e-fbb413872c0a
    creatorsName: cn=config
    createTimestamp: 20200910073505Z
#    olcSuffix: dc=hadoop,dc=com
#    olcRootDN: cn=ldapadm,dc=hadoop,dc=com
#    olcRootPW:: e1NTSEF9Rk9EVW1tTS9rZ0pJdFA2cEhwd1R0MTd5L3RXeHNkSzY=
    entryCSN: 20200910082351.777962Z#000000#000#000000
    modifiersName: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
    modifyTimestamp: 20200910082351Z

Create a (monitor.ldif) to edit the olcDatabase\=\{1\}monitor.ldif

# nano monitor.ldif
# add the following lines;
    dn: olcDatabase={1}monitor,cn=config
    changetype: modify
    replace: olcAccess
    olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external, 
    cn=auth" read by dn.base="cn=ldapadm,dc=hadoop,dc=com" read by * none

Add the corresponding LDAP entry by specifying the URI 

# sudo ldapadd -Y EXTERNAL -H ldapi:/// -f monitor.ldif
        SASL/EXTERNAL authentication started
        SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
        SASL SSF: 0
        modifying entry "olcDatabase={1}monitor,cn=config"

Now copy the sample database configuration file for slapd into the /var/lib/ldap directory, 
and set the correct permissions on the file DB_CONFIG File to /var/lib/ldap directory

# sudo cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
# sudo chown -R ldap:ldap /var/lib/ldap/DB_CONFIG or sudo chown -R ldap:ldap /var/lib/ldap/*
# sudo systemctl restart slapd

import some basic LDAP schemas from the /etc/openldap/schema directory as follows

# sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif 
# sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
# sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif

Create a base.ldif file

# nano base.ldif

Add the base file

# ldapadd -x -W -D "cn=ldapadm,dc=hadoop,dc=com" -f base.ldif - Rwquests for password entry
(All entries added to LDAP)

Run 
# ldapsearch -D cn="ldapadm,dc=hadoop,dc=com" -W -b "dc=hadoop,dc=com" objectclass=*
 To view all records;