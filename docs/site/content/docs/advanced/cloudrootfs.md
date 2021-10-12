+++
title = "CloudRootfs"
description = "Define your own CloudRootfs."
date = 2021-05-01T19:30:00+00:00
updated = 2021-05-01T19:30:00+00:00
draft = false
weight = 30
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = "CloudRootfs will package all the dependencies refers to the kubernetes cluster requirements."
toc = true
top = false
+++

# What is CloudRootfs

All the files witch run a kubernetes cluster needs.

Contains:

* Bin files, like docker containerd crictl kubeadm kubectl...
* Config files, like kubelet systemd config, docker systemd config, docker daemon.json...
* Registry docker image
* Some Metadata, like Kubernetes version.
* Registry files, contains all the docker image, like kubernetes core components docker images...
* Scripts, some shell script using to install docker and kubelet... sealer will call init.sh and clean.sh.
* Other static files

```yaml
.
├── bin
│   ├── conntrack
│   ├── containerd-rootless-setuptool.sh
│   ├── containerd-rootless.sh
│   ├── crictl
│   ├── kubeadm
│   ├── kubectl
│   ├── kubelet
│   ├── nerdctl
│   └── seautil
├── cri
│   ├── containerd
│   ├── containerd-shim
│   ├── containerd-shim-runc-v2
│   ├── ctr
│   ├── docker
│   ├── dockerd
│   ├── docker-init
│   ├── docker-proxy
│   ├── rootlesskit
│   ├── rootlesskit-docker-proxy
│   ├── runc
│   └── vpnkit
├── etc
│   ├── 10-kubeadm.conf
│   ├── Clusterfile  # image default Clusterfile
│   ├── daemon.json
│   ├── docker.service
│   ├── kubeadm-config.yaml
│   └── kubelet.service
├── images
│   └── registry.tar  # registry docker image, will load this image and run a local registry in cluster
├── Kubefile
├── Metadata
├── README.md
├── registry # will mount this dir to local registry
│   └── docker
│       └── registry
├── scripts
│   ├── clean.sh
│   ├── docker.sh
│   ├── init-kube.sh
│   ├── init-registry.sh
│   ├── init.sh
│   └── kubelet-pre-start.sh
└── statics # yaml files, sealer will render values in those files
    └── audit-policy.yml
```

# How can I get CloudRootfs

1. Pull a BaseImage `sealer pull kubernetes:v1.19.9`
2. Get into the BaseImage Layer `cd /var/lib/sealer/data/overlay2/my-cluster/mount/`
3. Cd the BaseImage hash layer `ls && cd [layer-hash-id]`

You will found the CloudRootfs layer.

# Build your own BaseImage

You can edit any files in CloudRootfs you want, for example you want define your own docker daemon.josn, just edit it and build a new CloudImage.

```shell script
FROM scratch
COPY . .
```

```shell script
sealer build -t user-defined-kubernetes:v1.19.9 .
```

Then you can use this image as a BaseImage.

# OverWrite CloudRootfs files

Sometimes you don't want to care about the CloudRootfs context, but need custom some config.

You can using `kubernetes:v1.19.9` as BaseImage, and use your own config file to overwrite the default file in CloudRootfs.

For example: daemon.json is your docker engine config, using it to overwrite default config:

```shell script
FROM kubernetes:v1.19.9
COPY daemon.json etc/
```

```shell script
sealer build -t user-defined-kubernetes:v1.19.9 .
```