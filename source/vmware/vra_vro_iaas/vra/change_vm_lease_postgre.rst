.. _change_vm_lease_postgre:

Change vm lease in vRa postgre
********************************

Identy the postgre master node using the vami (database table)

.. code-block:: rest

    https://vra-va-hostname.domain.name:5480

Connect to it using ssh

Connect to the database and use postgre transaction (see :ref:`postgre_transaction`)

Get the id of the vm

.. code-block:: psql

    select name,id,lease_end_date from cat_resource where status = 'ACTIVE' and name = 'my_vm_id';

Update the lease

.. code-block:: psql

    update cat_resource set lease_end_date='2020-12-31 22:59:59.000' where id='my_vm_id';

Ensure you got UPDATE 1 response from the db, otherwise you may have another issue :)

Verify the new lease

.. code-block:: psql

    select name,id,lease_end_date from cat_resource where status = 'ACTIVE' and name = 'my_vm_id';

Commit your transaction, eventually check the new lease again.

To avoir inconsistency, you should also do it in the iaas db, see :ref:`change_vm_lease_iaas`
 