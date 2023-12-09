# mTLS
This is a repo for the mTLS related troubleshooting and details.

It’s a mutual authentication mechanism where both parties that are connected are actually who they claim to be and not an imposter.

By default, the TLS protocol only requires a server to authenticate itself to the client. The authentication of the client to the server is managed by the application layer. The TLS protocol also offers the ability for the server to request that the client send an X.509 certificate to prove its identity. This is called mutual TLS (mTLS) as both parties are authenticated via certificates with TLS.

Mtls protects from malicious api requests, credentials stuffing, phishing attacks, brute force attacks, spoofing attacks etc
Commonly used in OpenBanking solutions, IOT to authenticate devices using digital certificates

---

The client interacts with the server
The server shows its TLS certificate
The client validates the certificate of the server
The client provides the TLS certificate
The client's certificate is verified by the server
The server provides access
The client and server transfer information over an encrypted TLS connection.


---

<img width="244" alt="image" src="https://github.com/adityapandita97/mTLS/assets/72813685/4f1993a6-6b6c-4cd8-bddd-c14c165dd0c8">

<img width="576" alt="image" src="https://github.com/adityapandita97/mTLS/assets/72813685/ba0353e4-8c6c-495d-9635-b3dc5539d8c0">

---

# Setting up mTLS on API Gateway

https://aws.amazon.com/blogs/compute/introducing-mutual-tls-authentication-for-amazon-api-gateway/

After following all the steps, we should have a total of 5 files 
RootCA.key (root CA private key)
RootCA.pem (root CA public key)
my_client.csr (client certificate signing request)
my_client.key (client certificate private key)
my_client.pem (client certificate public key)

---

# Troubleshooting Steps

Check if mTLS is correctly setup on APIGW 

Check S3 URL for truststore bundle and also ensure version id is added in case S3 has bucket versioning enabled

Get complete verbose output of the API Call by asking command to make a curl request with –v attribute

Verify if issuer of client cert is included in the trust store bundle.

Can be achieved by using the following command:

```
openssl x509 -in truststore.pem -text -noout
```
---

# Troubleshooting Steps Server Side

Access denied. Reason: Could not find issuer for certificate" errors

Verify if issuer of client cert is included in the trust store bundle.

Can be achieved by using the following command:
```
openssl verify -CAfile truststore.pem my_client.pem
```
If certificate has multiple intermediate CAs, use the following command:
```
openssl verify -CAfile truststore.pem -untrusted intCA.pem my_client.pem
```
Verify if all clients certificates in the truststore are valid by using the following command:
```
openssl x509 -in truststore.pem -text -noout
```

---

# Troubleshooting Steps Server Side

Access denied. Reason: Client cert using an insecure Signature Algorithm

Verify if the truststore file uses only a supported hashing algorithms

API Gateway supports the following algorithms:

->SHA-256 or stronger
->RSA-2048 or stronger
->ECDSA-256 or stronger

Check the current algorithm used by truststore bundle file by using the following command:

```
openssl x509 -in truststore.pem -text -noout | grep 'Signature Algorithm'
```

---

# Troubleshooting Steps Server Side

Access denied. Reason: self signed certificate" errors

Verify if the modulus of all files (my_client.key, my_client.csr & my_client.pem)

To compare the modulus for each, use the following commands:

```
openssl rsa -noout -modulus -in my_client.key | openssl md5openssl req -noout -modulus -in my_client.csr | openssl md5openssl x509 -noout -modulus -in my_client.pem | openssl md5
```

---

# Troubleshooting Steps Client Side

Missing certificate or private key not passed correctly in the API Call

Incorrect certificate or private key used

Incorrect file extension used, for eg .crt used but the file has the format .pem

---

# Ownership Certificate

When using a private/imported certificate from ACM or AWS Private Certificate Authority, while enabling mTLS, you also have to provide Ownership Certificate issued by ACM

This certificate is not used ANYWHERE during the mTLS Handshake

This is needed just to verify that you have the permissions to use the domain with the help of the imported certificate

NOTE: What if we are using Amazon Issued Public RSA certificate? Well, we know that the OVC is not needed in that case :P

---

# Points to Remember

API Gateway does not verify if the certificate has been revoked or expired

If you have setup lambda auth with your api, the client certs get passed automatically only in 2 case:

apigw passes the client cert to the lambda, if the lambda auth is Request based not the Token based.
apigw passes the client cert to the backend integration, if the integration is proxy

- Apigw doesn’t check for client cert revocation but you can do that inside a lambda auth.
  
To use mTLS, you must use a Regional Custom Domain with a minimum tls version of 1.2

Certificates must be issued from a single issuer only. For each subject in the certificate, there can only be one issuer in API Gateway for mutual TLS domains.

---

"The certificate subject *.<certificate> conflicts with an existing certificate from a different issuer

The above error can also be when using an imported cert for mTLS configuration

*.adityapandita.com - AMAZON ISSUED 
adityapandita.com – IMPORTED

This is due to API gateway only allowing certificates for a domain from one issuer to prevent collision. The use of 2 wildcard certificates causes a collision and hence the error.

---

# Unrelated to mTLS

Customers often get confused when enabling mTLS and thinking the certificate on domain should also have the same CN

When making a curl request for a mTLS custom domain with ACM issued Certificate, you would see the Common Name or the chain as *.execute-api.amazonaws.com

However, with the use of openssl, Customers make the following command to check the certificate chain:

```
openssl s_client -connect mtls.aadianil.awsps.myinstance.com:443 –showcerts
```

The above also shows the same *.execute-api.eu-west-1.amazonaws.comchain

This is because openssl s_client does not send the Server Name Indication by default and it has to be passed to get the correct results

```
openssl s_client -servername mtls.aadianil.awsps.myinstance.com -connect mtls.aadianil.awsps.myinstance.com:443 -showcerts
```

---

# Links that will help

https://aws.amazon.com/blogs/compute/introducing-mutual-tls-authentication-for-amazon-api-gateway/
https://www.freecodecamp.org/news/openssl-command-cheatsheet-b441be1e8c4a/
https://repost.aws/knowledge-center/api-gateway-mutual-tls-403-errors
https://kumo-knowledge-ui-iad-prod.amazon.com/view/article_22299
https://aws.amazon.com/blogs/compute/introducing-mutual-tls-authentication-for-amazon-api-gateway/
https://docs.aws.amazon.com/apigateway/latest/developerguide/rest-api-mutual-tls.html#rest-api-mutual-tls-configure
https://command-center.support.aws.a2z.com/harbinger-notices#/9890d362-a7bc-4803-afd6-f3a7a5d91a72/details/overview
https://knowledge.broadcom.com/external/article/136140/api-gateway-policy-manager-displays-a-di.html










