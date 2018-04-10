---
layout: post
section-type: post
category: tech
tags: [ 'Openshift,3.7' ]
title: "Notes for Openshift Disconnected with Bring your own Registry"
description: Notes for Openshift Disconnected with Bring your own Registry
---

**Notes for Openshift Disconnected with Bring your own Registry**

*Inventory Example*
The following inventory changes are needed for a disconnected install.  These items were collected from v1.uncontained.io, and official openshift documentation.

```
[OSEv3:children]
nodes
masters
nfs
etcd
[OSEv3:vars]
openshift_master_cluster_public_hostname=node12.dev9.acs.sd.spawar.navy.mil
openshift_master_default_subdomain=apps.dev9.acs.sd.spawar.navy.mil
ansible_ssh_user=root
openshift_master_cluster_hostname=node12.dev9.acs.sd.spawar.navy.mil
openshift_override_hostname_check=true
deployment_type=openshift-enterprise
#### Additional After default config ###
openshift_disable_check=disk_availability,docker_storage
openshift_docker_additional_registries=c3po.sd.spawar.navy.mil:5000
openshift_docker_insecure_registries=c3po.sd.spawar.navy.mil:5000
openshift_docker_blocked_registries=registry.access.redhat.com,docker.io
oreg_url=c3po.sd.spawar.navy.mil:5000/openshift3/ose-${component}:${version}
openshift_examples_modify_imagestreams=true
openshift_disable_check=disk_availability,docker_storage
openshift_release="3.7"
openshift_image_tag="v3.7.42"
openshift_portal_net=172.30.0.0/16
osm_cluster_network_cidr=10.128.0.0/14
osm_host_subnet_length=9
openshift_master_cluster_method=native
openshift_hosted_registry_storage_kind=nfs
openshift_hosted_registry_storage_access_modes=['ReadWriteMany']
openshift_hosted_registry_storage_host=nfs.myorg.com
openshift_hosted_registry_storage_nfs_directory=/exports
openshift_hosted_registry_storage_volume_name=registry
openshift_hosted_registry_storage_volume_size=100Gi
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider', 'filename': '/etc/origin/master/htpasswd'}]
openshift_master_htpasswd_users={'admin': '$apr1$we44dyLk$novkgu.aJNm4HR.TdTzOC0', 'developer': '$apr1$we44dyLk$novkgu.aJNm4HR.TdTzOC0'}
[nodes]
node12.dev9.acs.sd.spawar.navy.mil  openshift_public_ip=10.10.9.112 openshift_ip=10.10.9.112 openshift_public_hostname=node12.dev9.acs.sd.spawar.navy.mil openshift_hostname=node12.dev9.acs.sd.spawar.navy.mil openshift_hosted_registry_storage_kind=nfs openshift_node_labels="{'region': 'infra'}" openshift_schedulable=True ansible_connection=ssh
node13.dev9.acs.sd.spawar.navy.mil  openshift_public_ip=10.10.9.113 openshift_ip=10.10.9.113 openshift_public_hostname=node13.dev9.acs.sd.spawar.navy.mil openshift_hostname=node13.dev9.acs.sd.spawar.navy.mil openshift_hosted_registry_storage_kind=nfs openshift_node_labels="{'region': 'primary', 'zone': 'default'}" openshift_schedulable=True ansible_connection=ssh
node14.dev9.acs.sd.spawar.navy.mil  openshift_public_ip=10.10.9.114 openshift_ip=10.10.9.114 openshift_public_hostname=node14.dev9.acs.sd.spawar.navy.mil openshift_hostname=node14.dev9.acs.sd.spawar.navy.mil openshift_hosted_registry_storage_kind=nfs openshift_node_labels="{'region': 'primary', 'zone': 'default'}" openshift_schedulable=True ansible_connection=ssh
node15.dev9.acs.sd.spawar.navy.mil  openshift_public_ip=10.10.9.115 openshift_ip=10.10.9.115 openshift_public_hostname=node15.dev9.acs.sd.spawar.navy.mil openshift_hostname=node15.dev9.acs.sd.spawar.navy.mil openshift_hosted_registry_storage_kind=nfs openshift_node_labels="{'region': 'primary', 'zone': 'default'}" openshift_schedulable=True ansible_connection=ssh
[masters]
node12.dev9.acs.sd.spawar.navy.mil  openshift_public_ip=10.10.9.112 openshift_ip=10.10.9.112 openshift_public_hostname=node12.dev9.acs.sd.spawar.navy.mil openshift_hostname=node12.dev9.acs.sd.spawar.navy.mil openshift_hosted_registry_storage_kind=nfs ansible_connection=ssh
[nfs]
node12.dev9.acs.sd.spawar.navy.mil  openshift_public_ip=10.10.9.112 openshift_ip=10.10.9.112 openshift_public_hostname=node12.dev9.acs.sd.spawar.navy.mil openshift_hostname=node12.dev9.acs.sd.spawar.navy.mil openshift_hosted_registry_storage_kind=nfs ansible_connection=ssh
[etcd]
node12.dev9.acs.sd.spawar.navy.mil  openshift_public_ip=10.10.9.112 openshift_ip=10.10.9.112 openshift_public_hostname=node12.dev9.acs.sd.spawar.navy.mil openshift_hostname=node12.dev9.acs.sd.spawar.navy.mil openshift_hosted_registry_storage_kind=nfs ansible_connection=ssh
```

*Setting up the registiry*
The openshift-toolkit provides some useful utilities for starting the installation of the BYOR, but openshift can and willr equire a different set of images based upon your the version of the RPMs you are using.  In the middle of the install images switched from v3.7.23 to v3.7.42 which did not work with the base script.

```
wget https://raw.githubusercontent.com/redhat-cop/openshift-toolkit/master/disconnected_registry/docker-registry-sync.py
curl -O https://raw.githubusercontent.com/redhat-cop/openshift-toolkit/master/disconnected_registry/docker_tags.json
chmod +x docker-registry-sync.py
./docker-registry-sync.py --from=registry.access.redhat.com --to=c3po.sd.spawar.navy.mil:5000 --file=./docker_tags.json --openshift-version=3.7 
docker pull registry.access.redhat.com/openshift3/ose-web-console:v3.9
docker tag registry.access.redhat.com/openshift3/ose-web-console:v3.9 c3po.sd.spawar.navy.mil:5000/openshift3/ose-web-console:v3.9
docker push c3po.sd.spawar.navy.mil:5000/openshift3/ose-web-console:v3.9
docker pull registry.access.redhat.com/openshift3/ose-haproxy-router:v3.7.23
docker tag registry.access.redhat.com/openshift3/ose-haproxy-router:v3.7.23 c3po.sd.spawar.navy.mil:5000/openshift3/ose-haproxy-router:v3.7.23
docker push c3po.sd.spawar.navy.mil:5000/openshift3/ose-haproxy-router:v3.7.23
docker pull registry.access.redhat.com/openshift3/ose-deployer:v3.7.23
docker tag registry.access.redhat.com/openshift3/ose-deployer:v3.7.23 c3po.sd.spawar.navy.mil:5000/openshift3/ose-deployer:v3.7.23
docker push c3po.sd.spawar.navy.mil:5000/openshift3/ose-deployer:v3.7.23
docker pull registry.access.redhat.com/openshift3/registry-console:v3.7
docker tag registry.access.redhat.com/openshift3/registry-console:v3.7 c3po.sd.spawar.navy.mil:5000/openshift3/registry-console:v3.7
docker push c3po.sd.spawar.navy.mil:5000/openshift3/registry-console:v3.7

```
