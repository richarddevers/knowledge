.. _change_owner_iaas:

Change owner in the vRa Iaas db
=========================================

Find new owner id

.. code-block:: sql

    select * from users where userName like '%John Doe%'

Update the virtualmachine with the new owner

.. code-block:: sql

    set Owner = 'new_owner_id' where VirtualMachineName= 'my_vm_name' 