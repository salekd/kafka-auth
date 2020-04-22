# SASL/PLAIN Authentication with SSL Encryption

```
$ keytool -keystore kafka.server.keystore.jks -alias localhost -keyalg RSA -validity 365 -genkey -storepass 123456 -keypass 123456 -ext SAN=DNS:localhost
What is your first and last name?
  [Unknown]:  localhost
What is the name of your organizational unit?
  [Unknown]:
What is the name of your organization?
  [Unknown]:
What is the name of your City or Locality?
  [Unknown]:
What is the name of your State or Province?
  [Unknown]:
What is the two-letter country code for this unit?
  [Unknown]:
Is CN=localhost, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown correct?
  [no]:  yes

$ openssl req -new -x509 -keyout ca-key -out ca-cert -days 365
Generating a RSA private key
...................+++++
..........................+++++
writing new private key to 'ca-key'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:localhost
Email Address []:

$ keytool -keystore kafka.client.truststore.jks -alias CARoot -import -file ca-cert
$ keytool -keystore kafka.server.truststore.jks -alias CARoot -importcert -file ca-cert

$ keytool -keystore kafka.server.keystore.jks -alias localhost -certreq -file cert-file
$ openssl x509 -req -CA ca-cert -CAkey ca-key -in cert-file -out cert-signed -days 365 -CAcreateserial -passin pass:123456
$ keytool -keystore kafka.server.keystore.jks -alias CARoot -import -file ca-cert
$ keytool -keystore kafka.server.keystore.jks -alias localhost -import -file cert-signed
```

```
cp server.properties ~/confluent-5.4.1/etc/kafka/server.properties

confluent local start kafka

kafka-topics --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test

kafka-topics --list --bootstrap-server localhost:9092

echo 42 | kafka-console-producer --broker-list localhost:9092 --topic test
echo 42 | kafka-console-producer --broker-list localhost:9093 --topic test --producer.config client.properties_alice

kafka-topics --delete --bootstrap-server localhost:9092 --topic test

confluent local stop kafka

confluent local destroy
```
