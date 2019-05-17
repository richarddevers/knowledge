Db Dump
*********

Dump postgre table
--------------------

.. code-block:: bash 

    /opt/vmware/vpostgres/current/bin/pg_dump -U postgres --table=comp_bprequest --data-only --column-inserts vcac > /storage/db/comp_bprequest.sql

Dump the whole postgre db
---------------------------

https://kb.vmware.com/s/article/2074214

Log in to the VMware vRealize Automation virtual appliance using SSH.
Run this command to stop the VMware vRealize Automation service.

.. code-block:: bash

    service vcac-server stop
 
Go to /tmp

.. code-block:: bash

    cd /tmp
 
Run this command to create a copy of the database in /tmp (Note: Both the database name and user name are vcac.
It is recommended to use pg_dumpall as it is a complete export file. This includes multiple databases (if present) and user role information.

.. code-block:: bash

    su -m -c "/opt/vmware/vpostgres/current/bin/pg_dumpall -c -f /tmp/vcac.sql" postgres
 
Run this command to compress the database:

.. code-block:: bash

    bzip2 -z /tmp/vcac.sql
 
Run this command to restart the VMware vRealize Automation service:

.. code-block:: guess

    service vcac-server start
 
Use SCP or WinSCP to transfer the vcacvadbdump.bz2 file out of the appliance.