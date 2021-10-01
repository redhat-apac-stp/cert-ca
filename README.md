# cert-ca
The following procedure can be used to create certs and CA for your projects.

1. Start by generating the private key:

```shell
openssl genrsa -des3 -out myCA.key 2048
```

You can add a pass phrase so that only people who know it can use it to generate more certs.

2. Then generate your root certificate

```shell
openssl req -x509 -new -nodes -key myCA.key -sha256 -days 1825 -out myCA.pem
```
Fill out the details of the root cert.

Now you have to files which are part of your root CA.
myCA.key (private key)
myCA.pem (your root cert)

3. Install your root certs
On a mac:
```shell
sudo security add-trusted-cert -d -r trustRoot -k "/Library/Keychains/System.keychain" myCA.pem
```
Or just import using the keychain access.

On Linux:
TODO

4. Use this cert to generate the certs your your cluster, oc
```bash
#!/bin/sh

if [ "$#" -ne 1 ]
then
  echo "Usage: Must supply a domain"
  exit 1
fi

DOMAIN=$1

cd ~/certs

openssl genrsa -out $DOMAIN.key 2048
openssl req -new -key $DOMAIN.key -out $DOMAIN.csr

cat > $DOMAIN.ext << EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = $DOMAIN
DNS.2 = *.$DOMAIN
EOF

openssl x509 -req -in $DOMAIN.csr -CA ./myCA.pem -CAkey ./myCA.key -CAcreateserial \
-out $DOMAIN.crt -days 825 -sha256 -extfile $DOMAIN.ext
```

Or this example you can append your cluster name and domain ie mycluster.ocp.uluvus

```bash
#!/bin/sh

if [ "$#" -ne 1 ]
then
  echo "Usage: Must supply a domain"
  exit 1
fi

DOMAIN=$1

cd ~/certs

openssl genrsa -out $DOMAIN.key 2048
openssl req -new -key $DOMAIN.key -out $DOMAIN.csr

cat > $DOMAIN.ext << EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = apps.$DOMAIN
DNS.2 = *.apps.$DOMAIN
DNS.3 = api.$DOMAIN
EOF

openssl x509 -req -in $DOMAIN.csr -CA ./myCA.pem -CAkey ./myCA.key -CAcreateserial \
-out $DOMAIN.crt -days 825 -sha256 -extfile $DOMAIN.ext
```


