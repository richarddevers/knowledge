Dump postgre table
==================

.. code-block:: bash 

    /opt/vmware/vpostgres/current/bin/pg_dump -U postgres --table=comp_bprequest --data-only --column-inserts vcac > /storage/db/comp_bprequest.sql
