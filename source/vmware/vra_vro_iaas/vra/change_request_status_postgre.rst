Change request status
=====================

Identy the postgre master node using the vami (database table)

.. code-block:: rest

    https://vra-va-hostname.domain.name:5480

Connect to it using ssh

Connect to the database and use postgre transaction (see :ref:`postgre_transaction`)

Check the request status

.. code-block:: psql

    select * from cat_request where requestnumber='your_request_id';

Update it to change it state to 'FAILED'

.. code-block:: psql

    update cat_request set state='FAILED' where requestnumber='your_request_id';

Ensure you got UPDATE 1 response from the db, otherwise you may have another issue :)

Verify the new state:

.. code-block:: psql

    select * from cat_request where requestnumber='your_request_id';

Once done, you can commit your transaction

Eventually, got to the vRa portal and check the request status.