Vmotion greyed in vsphere
=============================

Connect to vcenter database:

.. code-block:: guess

    /opt/vmware/vpostgres/current/bin/psql -d VCDB -U postgres 

Get the VM id by running following query:

.. code-block:: psql

    select * from VPX_VM where FILE_NAME LIKE '%VMNAME%';


Check if the migration is locked

.. code-block:: psql

    select * from VPX_DISABLED_METHODS where ENTITY_MO_ID_VAL = '{{my_vm_id}}';

It should return something like

.. code-block:: psql

    -[ RECORD 1 ]----+------------------------------
    entity_mo_id_val | vm-291797
    method_name      | vim.VirtualMachine.relocate
    source_id_val    | vm-291797
    reason_id_val    | Prevent vMotion during backup 


Note the VM id then following KB:

https://kb.vmware.com/s/article/2014714 (first steps seems to be to retrieve only the ID again using the mob)

BUT in step #9, replace method by following only:
<method>RelocateVM_Task</method>

This should allow migrate operation back afterwards:

Externals refs : 

https://kb.vmware.com/s/article/2014714  (turn the option on again without restarting vCenter)
https://kb.vmware.com/s/article/2147285 (connecting to vpostgres)
https://kb.vmware.com/s/article/2008957 (getting in the DB if there’s a lock : steps #3 & #5), also provide a solution but needs vcenter restart ☹ )


