---
layout: post
section-type: post
category: tech
tags: [ 'bug' ]
layout: post
title: "Packer QEMU Error"
description: working with packer and qemu
---

**Issue**

Running:
```
packer build  packer-centos7.json 
```

Produces

```
centos7-base output will be in this color.

Build 'centos7-base' errored: Failed creating Qemu driver: exec: "qemu-system-x86_64": executable file not found in $PATH

==> Some builds didn't complete successfully and had errors:
--> centos7-base: Failed creating Qemu driver: exec: "qemu-system-x86_64": executable file not found in $PATH

==> Builds finished but no artifacts were created.
```

Fix:

```
ln -s /usr/libexec/qemu-kvm /usr/local/bin/qemu-system-x86_64
```
