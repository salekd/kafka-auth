# Authentication with LDAP + Authorization using ACLs

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

ldapsearch -x -H ldap://localhost -b dc=example,dc=org -D "cn=admin,dc=example,dc=org" -w admin

ldapwhoami -vvv -h ldap://localhost -p 389 -D "uid=adam,ou=people,dc=example,dc=org" -x -w adam-secret
```

```
cp server.properties ~/confluent-5.4.1/etc/kafka/server.properties

confluent local start kafka

kafka-topics --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test

kafka-topics --list --bootstrap-server localhost:9092

# Everyone can produce because of allow.everyone.if.no.acl.found=true
echo 42 | kafka-console-producer --broker-list localhost:9092 --topic test
echo 42 | kafka-console-producer --broker-list localhost:9093 --topic test --producer.config client.properties_adam
echo 42 | kafka-console-producer --broker-list localhost:9093 --topic test --producer.config client.properties_bob

# Create topic for Adam.
kafka-topics --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic adam 
kafka-acls --bootstrap-server localhost:9092 --add --allow-principal User:adam --operation read --operation write --topic adam
kafka-acls --bootstrap-server localhost:9092 --list --topic adam

# Create topic for Bob.
kafka-topics --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic bob 
kafka-acls --bootstrap-server localhost:9092 --add --allow-principal User:bob --operation read --operation write --topic bob
kafka-acls --bootstrap-server localhost:9092 --list --topic bob

# Adam can produce to his topic.
echo 42 | kafka-console-producer --broker-list localhost:9093 --topic adam --producer.config client.properties_adam
# Adam cannot produce to Bob's topic.
echo 42 | kafka-console-producer  --broker-list localhost:9093 --topic bob --producer.config client.properties_adam

kafka-topics --delete --bootstrap-server localhost:9092 --topic test

confluent local stop kafka

confluent local destroy
```
