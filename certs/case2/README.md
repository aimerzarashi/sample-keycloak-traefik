## Create a Self Signed root certificate authority (CA):

### let's proceed with the creation of the CA certificate

```
openssl req -x509 -sha256 -days 3650 -newkey rsa:4096 -keyout rootCA.key -out rootCA.crt -subj "/C=JP/ST=Tokyo/O=rootCA/CN=rootCA"
1234
```

## Host Certificate:

### creating what is known as a certificate signing request (CSR)

```
openssl req -new -newkey rsa:4096 -keyout server.key -out server.csr -nodes -subj "/C=JP/ST=Tokyo/O=Keycloak/CN=*.aimerzarashi.com"
```

### sign the request with our rootCA.crt certificate and its private key

```
openssl x509 -req -CA rootCA.crt -CAkey rootCA.key -in server.csr -out server.crt -days 365 -CAcreateserial -extfile server.ext
```

### Create pkcs12 file for server

```
openssl pkcs12 -export -out keystore.p12 -in server.crt -inkey server.key -name "server certificate" -chain -CAfile rootCA.crt -caname "self signed ca certificate" -passin pass:1234 -passout pass:2345

openssl pkcs12 -info -in keystore.p12
```

## Client (User) certificate:

```
openssl req -new -newkey rsa:4096 -nodes -keyout user1.key -out user1.csr -subj "/C=jp/ST=Tokyo/O=Client/CN=*.aimerzarashi.com/emailAddress=user1@example.com"
openssl x509 -req -CA rootCA.crt -CAkey rootCA.key -in user1.csr -out user1.crt -days 365 -CAcreateserial
openssl pkcs12 -export -out user1.p12 -name "user1" -inkey user1.key -in user1.crt** -passin pass:1234 -passout pass:3456
```

### Create a keystore using keytool

```
openssl pkcs12 -in keystore.p12 -out keystore.pem -nodes
2345

docker run -it --rm -v .:/etc/keystore -w /etc/keystore joostdecock/keytool -import -file keystore.pem -keystore keystore.jks -alias "server certificate" -srcstorepass 123456 -deststorepass 123456
```

### Create a truststore using keytool

```
openssl pkcs12 -export -in rootCA.crt -inkey rootCA.key -out truststore.p12 -name "server certificate" -chain -CAfile rootCA.crt -caname "self signed ca certificate" -passin pass:1234 -passout pass:4567

openssl pkcs12 -in truststore.p12 -out truststore.pem -nodes

docker run -it --rm -v .:/etc/keystore -w /etc/keystore joostdecock/keytool -import -file truststore.pem -keystore truststore.jks -alias "server certificate" -srcstorepass 234567 -deststorepass 234567
```
