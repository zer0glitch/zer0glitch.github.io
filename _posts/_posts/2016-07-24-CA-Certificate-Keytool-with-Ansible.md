---
layout: post
title: "OpenSSL CA, Certificate and Keytool Management with Ansible"
description: Managing a CA and certificates with Ansible
---



***Introduction***

Ansible is a fast growing product for provisioning of compute instances, and software on the computer instances.  A simple way way of managing certificates and users is through the use of System Integrity Management Platform (SIMP).  Unfortunately, SIMP uses puppet to manage certificates and servers, and this is a large commitment to manage a cluster for development purposes.  

The OpenSSL module can solve the issue of certificate management in Ansible.  The module, or extension of Ansible, allows for complete CA, Certificate, and Keystore management.  The system will work existing CAs or create a custom CA for internal development.  

Ansible-OpenSSL will create certificates that are compatible with Apache httpd, Apache Tomcat, Jboss, IIS, etc.  This includes server and client certificates for accessing the system.

***Installing the OpenSSL Module***

```bash
git clone git@github.com:rclayton-the-terrible/Ansible-OpenSSL.git
```

Follow the installation instructions from the github [project](https://github.com/rclayton-the-terrible/Ansible-OpenSSL).

***Creating a CA***

The essential of certificate management is the Certificate Authority.  The Certificate Authority is responsible for being the master trust for a certificate chain.  As part of this responsibility it will sign the client and server certificate requests.

For our purproses we will create a CA if you already have a CA, then you can skip this step.  Our certitifcate directory will be `/etc/certs` the subject `/DC=com/DC=example/CN=CA`.  The format is typically the reverse of your DNS name.  To maintain uniquness in the system you can add additional information to the CN if you are planning on running multiple developemnt enviornments.

```yaml
- name: Setup a CA
  ca: certdir="/etc/certs" subj="/DC=com/DC=example/CN=CA/"
```

You should now have a basic structure for your CA created under /etc/certs.  There are many suplimental files and directories that are created as part of this process, so do not delete from this directory.  It is safe to remove files in the server, keystores, client, and trustore directories, but not the directories themselves.

***Creating a server certificate***

After setting up our CA, we will now use the certificate module to create our server certificate to secure a web server.  

```yaml
- name: Create a Server Cert
  certificate: cadir="/etc/certs" certname="www.example.com" subj="/DC=com/DC=example/CN=www/" p12password="changeit" subjectAltNames="DNS:client,DNS:www.example.com,IP:192.168.2.2"
```

The files created for the certificate are stored in `/etc/certs/server/www.example.com`

The task will create the following files:

```
openssl.cnf - The definition of how the certificate request is generated
test.openampere.com.req.pem - The certificate request to be signed by the CA
test.openampere.com.cert.pem.pub - The signed certificate file
test.openampere.com.key.pem - The certificates key file
test.openampere.com.keycert.pem - The certificate and key file in one file
test.openampere.com.keycert.p12 - The PKCS12 file used to import into IIS or a java keystore
```

The next step is to install the certificate on the server of choice.  In our case, we are using the JBoss EAP Server as our web application server.  To accomplish this we first must create a truststore for the server to read.

***Creating a truststore***

By default the truststore will contain our default CA, and it should trust the itself for communications that are protected by client certificates.

```yaml
- name: Create a java server trustore and trust the server hosts
  keytool: cadir="/etc/certs" certname="www.example.com" store_password='changeit' hosts_to_trust="www.example.com"
```

The result truststore is stored in `/etc/certs/truststore`

The following file will be created:

```yaml
www.example.com.trust.jks
```

***Creating a keystore***

The keystore will store the server certificate key for Jboss.  You must use the keystore util to create the keystore used by jboss.

```yaml
name: Create a java server keystore 
  keytool: cadir="/etc/certs" certname="www.example.com" store_password='changeit'  certtype="keystore" src_password='changeit'
```

This will creating the following files:

```
/etc/certs/keystores/www.example.com.keystore.jks
```

The foundation is now set for a simple example of securing a single server.  The certificates generated will secure several different types of servers.

Part Two of this article will examine a certificate system.  [Part 2](CA-Certificate-Keytool-with-Ansible-Part2)

