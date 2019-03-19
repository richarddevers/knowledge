.. _change_vm_lease_iaas:

Change vm lease in iaas db
==========================

Find data about your vm

.. code-block:: sql

    select VirtualMachineID, VirtualMachineName, Expires from VirtualMachine where VirtualMachineName='my_vm_name' and IsManaged = 1

Update the lease date

.. code-block:: sql

    update into VirtualMachine (Expires) values ("2020-12-31 22:59:59.000") where VirtualMachineID='my_vm_id'

Check the result

.. code-block:: sql

    select VirtualMachineID, VirtualMachineName, Expires from VirtualMachine where VirtualMachineID = 'my_vm_id'

To avoir inconsistency, you should also do it in the iaas db, see :ref:`change_vm_lease_postgre`