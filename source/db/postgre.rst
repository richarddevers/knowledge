.. _postgre_transaction:

*************
Transaction
*************

If implemented, switch to the postgre user context

.. code-block:: psql

    su - postgres

Connect to the db (in this example, the db is vcac)

.. code-block:: psql

    psql vcac

Set extended display mode

.. code-block:: psql

    \x

Begin the new transaction

.. code-block:: psql

    BEGIN;

Now you can perform safely your request without any risk of harming your db.

If your update perform a one line update in a table, you 'll have the following result 'UPDATE 1'

eg for request:

.. code-block:: psql

    update my_table set my_column='my_data' where my_field='other_data';

You'll get the message

.. code-block:: psql

    UPDATE 1;

If you want to commit your modification use the following:

.. code-block:: psql

    COMMIT;