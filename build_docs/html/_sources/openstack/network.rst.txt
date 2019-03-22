Network
*********

OpenVswitch
===========

List tags by namespace/project/instance

.. code-block:: bash

    ovs-vsctl show

Dump flow of a specific bridge:

.. code-block:: bash

    ovs-vfctl dump-flow $(BRIDGE_NAME)

Various
=======

Lister les namespace réseaux

.. code-block:: bash

    ip netns

Executer une commande depuis la stack ip d'un namespace réseaux:

.. code-block:: bash

    ip netns exec <namespace_name> <command dans le namespace>
    ip netns exec <namespace_name> ip api
    eg: ip netns exec qrouter-90321d2f-2b4f-46af-a8d5-b11f2afa67f7 curl 10.5.5.4


.. code-block:: bash

    ovs-vsctl show => list tags by namespace/project/instance