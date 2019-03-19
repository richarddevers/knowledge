Check if the failover is activated
==================================

Through vRa:

.. code-block:: python

    python /usr/lib/vcac/tools/vami/commands/manager-service-automatic-failover ENABLE

Through Model manager windows appliance:

.. code-block:: none

    ManagerService.exe.config
    <add key="FailoverModeEnabled" value="True" /> 
