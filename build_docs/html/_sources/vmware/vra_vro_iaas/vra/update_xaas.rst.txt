Update xaas through vRa api
*****************************

The overall process is:
 - Get the id of the xaas-blueprint you want to modify
 - Create a package including theses xaas-blueprint
 - Download the package
 - Modify the yaml file describing your xaas-blueprint
 - Upload the package 


Get the id of the xaas-blueprint
=================================
Perform a GET request on the following url:

.. code-block:: rest

    https://{{vra-fqdn}}/content-management-service/api/contents?$filter=contentTypeId%20eq%20'xaas-blueprint'&limit=9000

Curl code:

.. code-block:: rest

    curl -X GET \
      'https://{{vra-fqdn}}/content-management-service/api/contents?$filter=contentTypeId%20eq%20%27xaas-blueprint%27&limit=9000' \
      -H 'Accept: application/json' \
      -H 'Content-Type: application/json'

Get the id of the blueprint
=============================

Perform a GET request on the following url:

.. code-block:: rest

    https://{{vra-fqdn}}/content-management-service/api/contents?$filter=contentTypeId%20eq%20'composite-blueprint'&limit=9000

Curl code:

.. code-block:: rest

    curl -X GET \
      'https://{{vra-fqdn}}/content-management-service/api/contents?$filter=contentTypeId%20eq%20'composite-blueprint'&limit=9000' \
      -H 'Accept: application/json' \
      -H 'Content-Type: application/json'


Get the blueprint data
=============================

Perform a GET request on the following url:

.. code-block:: rest

    https://{{vra-fqdn}}/content-management-service/api/contents/{{blueprint_id}}/data

Curl code:

.. code-block:: rest

    curl -X GET \
      'https://{{vra-fqdn}}/content-management-service/api/contents/{{blueprint_id}}/data' \
      -H 'Accept: text/yaml' \
      -H 'Content-Type: text/yaml'


Update the blueprint
=============================

After getting the YAML data of the blueprint, you can send it with your modifications on the following URL:

.. code-block:: rest

    https://{{vra-fqdn}}/content-management-service/api/contents/composite-blueprint/{{blueprind_id}}/data

Curl code:

.. code-block:: rest

    curl -X POST \
      'https://{{vra-fqdn}}/content-management-service/api/contents/composite-blueprint/{{blueprind_id}}/data' \
      -H 'Accept: text/yaml' \
      -H 'Content-Type: text/yaml'
      -d @data.yaml


Create a package
=============================
Perform a POST request on the following url:

.. code-block:: rest

    https://{{vra-fqdn}}/content-management-service/api/packages/

The body have this structure:

.. code-block:: rest

    {
    "name":"Test",
    "description":"test",
    "contents":[ "{{id}}"]
    } 

**contents** is the array containing the id of the xaas-blueprint you want to modify.

If the request is accepted, vRa send you a 201 CREATED code.
In response header, you will find the id of your package in the **Location** field.

.. code-block:: rest

    Location  →https://{{vra-fqdn}}/content-management-service/api/packages/{{loc_id}}

Curl code:

.. code-block:: rest

    curl -X POST \
      https://{{vra-fqdn}}/content-management-service/api/packages/ \
      -H 'Accept: application/json' \
      -H 'Authorization: Bearer XXXXXXXXXXXXXXXXXXXXXX' \
      -H 'Content-Type: application/json' \
      -d '{
    "name":"Test",
    "description":"test",
    "contents":["{{id}}"]
    } '

Download the package
=============================

Perform a GET request on the following url where {{id}} is the package's id:

.. code-block:: rest

    https://{{vra-fqdn}}/content-management-service/api/packages/{{id}}

Set the following header:

.. code-block:: rest

    Accept : application/zip

Using POSTMAN, you must send the request using the **Send & Download** button.

You'll download a file (without format or bin format). Rename it to .zip and unzip it. You'll get a file structure similar to the xaas folder of the current repository.

Curl code:

.. code-block:: rest

    curl -X GET \
      https://{{vra-fqdn}}/content-management-service/api/packages/{{id}} \
      -H 'Accept: application/zip'

  
Update package
=============================

The **metadata.yaml** must not be modified.
The code of your xaas-blueprint is in the subfolder **xaas-blueprint**.
Modify these yaml according to your needs.
Once done, zip your package.

Upload package
=============================
Perform a POST request on the following url:

.. code-block:: rest

    https://{{vra-fqdn}}/content-management-service/api/packages/

Set the following header:

.. code-block:: rest

    Accept : application/zip

Set the body as **form-data**
Add a key to the body name **file**
Upload the zipped package.

Curl code:

.. code-block:: rest

    curl -X POST \
      https://{{vra-fqdn}}/content-management-service/api/packages/ \
      -H 'Accept: application/zip' \
      -H 'content-type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW' \
      -F 'file=@C:\my_package.zip'

  
(Optional) List package
=============================
Perform a GET request on the following url:

.. code-block:: rest

    https://{{vra-fqdn}}/content-management-service/api/packages/

(Optional) Delete a package
================================
Perform a DEL request on the following url where {{id}} is the package's id:

.. code-block:: rest

    https://{{vra-fqdn}}/content-management-service/api/packages/{{id}}