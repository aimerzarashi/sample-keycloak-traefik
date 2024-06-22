## Create a Self Signed root certificate authority (CA):

### let's proceed with the creation of the CA certificate

```
openssl req -x509 -sha256 -days 3650 -newkey rsa:4096 -keyout rootCA.key -out rootCA.pem -subj "/C=JP/ST=Tokyo/O=rootCA/CN=rootCA"
1234
```

## Host Certificate:

### creating what is known as a certificate signing request (CSR)

```
openssl req -new -newkey rsa:4096 -keyout server.key -out server.csr -nodes -subj "/C=JP/ST=Tokyo/O=Keycloak/CN=*.aimerzarashi.com"
```

### sign the request with our rootCA.crt certificate and its private key

```
openssl x509 -req -CA rootCA.pem -CAkey rootCA.key -in server.csr -out server.pem -days 365 -CAcreateserial -extfile server.ext

openssl x509 -in server.pem -text
```

## Client (User) certificate:

```
openssl req -new -newkey rsa:4096 -nodes -keyout user1.key -out user1.csr -subj "/C=JP/ST=Tokyo/O=Client/CN=*.aimerzarashi.com/emailAddress=user1@example.com"

openssl x509 -req -CA rootCA.pem -CAkey rootCA.key -in user1.csr -out user1.pem -days 365 -CAcreateserial

openssl pkcs12 -export -out user1.p12 -name "user1" -inkey user1.key -in user1.pem**
```
