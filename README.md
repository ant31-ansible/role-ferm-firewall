Ferm-Firewall
==========

Manage and configure the ferm firewall. Send separate configuration file per groups.
You may need to change ansible `hash` from replace to merge .

Requirements
------------

 - ferm package

Role Variables
--------------
Default variables:

```yaml
---
# Your default ferm rules for all hosts
ferm_default_rules:
  # INPUT
  - {rule: "policy DROP;", comment: "global policy", chain: "INPUT"}
  - {rule: "mod state state INVALID DROP;", comment: "connection tracking: drop", chain: "INPUT"}
  - {rule: "mod state state (ESTABLISHED RELATED) ACCEPT;", comment: "connection tracking", chain: "INPUT"}
  - {rule: "interface lo ACCEPT;", comment: "allow local packet", chain: "INPUT"}
  - {rule: "proto icmp ACCEPT;", comment: "respond to ping", chain: "INPUT"}
  - {rule: "proto tcp dport ssh ACCEPT;", comment: "allow SSH connections", chain: "INPUT"}
  # OUTPUT
  - {rule: "policy ACCEPT;", comment: "global policy", chain: "OUTPUT"}
  #FORWARD
  - {rule: "policy DROP;", comment: "global policy", chain: "FORWARD"}
  - {rule: "mod state state INVALID DROP;", comment: "connection tracking: drop", chain: "FORWARD"}
  - {rule: "mod state state (ESTABLISHED RELATED) ACCEPT;", comment: "connection tracking", chain: "FORWARD"}

# The specifics ferm rules (empty by default).
ferm_rules: {
key: {rule: policy, comment: describe rule, chain: CHAIN}
}
# Identify your playbook with an app_name
app_name: "appname"

```

Dependencies
------------
 - None

Example Playbook
----------------
Ferm rules are hash instead of array. The main reason is to be able to merge hashes when configure same host with different roles.
```yaml
- hosts: mongodb
  vars:
    - ferm_rules:
  roles:
    - ferm
```
License
-------

MIT
