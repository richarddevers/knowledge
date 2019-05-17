********
Package
********

In vRo during the creation of package, the action will be included in the package from the scriptable and the action used in the wf if the getModule call is done using double quote (") for the path.

In this eg, the action will NOT be added to the package:

.. code-block:: js
    
    var myVar = System.getModule('my.path.to.module').myAction();


In this eg, the action will be added to the package:

.. code-block:: js
    
    var myVar = System.getModule("my.path.to.module").myAction();


This is the same behavior for the action used in  OGNL.

However, for OGNL, the action of depth 1 will be taken in account.

Also for OGNL, no space must be set in you action call.


In this OGNL eg, the action will NOT be added to the package due to the space between the comma and the next double quote:

.. code-block:: js
    
    GetAction("my.path.to.module", "myAction");

However, this one will be add to the package by vRo

.. code-block:: js
    
    GetAction("my.path.to.module","myAction");