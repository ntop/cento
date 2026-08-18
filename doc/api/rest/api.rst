RESTful API Specification
=========================

Authentication
--------------

There are no default credentials. Authentication is disabled for every user until a
password is explicitly configured for that user in Redis, as described below.

Please note that the HTTP basic access authentication should be used for authentication,
for example with `curl` it is possible to specify username and password with
:code:`-u <user>:<password>` as in the command below:

.. code:: bash

   curl -u <user>:<password> "https://192.168.1.1:8880/egress/aggregated/default?action=forward"

Please check the *Examples* section for more examples.

Setting the REST API Password
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Credentials are stored in Redis (configured via :code:`--redis`). Passwords are kept as
salted SHA-256 hashes under the key :code:`cento.user.<username>.password`, in the format
:code:`sha256:<salt>:<hash>`, where :code:`<salt>` is a random 16-byte hex string and
:code:`<hash>` is the SHA-256 digest of :code:`<salt>` concatenated with the plaintext
password.

To set or change the password for a user, run the command below by replacing
:code:`admin` with the actual username and :code:`newpassword` with the desired password.

.. code:: bash

   SALT=$(openssl rand -hex 16); redis-cli SET cento.user.admin.password "sha256:${SALT}:$(printf '%s%s' "$SALT" 'newpassword' | openssl dgst -sha256 -r | awk '{print $1}')"

If the Redis instance listens on a non-default host or port, supply the connection details
with :code:`-h host` and :code:`-p port`.

API
---

.. swaggerv2doc:: rest-api.json
