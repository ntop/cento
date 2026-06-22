RESTful API
===========

Cento implements a RESTful API, using an embedded HTTP server, for dynamically reconfiguring the forwarding engine.

The RESTful API can be enabled by adding the :code:`[--rest-address|-Y] <address>` 
option and accessed via HTTP (on port 8880 by default) or via HTTPS (on port 4443 by default).

HTTP and HTTPs ports can be changes using the :code:`[--rest-http-port|-T] <port>`
and :code:`[--rest-https-port|-S] <port>` options.

In order to use HTTP, a certificate is required. Create it with:

.. code-block:: console

   mkdir -p /usr/local/share/cento/httpdocs/ssl
   openssl req -new -x509 -sha1 -extensions v3_ca -nodes -days 365 -out cert.pem
   cat privkey.pem cert.pem > /usr/local/share/cento/httpdocs/ssl/ssl-cert.pem

Sample command line using the --rest-address option for enabling the REST API:

.. code-block:: console

   cento-ids -i zc:eth1 --aggregated-egress-queue --egress-conf egress.example --dpi-level 2 -v 4 --rest-address 127.0.0.1

Defaults settings:

 - HTTP port: :code:`8880`
 - HTTPs port: :code:`4443`
 - User: :code:`admin`
 - Password: :code:`admin`

Please check the REST API specifications for information about the available REST endpoints.

This is available on *cento-ids* and *cento-bridge* only as it is mainly used 
to change the egress configuaration at runtime.


.. toctree::
    :maxdepth: 2
    :numbered:

    api
    examples
