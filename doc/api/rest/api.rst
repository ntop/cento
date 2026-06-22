RESTful API Specification
=========================

Authentication
--------------

The default credentials are:

- **Username:** ``admin``
- **Password:** ``admin``

Please note that the HTTP basic access authentication should be used for authentication,
for example with `curl` it is possible to specify username and password with
:code:`-u <user>:<password>` as in the command below:

.. code:: bash

   curl -u <user>:<password> "http://192.168.1.1:8880/egress/aggregated/default?action=forward"

Please check the *Examples* section for more examples.

Changing the REST API Password
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Credentials are stored in Redis (configured via :code:`--redis`). Passwords are kept as MD5
hashes under the key :code:`cento.user.<username>.password`.

To set or change the password for a user, compute the MD5 hash of the desired password and
write it to Redis:

.. code:: bash

   # Compute the MD5 hash of the new password
   NEW_HASH=$(printf '%s' 'newpassword' | md5sum | awk '{print $1}')

   # Store it in Redis (replace 'admin' with the actual username)
   redis-cli SET cento.user.admin.password "$NEW_HASH"

If the Redis instance listens on a non-default host or port, supply the connection details
with :code:`-h` and :code:`-p`:

.. code:: bash

   redis-cli -h 127.0.0.1 -p 6379 SET cento.user.admin.password "$NEW_HASH"

API
---

.. swaggerv2doc:: rest-api.json
