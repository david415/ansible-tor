Ansible Tor
===========

This is an Ansible role for use with Tor - https://www.torproject.org/

I hope that relay operators will find this useful for deploying
and maintaining large numbers of Tor relays and bridges with
finesse, concurrency and idempotency!

Feature requests encouraged!

Future code commits coming soon... including
task lists to install and fully configure tor bridges
with the latest obfsproxy Pluggable Transports (such as
Scramblesuit and obfs3) and example playbooks demonastrating
various maintenance procedures such as Tor bridge
IP address reassignment and  building a local inventory
of hidden ssh service onion addresses.



Requirements
------------

Works on Debian and Ubuntu.

Example Tor Bridge Playbook
---------------------------

```yml
---

- hosts: tor-bridges
  user: ansible
  connection: ssh
  vars:
    relay_hidden_service_name: "hidden_ssh"
    hidden_ssh_virtport: 6669
    hidden_ssh_target: "127.0.0.1:22"
    relay_hidden_services_parent_dir: "/var/lib/tor/services"
    relay_hidden_services: [ { dir: "{{ relay_hidden_service_name }}",
                               ports: [ { virtport: "{{ hidden_ssh_virtport }}",
                                 target: "{{ hidden_ssh_target }}" } ] }
    ]
  roles:
    - { role: ansible-role-firewall,
        firewall_allowed_tcp_ports: [ 22, 4703 ],
        sudo: yes
      }
    - { role: ansible-wheezy-common,
        sudo: yes
      }
    - { role: Ansibles.openssh,
        sudo: yes
      }
    - { role: david415.ansible-tor,
        ansible_distribution_release: "wheezy",
        tor_BridgeRelay: 1,
        tor_PublishServerDescriptor: "bridge",
        tor_obfsproxy_home: "/home/ansible",
        tor_obfsproxy_virtenv: "virtenv_obfsproxy",
        tor_ORPort: 9001,
        tor_ServerTransportPlugin: "scramblesuit exec {{ tor_obfsproxy_home }}/{{ tor_obfsproxy_virtenv }}/bin/obfsproxy managed",
        tor_ServerTransportListenAddr: "scramblesuit 0.0.0.0:4703",
        tor_obfsproxy_virtenv_version: "virtualenv-1.11.4",
        tor_virtenv_tarball_url: "https://pypi.python.org/packages/source/v/virtualenv/virtualenv-1.11.4.tar.gz",
        tor_virtenv_tarball: "virtualenv-1.11.4.tar.gz",
        tor_ExitPolicy: "reject *:*",
        tor_hidden_services: "{{ relay_hidden_services }}",
        tor_hidden_services_parent_dir: "{{ relay_hidden_services_parent_dir }}",
        tor_wait_for_hidden_services: yes,
        sudo: yes
      }
```



Example Tor Relay Playbook
--------------------------

This ansible play will fully configure a fleet of servers as tor
relays... with an ssh hidden service and appropriate firewall rules.

Additionally here we demonstrate this ansible-tor role's ability to
wait for the hidden services to be created... by awaiting the
existence of the tor hidden service hostname files. This happens
when the role variable "tor_wait_for_hidden_services" is set to yes.

This feature could be useful when configuring other services that
depend on knowing the hidden service's onion address.


Read about tor hidden services here:
https://www.torproject.org/docs/tor-hidden-service.html.en


```yml
---
- hosts: tor-relays
  user: ansible
  connection: ssh
  vars:
    relay_hidden_service_name: "hidden_ssh"
    hidden_ssh_virtport: 6669
    hidden_ssh_target: "127.0.0.1:22"
    relay_hidden_services_parent_dir: "/var/lib/tor/services"
    relay_hidden_services: [ { dir: "{{ relay_hidden_service_name }}",
                               ports: [ { virtport: "{{ hidden_ssh_virtport }}",
                                 target: "{{ hidden_ssh_target }}" } ] }
    ]
  roles:
    - { role: ansible-role-firewall,
        firewall_allowed_tcp_ports: [ 22, 9001 ],
        sudo: yes
      }
    - { role: ansible-wheezy-common,
        sudo: yes
      }
    - { role: Ansibles.openssh,
        sudo: yes
      }
    - { role: david415.ansible-tor,
        ansible_distribution_release: "wheezy",
        tor_ExitPolicy: "reject *:*",
        tor_hidden_services: "{{ relay_hidden_services }}",
        tor_hidden_services_parent_dir: "{{ relay_hidden_services_parent_dir }}",
        tor_wait_for_hidden_services: yes,
        sudo: yes
      }
```


Tor configuration - torrc
-------------------------

Suggestions for a better templating scheme welcome.
For the time being it works fairly well to have
torrc options be set from host_vars/group_vars and
also set from role variables.

The host_vars variable names must begin with "tor_";
Here's an example setting "Nickname":

```yml
tor_Nickname: [ "OnionRobot" ]
```

The dictionary value is a list because in some cases you may want to
specify multiple lines in the torrc that begin with the dictionary key.

Currently, the torrc template looks like this:

```yml
{% for configkey, value in hostvars[inventory_hostname].iteritems() %}
{% if configkey|truncate(3, True) == "tor..." %}
{% for element in hostvars[inventory_hostname][configkey] %}
{{ configkey|replace("tor_", "") }} {{ element }}
{% endfor %}
{% endif %}
{% endfor %}

{% if tor_ExitPolicy is defined %}
ExitPolicy {{ tor_ExitPolicy }}
{% endif %}

{% if tor_ORPort is defined %}
ORPort {{ tor_ORPort }}
{% endif %}
{% if tor_UseBridges is defined %}
UseBridges {{ tor_UseBridges }}
{% endif %}
{% if tor_BridgeRelay is defined %}
BridgeRelay {{ tor_BridgeRelay }}
{% endif %}
{% if tor_PublishServerDescriptor is defined %}
PublishServerDescriptor {{ tor_PublishServerDescriptor }}
{% endif %}
{% if tor_ServerTransportPlugin is defined %}
ServerTransportPlugin {{ tor_ServerTransportPlugin }}
{% endif %}
{% if tor_ServerTransportListenAddr is defined %}
ServerTransportListenAddr {{ tor_ServerTransportListenAddr }}
{% endif %}


{% if tor_hidden_services is defined %}
{% for service in tor_hidden_services %}
HiddenServiceDir {{ tor_hidden_services_parent_dir }}/{{ service.dir }}
{% for hidden_port in service.ports %}
HiddenServicePort {{ hidden_port.virtport }} {{ hidden_port.target }}
{% endfor %}
{% endfor %}
{% endif %}
```



License
-------

MIT


Feature requests and bug-reports welcome!
-----------------------------------------

https://github.com/david415/ansible-tor/issues

