Ferm-Firewall
==========

Manage and configure the ferm firewall. Send separate configuration file per groups.
You may need to change ansible `hash` from replace to merge .

Requirements
------------

 - ferm package

Configuration files and variables structure
-------------------------------------------

 - roles/ansible-role-ferm-firewall/defaults/main.yml
  - this is used in case there is no ferm\_rules defined any where else

Example configuration:

 - group\_vars/all/all
  - this can have a ferm\_rules defined - used on all hosts
 - group\_vars/group/group.yml
  - this can have a ferm\_rules\_extra defined - used in addition to the ferm\_rules

Role Variables
--------------
To configure ferm, you need to provide a key to associate a set of rules to a role/software. This way, rules splited in multiple var-files won't overwrite each other.
By default, if domains isn't defined, it will apply rules to ip6 and ip domains.
Configuration exemple:

```yaml
---
# Your default ferm rules for all hosts
ferm_rules:
# Create a file in /etc/ferm/ferm.d/default.conf
  default:
    - chain: INPUT
      rules:
        - {rule: "policy DROP;",  comment: "global policy"}
        - {rule: "mod state state INVALID DROP;", comment: "connection tracking: drop"}
        - {rule: "mod state state (ESTABLISHED RELATED) ACCEPT;", comment: "connection tracking"}
        - {rule: "interface lo ACCEPT;", comment: "allow local packet"}
        - {rule: "proto icmp ACCEPT;", comment: "respond to ping"}
        - {rule: "proto tcp dport ssh ACCEPT;", comment: "allow SSH connections"}
    # Different set of rules on ip / ip6
    - chain: OUTPUT
      domains:
        - ip
      rules:
        - rule: "policy ACCEPT;"
          comment: global policy
    - chain: OUTPUT
      domains:
        - ip6
      rules:
        - rule: "policy DROP;"
          comment: global policy ip6

    - chain: FORWARD
      domains: [ip, ip6]
      rules:
        - rule: "policy DROP;"
          comment: global policy
        - rule: "mod state state INVALID DROP;"
          comment: "connection tracking: drop"
        - rule: "mod state state (ESTABLISHED RELATED) ACCEPT;"
          comment: "connection tracking"

```

Dependencies
------------
 - None

Example Playbook
----------------
Ferm rules are hash instead of array. The main reason is to be able to merge hashes when configure same host with different roles.

Inventory:
```
[mongodb]
MachineA
[rabbitmq]
MachineA
```
Playbook:
```yaml
---
- hosts: mongodb
  vars:
    - ferm_rules:
        mongodb:
          - chain: INPUT
            rules:
              - {rule: "proto tcp dport (27017) ACCEPT;", comment: "MongoDB mongo shard/repl servers"}
              - {rule: "proto tcp dport (27701 27702 27703) ACCEPT;", comment: "MongoDB mongo configurati\
on servers" }
              - {rule: "proto tcp dport (27801) ACCEPT;", comment: "MongoDB mongo router server (mongos)"\
}
  roles:
    - ferm-firewall

- hosts: rabbitmq
  vars:
    - ferm_rules:
        rabbitmq:
          - chain: INPUT
            domains: [ip]
            rules:
              - rule: "proto tcp dport (5672) ACCEPT;"
                comment: "Rabbitmq-server"
  roles:
    - ferm-firewall

```
Result:

 - /etc/ferm/ferm.d/mongodb.conf
```conf
domain (ip ip6) table filter {
  chain INPUT {
     # MongoDB mongo shard/repl servers
     proto tcp dport (27017) ACCEPT;

     # MongoDB mongo configuration servers
     proto tcp dport (27701 27702 27703) ACCEPT;

     # MongoDB mongo router server (mongos)
     proto tcp dport (27801) ACCEPT;

    }
}
```
 -  /etc/ferm/ferm.d/rabbitmq.conf
```conf
domain (ip ) table filter {
  chain INPUT {
     # Rabbitmq-server
     proto tcp dport (5672) ACCEPT;

    }
}
```


License
-------

MIT
