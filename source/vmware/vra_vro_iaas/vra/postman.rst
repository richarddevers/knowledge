Postman
********

Pre-request auth script
=======================

.. code-block:: js

    pm.sendRequest({
        url: "https://" + pm.environment.get("vra-fqdn") +"/identity/api/tokens",
        method: 'POST',
        header: {
            'Content-Type': 'application/json',
            'Accept':'application/json'
        },
        body: {
            mode: 'raw',
            raw: JSON.stringify({ username: pm.environment.get("username"), password: pm.environment.get("password"),tenant: pm.environment.get("tenant")})
        }
    }, function (err, res) {
        pm.environment.set("vra_token", res.json().id);
    });