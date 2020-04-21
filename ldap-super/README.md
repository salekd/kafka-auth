# Authentication with LDAP + Authorization using ACLs + Super User

Run OpenLDAP server.

```
docker run -p 389:389 -p 636:636 --name my-openldap-container --detach osixia/openldap:1.3.0

docker exec my-openldap-container ldapsearch -x -H ldap://localhost -b dc=example,dc=org -D "cn=admin,dc=example,dc=org" -w admin
```

Create a user.

```
docker exec -it my-openldap-container bash

echo """dn: ou=people,dc=example,dc=org
objectClass: organizationalUnit
ou: people""" > people.ldif

ldapadd -x -D "cn=admin,dc=example,dc=org" -w admin -H ldap://localhost -f adam.ldif

echo """dn: uid=adam,ou=people,dc=example,dc=org
objectClass: top
objectClass: account
objectClass: posixAccount
objectClass: shadowAccount
cn: adam
uid: adam
uidNumber: 16859
gidNumber: 100
homeDirectory: /home/adam
loginShell: /bin/bash
gecos: adam
userPassword: adam-secret
shadowLastChange: 0
shadowMax: 0
shadowWarning: 0""" > adam.ldif

ldapadd -x -D "cn=admin,dc=example,dc=org" -w admin -H ldap://localhost -f adam.ldif

echo """dn: uid=bob,ou=people,dc=example,dc=org
objectClass: top
objectClass: account
objectClass: posixAccount
objectClass: shadowAccount
cn: bob
uid: bob
uidNumber: 16859
gidNumber: 100
homeDirectory: /home/bob
loginShell: /bin/bash
gecos: bob
userPassword: bob-secret
shadowLastChange: 0
shadowMax: 0
shadowWarning: 0""" > bob.ldif

ldapadd -x -D "cn=admin,dc=example,dc=org" -w admin -H ldap://localhost -f bob.ldif

ldapsearch -x -H ldap://localhost -b dc=example,dc=org -D "cn=admin,dc=example,dc=org" -w admin

ldapwhoami -vvv -h ldap://localhost -p 389 -D "uid=adam,ou=people,dc=example,dc=org" -x -w adam-secret
ldapwhoami -vvv -h ldap://localhost -p 389 -D "uid=bob,ou=people,dc=example,dc=org" -x -w bob-secret
```

```
cp server.properties ~/confluent-5.4.1/etc/kafka/server.properties

confluent local start kafka

# Only Adam (super user) can create topics.
kafka-topics --create --bootstrap-server localhost:9093 --command-config client.properties_adam --replication-factor 1 --partitions 1 --topic adam
kafka-topics --create --bootstrap-server localhost:9093 --command-config client.properties_bob --replication-factor 1 --partitions 1 --topic bob
kafka-topics --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test

# Only Adam (super user) can list topics.
kafka-topics --list --bootstrap-server localhost:9093 --command-config client.properties_adam
kafka-topics --list --bootstrap-server localhost:9093 --command-config client.properties_bob
kafka-topics --list --bootstrap-server localhost:9092

confluent local destroy
```
