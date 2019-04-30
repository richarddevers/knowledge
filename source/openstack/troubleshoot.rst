Troubleshooting
****************

Neutron
=======

P344 of Mastering OpenStack

List agent of a network node:
ip netns

qrouter-UUID: Represents the L3 agent router namespace and instance connected to the router UUID created in the network node
qdhcp-UUID: Represents the DHCP agent namespace for a private network created in the network node

Delving deeper in to this network picture would bring more details in each namespace and show different IP addresses attached to active interfaces:
   # ip netns exec qrouter-a029775e-204b-45b6-ad86-
   0ed2507d5bf ip a

Attached routes to the same router namespace can be checked as follows:
   # ip netns exec qrouter-a029775e-204b-45b6-ad86-0ed2507d5bf ip r

Dedicated IPtables per each namespace are available by running the following:
   # ip netns exec qrouter-a029775e-204b-45b6-ad86-0ed2507d5bf iptables -n

Nova
=====

nova list

nova hypervisor-list

nova live-migration --block-migrate b51498ca-0a59-42bd-945a-18246668186d cc02.pp (shared storage needed)

If compute down, reboot vm on another hypervisor (p343 of Mastering OpenStack):

MariaDB> use nova;
   MariaDB [nova]> update instances set host='cc02.pp'
   where host='cc01.pp';

Once record sets are updated, each instance requires a libvirt's XML file to boot properly, these can be located in/etc/libvirt/qemu/instance-*.xml. To start instances while creating the XML file, user reboot command line by adding the --hard option

nova reboot --hard b51498ca-0a59-42bd-945a-18246668186d

Log d'une instance:

nova console-log b51498ca-0a59-42bd-945a-18246668186d

The complete console logs for each instance resides by default in the /var/lib/nova/instances/INSTANCE_UIID/console.log file, where INSTANCE_UIID is the instance UUID under question.