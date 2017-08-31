# Cumulus MLAG Role

Define the Inventory like this. Start each MLAG pair with the words "mlag_...."
```
[mlag_leaf1_leaf2]
na-snv-slab-leaf1
na-snv-slab-leaf2

[mlag_spine1_spine2]
na-snv-slab-spine1
na-snv-slab-spine2

[mlag_leaf3_leaf4]
na-snv-slab-leaf3
na-snv-slab-leaf4

[mlag_switches:children]
mlag_spine1_spine2
mlag_leaf1_leaf2
mlag_leaf3_leaf4
```


Then in the group_vars add the following. Example:

```
# group_vars/mlag_leaf3_leaf4.yml
# --------------------------------

# Define a System Mac for the mlag pair
system_mac: '44:39:39:FF:40:12'

# Name of the 2 switches in the MLAG.
mlag_hosts: ['na-snv-slab-leaf3', 'na-snv-slab-leaf4']

# List the peer link members
peerlink:
  members: ['swp52-53']

# List the physical interfaces that form host bonds.
# Module will take care of creating bonds and call it bond[ifacename]
# Example bondswp1. Best practise implementation is applied.
# the port list can be written like this - { port: swp1, pvid: 2 }
# to define a different native vlan setting for the port.
# If you specify "vids" make sure to include the pvid(native vlan)
# unless you specify vlan 1, which means you don't want a native vlan.
# See examples from "inventory/group_vars/mlag_leaf1_leaf2.yml"
server:
  - { port: swp1, descr: server1, pvid: 1, vids: '1', descr: 'Server One' }
  - { port: swp5, descr: server2, pvid: 100, vids: '100-200', descr: 'Server Two' }

# Otherwise known as Orphan ports.
access:
  na-snv-slab-leaf3:
    - { port: swp23, vid: 22, descr: 'Orphan Server One' }
  na-snv-slab-leaf4: []


## Inner Workings

This role creates each interface as individual files in /etc/network/interfaces.d/
It creates a "interface bridge" interface and automatically adds the vlans that you define in the
``server:`` and ``access:``  setting.

It also automatically creates a _random_ clag ID based on the interface names. So it never changes onces it is created and so far has been pretty unique.

So all the user of this role needs to change is the group_var settings for ``server:`` and ``access:``.


## Requirements

* Cumulus OS 3.0+
* Ansible 2.2+


## License
MIT

## Author
Stanley Karunditu @linuxsimba

