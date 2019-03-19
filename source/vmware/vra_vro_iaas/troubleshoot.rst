Troubleshooting
===============

Log locations for VMware vRealize Automation 7.x (2141175)
    https://kb.vmware.com/s/article/2141175

Log presentation

.. code-block:: bash

    /var/log/vmware/vco/app-server/scripting.log

Log request origin

.. code-block:: bash

    /var/log/vcac/access_log.txt

Log de workflow

.. code-block:: bash

    less /var/log/vco/app-server/scripting.log*  | grep '{workflow_id}'

Log postgres

.. code-block:: bash

    tail -f /storage/db/pgdata/pg_log/postgresql.csv

Log replication

.. code-block:: bash

    tail -f /db/pgdata/pg_log/postgresql.csv

Log clustering (élèction master etc...)

.. code-block:: bash

    tail -f /var/log/vmware/vcac/vcac-config.log

Log server

.. code-block:: bash

    tail -f /var/log/vmware/vco/app-server/catalina.out
    tail -f /var/log/vmware/vco/app-server/server.log 
    tail -f /var/log/vmware/vcac/catalina.out

Iaas Windows
    Windows Event Viewer (Application / System)
    Mware\vCAC
    VMware\vCAC\Server\Logs
    VMware\vCAC\Server\Model Manager Web\Logs
    VMware\vCAC\Web API\Logs\Elmah

Restart vRa services

.. code-block:: bash

    service vco-server status
    service vco-configurator status
    service vcac-server status

Conf postgre

.. code-block:: bash

    /storage/db/pgdata/pg_hba.conf
    /storage/db/pgdata/postgresql.conf

Reload postgre conf

.. code-block:: bash

    /opt/vmware/vpostgres/current/bin/pg_ctl reload -D /var/vmware/vpostgres/current/pgdata/