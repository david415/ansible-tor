Ansible Tor
===========

This is an Ansible role for use with Tor - https://www.torproject.org/

Requirements
------------

written and tested on debian wheezy


Tor configuration - torrc
-------------------------

Suggestions for a better templating scheme welcome.
For the time being it works fairly well to have
torrc options be set from host_vars/group_vars and
also set from role variables.

Example using host_vars; variable names must begin with "tor_" :

```yml
tor_Nickname: [ "OnionRobot" ]
```

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

{% for service in tor_hidden_services %}
HiddenServiceDir {{ tor_hidden_services_parent_dir }}/{{ service.dir }}
{% for hidden_port in service.ports %}
HiddenServicePort {{ hidden_port.virtport }} {{ hidden_port.target }}
{% endfor %}
{% endfor %}

```


see example playbook below... >_<


Example Tor Relay Playbook
--------------------------

This play waits for the tor hidden services hostname files to exist
when this role variable is set:

```yml
tor_wait_for_hidden_services: yes
```

Here we setup a Tor relay with a Hidden Service directed to our ssh port...
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
        tor_ExitPolicy: "reject *:*",
        tor_hidden_services: "{{ relay_hidden_services }}",
        tor_hidden_services_parent_dir: "{{ relay_hidden_services_parent_dir }}",
        tor_wait_for_hidden_services: yes,
        sudo: yes
      }
```



License
-------

MIT


Feature requests and bug-reports welcome!
-----------------------------------------

https://github.com/david415/ansible-tor/issues

