# SSL

## 一、参考

> [security ssl](https://kafka.apache.org/documentation/#security_ssl)

> [【kafka】SSL和ACL的配置](https://zhuanlan.zhihu.com/p/360159387)


## 二、简介


```
keytool -keystore server.keystore.jks -alias localhost -keyalg RSA -validity 365  -storetype PKCS12 -genkey -dname "CN=10.100.65.93, OU=IT, O=MyCompany, L=Shanghai, ST=Shanghai, C=CN" 

```

```
# password1

keytool -keystore server.keystore.jks -alias localhost -keyalg RSA -validity 365  -storetype PKCS12 -genkey -dname "CN=testkafka.com, OU=IT, O=MyCompany, L=Shanghai, ST=Shanghai, C=CN" -ext "SAN:c=DNS:testkafka.com,IP:10.100.65.93"
Enter keystore password:
Re-enter new password:
Generating 2,048 bit RSA key pair and self-signed certificate (SHA256withRSA) with a validity of 365 days
        for: CN=testkafka.com, OU=IT, O=MyCompany, L=Shanghai, ST=Shanghai, C=CN

```

```

keytool -list -v -keystore server.keystore.jks

```



		
```

openssl req -new -x509 -keyout ca-key -out ca-cert -days 365

```


```
# password2

openssl req -new -x509 -keyout ca-key -out ca-cert -days 365
Generating a 2048 bit RSA private key
........................................+++++
..................................................................................................
..............+++++
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
Country Name (2 letter code) []:CN
State or Province Name (full name) []:Shanghai
Locality Name (eg, city) []:Shanghai
Organization Name (eg, company) []:MyCompany
Organizational Unit Name (eg, section) []:IT
Common Name (eg, fully qualified host name) []:testkafka.com
Email Address []:

```


```

keytool -keystore server.truststore.jks -alias CARoot -import -file ca-cert

```


```
# password3

keytool -keystore server.truststore.jks -alias CARoot -import -file ca-cert
Enter keystore password:
Re-enter new password:
Owner: CN=10.100.65.93, OU=IT, O=MyCompany, L=Shanghai, ST=Shanghai, C=CN
Issuer: CN=10.100.65.93, OU=IT, O=MyCompany, L=Shanghai, ST=Shanghai, C=CN
Serial number: c27acf3a4a31a6b1
Valid from: Mon Jul 31 15:50:19 CST 2023 until: Tue Jul 30 15:50:19 CST 2024
Certificate fingerprints:
         SHA1: 59:2A:98:7D:87:25:57:08:A3:B9:03:A4:C5:2F:60:8B:08:48:63:BA
         SHA256: D6:A6:40:C0:A8:49:0B:AB:4A:79:72:3D:E2:7B:CD:77:BD:F4:61:55:62:63:76:2E:8F:2F:EE:88:F7:73:C7:4B
Signature algorithm name: SHA256withRSA
Subject Public Key Algorithm: 2048-bit RSA key
Version: 1
Trust this certificate? [no]:  yes
Certificate was added to keystore

```


```

keytool -keystore client.truststore.jks -alias CARoot -import -file ca-cert

```


```

# password4

keytool -keystore client.truststore.jks -alias CARoot -import -file ca-cert
Enter keystore password:
Re-enter new password:
Owner: CN=10.100.65.93, OU=IT, O=MyCompany, L=Shanghai, ST=Shanghai, C=CN
Issuer: CN=10.100.65.93, OU=IT, O=MyCompany, L=Shanghai, ST=Shanghai, C=CN
Serial number: c27acf3a4a31a6b1
Valid from: Mon Jul 31 15:50:19 CST 2023 until: Tue Jul 30 15:50:19 CST 2024
Certificate fingerprints:
         SHA1: 59:2A:98:7D:87:25:57:08:A3:B9:03:A4:C5:2F:60:8B:08:48:63:BA
         SHA256: D6:A6:40:C0:A8:49:0B:AB:4A:79:72:3D:E2:7B:CD:77:BD:F4:61:55:62:63:76:2E:8F:2F:EE:88:F7:73:C7:4B
Signature algorithm name: SHA256withRSA
Subject Public Key Algorithm: 2048-bit RSA key
Version: 1
Trust this certificate? [no]:  yes
Certificate was added to keystore
```


```
# password1

 keytool -keystore server.keystore.jks -alias localhost -certreq -file cert-file
Enter keystore password:

```


```

openssl x509 -req -CA ca-cert -CAkey ca-key -in cert-file -out cert-signed -days 365 -CAcreateserial -passin pass:password2

```


```
# password2

openssl x509 -req -CA ca-cert -CAkey ca-key -in cert-file -out cert-signed -days 365 -CAcreateserial -passin pass:password2
Signature ok
subject=/C=CN/ST=Shanghai/L=Shanghai/O=MyCompany/OU=IT/CN=10.100.65.93
Getting CA Private Key

```


```
# passowrd1

keytool -keystore server.keystore.jks -alias CARoot -import -file ca-cert
Enter keystore password:
Owner: CN=10.100.65.93, OU=IT, O=MyCompany, L=Shanghai, ST=Shanghai, C=CN
Issuer: CN=10.100.65.93, OU=IT, O=MyCompany, L=Shanghai, ST=Shanghai, C=CN
Serial number: c27acf3a4a31a6b1
Valid from: Mon Jul 31 15:50:19 CST 2023 until: Tue Jul 30 15:50:19 CST 2024
Certificate fingerprints:
         SHA1: 59:2A:98:7D:87:25:57:08:A3:B9:03:A4:C5:2F:60:8B:08:48:63:BA
         SHA256: D6:A6:40:C0:A8:49:0B:AB:4A:79:72:3D:E2:7B:CD:77:BD:F4:61:55:62:63:76:2E:8F:2F:EE:88:F7:73:C7:4B
Signature algorithm name: SHA256withRSA
Subject Public Key Algorithm: 2048-bit RSA key
Version: 1
Trust this certificate? [no]:  yes
Certificate was added to keystore

```


```

keytool -keystore server.keystore.jks -alias localhost -import -file cert-signed
Enter keystore password:
Certificate reply was installed in keystore

```



```

bin/kafka-topics.sh --create --topic test --bootstrap-server 10.100.65.93:9092 --command-config config/client_ssl.properties


```
