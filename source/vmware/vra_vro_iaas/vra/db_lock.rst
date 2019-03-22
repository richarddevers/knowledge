Db lock
*********

Identiy DB locked
=================

.. code-block:: psql

    select * from saas.databasechangeloglock;

Delete DB Lock
==============

.. code-block:: psql

    database update saas.databasechangeloglock set locked=FALSE, lockgranted=NULL, lockedby=NULL where id=1;