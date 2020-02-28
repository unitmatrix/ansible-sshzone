# ansible-sshzone

[![GitHub release](https://img.shields.io/github/release/unitmatrix/ansible-sshzone.svg?style=flat-square)](https://github.com/unitmatrix/ansible-sshzone/releases)

An Ansible connection plugin for remotely provisioning Solaris zones separately from their zone host.

[Solaris zones](https://docs.oracle.com/cd/E18440_01/doc.111/e18415/chapter_zones.htm) are Oracle's way of isolating workloads inside a running Solaris OS - similar to [LXC](https://linuxcontainers.org/lxc/introduction/) and [Docker](https://www.docker.com/resources/what-container) containers in Linux, [jails](https://www.freebsd.org/doc/handbook/jails.html), [iocages](https://github.com/iocage/iocage) in FreeBSD, [WPARs](https://www.ibm.com/support/pages/aix-wpars-how) in AIX and other workload partitioning tools. Sometimes a zone may be isolated to the extent where it has limited network connectivity. In this case, this plugin can be used to access the zone through its host (the global zone).

This works by SSHing to the zone host using the standard Ansible SSH connection, moving any files into the zone directory, and using zlogin to execute commands in the scope of the zone.

The project is inspired by [ansible-sshjail](https://github.com/austinhyde/ansible-sshjail) by [Austin Hyde](https://github.com/austinhyde) with applied changes and modifications specific to Solaris OS and Solaris zones.

# Requirements

Control node (your workstation or deployment server):

* Ansible 2.0 RC3+
* Python 2.7

Zonehost:

* Solaris 10
* At least one configured zone
* Python 2.7
* SSH
* sudo

Target zone:

* Python 2.7

# Installation

This is a "Connection Type Plugin", as outlined in the [Ansible docs](http://docs.ansible.com/developing_plugins.html#connection-type-plugins).

To install sshjail:

1. Clone this repo.
2. Copy or link `sshzone.py` to one of the supported locations:
  * `/usr/share/ansible/plugins/connection_plugins/sshzone.py`
  * `path/to/your/toplevelplaybook/connection_plugins/sshzone.py`

# Usage

Using sshzone, each zone is its own inventory host, identified with `ansible_zone_name` variable. You must also specify `ansible_connection=sshzone` and `ansible_host`.

* `ansible_zone_name` is the name of the zone.
* `ansible_host` is the hostname or IP address of the zonehost.

If your python binary is not in the default location, make sure to specify this with the `ansible_python_interpreter` variable!

The following inventory entries are examples of using sshzone:

```
# bare minimum inventory example
my-solaris-zone1 ansible_connection=sshzone ansible_zone_name=my-solaris-zone1 ansible_host=1.2.3.4

# sample custom inventory example
my-solaris-zone1 ansible_connection=sshzone ansible_host=1.2.3.4 ansible_ssh_port=2222 ansible_python_interpreter=/usr/local/bin/python2.7 ansible_ssh_user=vagrant
```

Adding these hosts dynamically, like after freshly creating them via Ansible, or by iterating over `zoneadm` output, can be done via the [built-in `add_host` module](http://docs.ansible.com/add_host_module.html):

```YAML
- name: add my-solaris-zone1 to ansible inventory
  add_host: name=my-solaris-zone1 groups=zones
            ansible_ssh_host=my-solaris-zone1@
            ansible_ssh_host={{ansible_host}}
            ansible_ssh_port={{ansible_ssh_port}}
            ansible_python_interpreter=/usr/local/bin/python2.7
            ansible_connection=sshzone
```

## A note about privileges

By default in Solaris, only root can enter zones. This means that when invoking `ansible` or `ansible-playbook`,
you need to specify `--become`, and in a playbook, use `become: yes`/`become_method: sudo`. If sudo requires a password, you'll need `--ask-become-pass` as well.

This means any commands executed by sshzone roughly translate to `sudo zlogin $zoneName $command`.

Because of limitations of Ansible, this plugin cannot really do things like `sudo zlogin sudo -u myuser $command`

# Contributing

Let me know if you have any difficulties using this, by creating an issue.

Pull requests are always welcome! I'll try to get them reviewed in a timely manner.
