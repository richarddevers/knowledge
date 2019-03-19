Postman
========

vRo API official documentation
===============================
https://code.vmware.com/apis/176/vro-rest

Pre request script to log to vRo API
====================================

.. code-block:: js

    const obj = {
        "username":pm.environment.get("username"),
        "password":pm.environment.get("password"),
        "domain":pm.environment.get("tenant"),
        "client_id":pm.environment.get("cafe_cli")
    }
    const payload = Object.keys(obj).reduce((acc, cur) => {
        return `${acc}${cur}=${obj[cur]}&`;
    }, '');

    pm.sendRequest({
        url: "https://" + pm.environment.get("vra-fqdn") +"/SAAS/t/" +  pm.environment.get("tenant") + "/auth/oauthtoken?grant_type=password",
        method: 'POST',
        header: {
            'Content-Type': 'application/x-www-form-urlencoded'
        },
        body: {
            mode: 'x-www-form-urlencoded',
            raw: payload
        }
    }, function (err, res) {
        pm.environment.set("vro_token", res.json().access_token);
    });