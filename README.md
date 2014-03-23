Ansible Tor
===========

This is an Ansible role for use with Tor - https://www.torproject.org/
It can be used by relay or bridge operators... and of course this Tor
role can also configure Tor hidden services.

This is a rough draft work in progress.
I need to fix the way the torrc is templatized so that I make
all of tor's configuration options available.


Requirements
------------

written to work on debian wheezy...


Role Variables
--------------

see example playbook below... >_<


Dependencies
------------

I wrote this to work on debian.
If someone wants to submit a patch to make it
compatible with other linux distros that is fine!


Example Tor relay Playbook
--------------------------

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
        firewall_allowed_tcp_ports: [ 22, "{{ hidden_ssh_virtport }}" ],
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
        sudo: yes
      }
```



License
-------

MIT


Feature requests and bug-reports welcome
----------------------------------------

https://github.com/david415/ansible-tor/issues

