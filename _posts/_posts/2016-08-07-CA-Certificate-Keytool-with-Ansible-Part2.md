---
layout: post
title: "OpenSSL CA, Certificate and Keytool Management with Ansible Part 2"
description: Managing a CA and certificates with Ansible
---

***Introduction***

In the previous post we examined configuring a single JBoss server with a keystore.  In this post we will expand with a full example in Ansible on setting up the following server configuration:

> * Server 1: Httpd, Proxy, and OpenLDAP
> * Server 2: JBoss EAP 7
> * Server 3: JBoss EAP 7

There are several ways to get the project up and running.  For purposes of this example, there is a vagrant file, and simple playbook to install and setup the ansible provisioner from vagrant.  This can be downloaded from [file](here)

The vagrant up command will start the 3 servers install and start the services on the servers.

Once the servers are up executing the following ansible playbook will configure, the host services.

```yaml
---
# The local server will act as our CA server
- hosts: localhost
  vars:
    servers:
    - www
    - app1
    - app2
    cert_dir: ./exampleca
    passwd: changeit
    ca_base: "/DC=com/DC=example/"
  tasks:

  # Configure the CA
  - name: Setup a CA
    ca: 
    certdir="{{ cert_dir }}" 
    subj="{{ ca_base }}/CN=CA/"

  - name: generate server certificates for all hosts
    certificate:  
    cadir="{{ cert_dir }}" 
    certname="{{ item }}.{{ ansible_domain }}" 
    subj="{{ ca_base }}/CN={{ item }}/" 
    p12password="{{passwd}}" 
    subjectAltNames="DNS:{{ item }},DNS:{{ item }}.{{ ansible_domain }}"
    with_items: "{{ servers }}"

    
  - name: Create a java server trustore and trust the server hosts
    keytool: 
    cadir="{{ cert_dir }}" 
    certname="{{ item }}.{{ ansible_domain }}" 
    store_password='{{ passwd }}' 
    hosts_to_trust="{{ item }}.{{ ansible_domain }}"
    with_items: "{{ groups['app_servers'] }}"

  - name: Create a java server keystore 
    keytool: 
    cadir="{{ cert_dir }}" 
    certname="{{ item }}.{{ ansible_domain }}" 
    store_password='{{ passwd }}'  
    certtype="keystore" 
    src_password='{{ passwd }}'
    with_items: "{{ groups['app_servers'] }}"

```
