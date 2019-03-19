Modify heap size
================

.. code-block:: bash

    /usr/lib/vco/app-server/bin/setenv.sh
    MEM_OPTS="-Xmx2048m -XX:MaxPermSize=256m -Xss256k -XX:+HeapDumpOnOutOfMemoryError"