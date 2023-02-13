ansible-firewall
================

Performs firewall management for systems supporting iptables or UFW.

Requirements
------------

Role Variables
--------------

| Variable             | Required | Comments |
|----------------------|----------|----------|
| UFW_APPS_TEMPLATES   | yes      | See example playbook. |
| UFW_RULES            | yes      | See example playbook. |
| FIREWALL_RULES       | yes      | See example playbook. |   

Example Playbook
----------------
    - hosts: all
      vars:
         UFW_APPS_TEMPLATE:
           - SCRIPT_NAME: "sample-script"
             APP_TITLE: "App-Title"
             APP_TITLE_LONG: "Longer title"
             APP_DESC: "App description"
             APP_PORTS: "80,443/tcp"
         UFW_RULES:
           - name: "App-Title"
             rule: allow
             from_ip: "10.10.10.0/24"
         FIREWALL_RULES:
           - "-A INPUT -m state --state NEW -m tcp -p tcp -s 10.10.10.0/24 --dport 80 -j ACCEPT"
      roles:
         - ansible-firewall

License
-------
See LICENSE.md

Author Information
------------------
https://cyverse.org

Notes
-----
This repo was created from [core-services/iplant-ansible](https://gitlab.cyverse.org/core-services/iplant-ansible) 
with the following command;

        git clone git@gitlab.cyverse.org:core-services/iplant-ansible.git
        cd iplant-ansible/
        git remote add firewall git@gitlab.cyverse.org:config-mgmt/ansible-firewall.git
        git subtree split --prefix=roles/cso-firewall --branch cso-firewall
        git push firewall cso-firewall:master
        
        
