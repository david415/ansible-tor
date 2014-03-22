Ansible Tor
===========

Ansible Tor role for Tor relay operators.


Requirements
------------


Role Variables
--------------


Dependencies
------------


Example Tor relay Playbook
--------------------------

  - hosts: tor-relays
    sudo: True
    user: human
    connection: ssh
    vars:
      torrc: torrc-nonexit
    roles:
      - ansible-tor

License
-------

MIT


Feature requests and bug-reports welcome
----------------------------------------

https://github.com/david415/ansible-tor/issues

