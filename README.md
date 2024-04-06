# mtls-example

## Install

Install dependencies

```
yarn
```

Add fake servers and clients:

```
// AS ROOT
# echo '127.0.0.1 server.aaa.com' >> /etc/hosts
# echo '127.0.0.1 client.bbb.com' >> /etc/hosts
```

Create a server certificate:

```
openssl req -nodes -new -x509 -days 365 -keyout server-ca-key.pem -out server-ca-crt.pem
```

Fill out all the questions and use `aaa.com` as common name.

Create a random key:

```
openssl genrsa -out server-key.pem 4096
```

Create a  Certificate Signing Request with the key:

```
openssl req -new -sha256 -key server-key.pem -out server-csr.pem
```

The common name should now be `server.aaa.com`.

Create a server certificate:

```
openssl x509 -req -days 365 -in server-csr.pem -CA server-ca-crt.pem -CAkey server-ca-key.pem -CAcreateserial -out server-crt.pem
```

Verify the certificate:

```
openssl verify -CAfile server-ca-crt.pem server-crt.pem
```

Do the same steps for the client but use now `bbb.com` and `client.bbb.com` as common names:

```
openssl req -nodes -new -x509 -days 365 -keyout client-ca-key.pem -out client-ca-crt.pem
openssl genrsa -out client-key.pem 4096
openssl req -new -sha256 -key client-key.pem -out client-csr.pem
openssl x509 -req -days 365 -in client-csr.pem -CA client-ca-crt.pem -CAkey client-ca-key.pem -CAcreateserial -out client-crt.pem
openssl verify -CAfile client-ca-crt.pem client-crt.pem
```

Move all the generated files to the certs directory:

```
mv *.pem certs/
mv *.srl certs/
```

### Demo

Start the server:

```
node server.js
```

Test a node client:

```
node client.js
```

Test a curl client:

```
curl --cacert certs/server-ca-crt.pem --key certs/client-key.pem --cert certs/client-crt.pem https://server.aaa.com:8888/
```

The client output should say:

```
OK!
```

The server output should say:

```
Connection from client: client.bbb.com
Sat Apr 06 2024 13:25:17 GMT+0200 (Central European Summer Time) ::ffff:127.0.0.1 GET /
```

### See also

- https://www.matteomattei.com/client-and-server-ssl-mutual-authentication-with-nodejs/
- https://sebtrif.xyz/blog/2019-10-03-client-side-ssl-in-node-js-with-fetch/
