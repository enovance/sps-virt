[![Build Status](https://travis-ci.org/enovance/infra-virt.svg?branch=master)](https://travis-ci.org/enovance/infra-virt)

testou

# Virtualization


The virtualization directory contains the tools which aims to standardize the way
we test an architecture in a virtual environment.

## Prerequisites

On the local machine:

You must install libvirt-python first.

```sh
pip install -r requirements.txt
```

On the Hypervisor:

We use a Fedora 21 with the following extra packages:

- libvirt
- mtools
- qemu-kvm

You will need to disable firewalld and enable libvirt:

```sh
sudo systemctl enable libvirtd
sudo systemctl start libvirtd

sudo service iptables save
sudo systemctl disable firewalld
sudo systemctl enable iptables
sudo systemctl stop firewalld
sudo systemctl start iptables
```

### User account

At this point, you must create the user on the Hypervisor this way:

* sudo with no password. e.g: `joe     ALL=(ALL)       NOPASSWD: ALL`
* ssh-key with no password `ssh-keygen`
* ssh-key registred in the /root/.ssh/authorized_keys file

You must be able to run the following command with no password:

```sh
$ ssh root@localhost uname
$ sudo uname
```

## Collector

```sh
$ ./collector.py --help
usage: collector.py [-h] [--config-dir CONFIG_DIR] [--output-dir OUTPUT_DIR]
                    --sps-version SPS_VERSION [--qcow]

Collect architecture information from the edeploy directory as generated by
config-tools/download.sh.

optional arguments:
  -h, --help            show this help message and exit
  --config-dir CONFIG_DIR
                        The config directory absolute path.
  --output-dir OUTPUT_DIR
                        The output directory of the virtual configuration.
  --sps-version SPS_VERSION
                        The SpinalStack version.
  --qcow                Boot on qcow image.
```

### Virtualizor

```sh
usage: virtualizor.py [-h] [--replace] [--pub-key-file PUB_KEY_FILE]
                      [--prefix PREFIX] [--public_network PUBLIC_NETWORK]
                      input_file target_host

Deploy a virtual infrastructure.

positional arguments:
  input_file            the YAML input file, as generated by collector.py.
  target_host           the name of the libvirt server. The local user must be
                        able to connect to the root account with no password
                        authentification.

optional arguments:
  -h, --help            show this help message and exit
  --replace             existing conflicting resources will be remove
                        recreated.
  --pub-key-file PUB_KEY_FILE
                        the path to the SSH public key file that must be
                        injected in the install-server root and jenkins
                        account
  --prefix PREFIX       optional prefix to put in the machine and network to
                        avoid conflict with resources create by another
                        virtualizor instance. Thanks to this parameter, the
                        user can run as virtualizor as needed on the same
                        machine.
  --public_network PUBLIC_NETWORK
                        allow the user to pass the name of a libvirt NATed
                        network that will be used as a public network for the
                        install-server. This public network will by attached
                        to eth1 interface and IP address is associated using
                        the DHCP.
```

### Example

```sh
$ cd ~
$ git clone https://github.com/enovance/config-tools.git
$ cd config-tools/
$ git checkout I.1.3
$ ./download.sh I.1.3.0 deployment-3nodes-D7.yml version=D7-I.1.3.0
```

It generates a directory 'top/' in the current directory.

```sh
$ cd ~
$ git clone https://github.com/enovance/virt-infra.git
$ cd virt-infra/
$ ./collector.py --config-dir ~/config-tools/top/etc --sps-version D7-I.1.3.0
Virtual platform generated successfully at 'virt_platform.yml' !
```

It will generate a file 'virt_platform.yml' which describe the corresponding virtual
platform. You may take a look at a sample in the virtualization directory.

```sh
$ cd ~/virt-infra/
$ ./virtualizor.py virt_platform.yml my-hypervisor-node --replace --pub-key-file ~/.ssh/boa.pub
```

### virtualize.sh

`virtualize.sh` is a script built on top of `virtualizor.py` to play SpinalStack deployment and upgrade.

```sh
$ ./virtualize.sh --help
usage: virtualize.sh [OPTION] workdir1 workdir2 etc
Collect architecture information from the edeploy directory as generated
by config-tools/download.sh.

optinal arguments:
  -H|--hypervisor=name: change the hypervisor name, default ()
arguments:
virtualize.sh will use the first argument as the location of the
SpinalStack environment to deploy. It will then upgrade the newly deployed
SpinalStack to the following environment directory.

For example: ./virtualize.sh I.1.2.1/
will deploy environment from the I.1.2.1/ directory.
```
