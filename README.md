Ansible Tor
===========

This is an Ansible role for use with Tor - https://www.torproject.org/

I hope that relay operators will find this useful for deploying
and maintaining large numbers of Tor relays and bridges with
finesse, concurrency and idempotency!

This ansible role can help you reduce the complexity to a single
command for deploying one or more tor relays or tor bridges with or
without obfsproxy.

Here I assume the user has setup their ansible project directory according to the best practices
directory-layout specified here:

https://docs.ansible.com/playbooks_best_practices.html#directory-layout

...and simply run a command like this: `ansible-playbook -i production tor-relays.yml`

In this case I'm running the `tor-relays.yml` playbook against
the "production" inventory file. This tor-relays playbook might
specify a host group called tor-relays... which is defined in the
inventory file. I could have many other host groups defined in the
inventory as well such as: tor-exit-relays, tor-bridges,
tor-bananaphone-bridges, tor-hidden-tahoe-storage-nodes etc.

For example configurations, checkout the inventory variables under [examples_host_vars/][].

Requirements
------------

Works on Debian and Ubuntu.
I've tried it... so I know. =-)

FIXME: Also refer to Travis build.


## Example configurations

#### Tor bridge using obfs4 pluggable transport

```YAML
tor_BridgeRelay: 1,
tor_PublishServerDescriptor: "bridge",
tor_ExtORPort: "auto",
tor_ORPort: 9001,
tor_ServerTransportPlugin: "obfs4 exec /usr/bin/obfs4proxy",
tor_ExitPolicy: "reject *:*",
tor_obfs4proxy_enabled: True,
```

#### Tor bridge using obfsproxy together with scramblesuit

Configures a scramblesuit tor bridge using the
latest obfsproxy available to pip (installs into a python virtualenv).

http://www.cs.kau.se/philwint/scramblesuit/

```YAML
tor_BridgeRelay: 1,
tor_PublishServerDescriptor: "bridge",
tor_obfsproxy_home: "/home/ansible",
tor_ORPort: 9001,
tor_ServerTransportPlugin: "scramblesuit exec {{ tor_obfsproxy_home }}/{{ tor_obfsproxy_virtenv }}/bin/obfsproxy --log-min-severity=info --log-file=/var/log/tor/obfsproxy.log managed",
tor_ServerTransportListenAddr: "scramblesuit 0.0.0.0:4703",
tor_ExitPolicy: "reject *:*",
```

#### Tor bridge using bananaphone pluggable transport

Configures a tor bridge
with an obfsproxy installed from my git repository so that
the bananaphone pluggable transport is available (it has not been
merged upstream).

Bananaphone provides tor over markov chains!
If you have sensitive or interesting documents then please consider
operating a bananaphone bridge utilizing these text corpuses.
Read about the bananaphone pluggable transport for tor.
https://bananaphone.readthedocs.org/

```YAML
tor_BridgeRelay: 1,
tor_PublishServerDescriptor: "bridge",
tor_obfsproxy_home: "/home/ansible",
tor_ORPort: 9001,
tor_obfsproxy_git_url: "git+https://github.com/david415/obfsproxy.git",
tor_ServerTransportPlugin: "bananaphone exec {{ tor_obfsproxy_home }}/{{ tor_obfsproxy_virtenv }}/bin/obfsproxy --log-min-severity=info --log-file=/var/log/tor/obfsproxy.log managed",
tor_ServerTransportOptions: "bananaphone corpus=/usr/share/dict/words encodingSpec=words,sha1,4 modelName=markov order=1",
tor_ServerTransportListenAddr: "bananaphone 0.0.0.0:4703",
tor_ExitPolicy: "reject *:*",
```

#### Tor hidden service for SSH

This demonstrates waiting for the hidden services
to be created by awaiting the existence of the tor hidden service
hostname files. This happens when the role variable
`tor_wait_for_hidden_services` is set to yes.

This feature could be useful when configuring other services that
depend on knowing the hidden service's onion address such as
my [`david415.tahoe-lafs`][david415.tahoe-lafs] role:

Read about Tahoe-LAFS here:
https://tahoe-lafs.org/trac/tahoe-lafs

Read about tor hidden services here:
https://www.torproject.org/docs/tor-hidden-service.html.en

```YAML
relay_hidden_services_parent_dir: "/var/lib/tor/services"
relay_hidden_services:
  - dir: "hidden_ssh",
    ports:
      - virtport: "22"
        target: "localhost:22"

tor_ExitPolicy: "reject *:*",
tor_hidden_services: "{{ relay_hidden_services }}",
tor_hidden_services_parent_dir: "{{ relay_hidden_services_parent_dir }}",
tor_wait_for_hidden_services: yes,
```

### Multi-tor-instance playbook


```YAML
---
- hosts: tor-relays
  vars:
    tor_ExitPolicy: "reject *:*",

  roles:
    - { role: david415.ansible-tor,
        sudo: yes
      }
```

A simple playbook like this can be used to deploy many instances of
tor on many servers. You can configure multiple tor instances using
the host_vars file for each host.

Here's what an example host_vars file.

host_vars/tor_relay1.example.com:
```YAML
tor_Nickname: "ScratchMaster"
tor_instances:
  - name: "relay1"
    tor_ORPort: ["192.168.1.1:9002"]
    tor_SOCKSPort: ["8041"]
  - name: "relay2"
    tor_ORPort: ["192.168.1.2:9002"]
    tor_SOCKSPort: ["8042"]
  - name: "relay3"
    tor_ORPort: ["192.168.1.3:9002"]
    tor_SOCKSPort: ["8043"]
```

In the above example playbook, all the role variables get applied to
all tor instances. If you want to control the role variables for a
specific host then you must use that host's host_vars file.

Note: When this role is used in "multi-tor process mode" meaning
that if the `tor_instances` variable is defined then the torrc template will set
reasonable defaults for these torrc options: User, PidFile, Log and DataDirectory.

Tor configuration - torrc
-------------------------

torrc may have options set from host_vars/group_vars and
also set from role variables.

The host_vars can set arbitrary torrc configuration options.
Refer to the templates/torrc.j2 for details.

The host_vars variable names must begin with "tor\_";
Here's an example setting "Nickname":

```YAML
tor_Nickname: "OnionRobot"
```

The dictionary value is a list because in some cases you may want to
specify multiple lines in the torrc that begin with the dictionary key.

## Operational security

When you run an Tor relay, please secure your server and administration system as good as possible.

Be sure to read thought the wiki Artikel [how to Run a Secure Tor Server][].

Additionally, there are a few roles which can help you in that regard:

* [`david415.tlsdate`][david415.tlsdate]

  Can provide accurate time information over a secure (side) channel.
  FIXME: Add reference why this improves tor security.

* [`david415.openssh-hardened`][david415.openssh-hardened] or [`hardening.ssh-hardening`][hardening.ssh-hardening] or [`debops.sshd`][debops.sshd]

  Furthermore it is also a good idea to have a hardened openssh-server
  setup that supports the new ed25515 key exchange and
  the new polychacha1305 DJB-inspired crypto transport.

* [`hardening.os-hardening`][hardening.os-hardening]

* [`ypid.apparmor`][ypid.apparmor]

* [`debops.ferm`][debops.ferm]

License
-------

MIT


Feature requests and bug-reports welcome!
-----------------------------------------

https://github.com/david415/ansible-tor/issues


[david415.tlsdate]: https://github.com/david415/ansible-tlsdate
[david415.openssh-hardened]: https://github.com/david415/ansible-openssh-hardened
[hardening.ssh-hardening]: https://github.com/hardening-io/ansible-ssh-hardening
[debops.sshd]: https://github.com/debops/ansible-sshd
[hardening.os-hardening]: https://github.com/hardening-io/ansible-os-hardening
[ypid.apparmor]: https://github.com/ypid/ansible-apparmor
[debops.ferm]: https://github.com/debops/ansible-ferm
[david415.tahoe-lafs]: https://github.com/david415/ansible-tahoe-lafs

[How to Run a Secure Tor Server]: https://trac.torproject.org/projects/tor/wiki/doc/OperationalSecurity
[examples_host_vars/]: https://github.com/david415/ansible-tor
