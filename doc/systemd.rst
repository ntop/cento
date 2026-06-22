Systemd
=======

cento is controlled using utility `systemctl` on operating systems
and distributions that use the `systemd` service manager.

Upon successful package installation, the cento service is
automatically started on the loopback interface. The service uses a
configuration file that is located at `/etc/cento/cento.conf` and
that is populated with some defaults during installation. The
configuration file can be edited and extended with any configuration
option supported by cento. A service restart is required after
configuration file modifications.

The cento service is always started on boot by default. The service
must be disabled to prevent this behavior.

The cento service configuration file
------------------------------------

The configuration file is located at `/etc/cento/cento.conf`.

Controlling cento
~~~~~~~~~~~~~~~~~

To start, stop and restart the cento service type:

.. code-block:: console

   systemctl start cento
   systemctl stop cento
   systemctl restart cento

To prevent cento from starting on boot type:

.. code-block:: console

   systemctl disable cento

To start cento on boot, assuming it has previously been disabled,
type:

.. code-block:: console

   systemctl enable cento

To check the status of the service, including its output and PID, type:

.. code-block:: console

   systemctl status cento

Instantiated cento services
~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are circumstances under which multiple instances of the cento
service may run on the same host. To manage a particular `<instance>`
of the service, append an `@<instance>` to the `cento` service name.

Typically, `<instance>` corresponds to an interface name (e.g., `eno1`).
This convention allows an easy identification of the purpose of each
service. Nonetheless, any string is acceptable as value for `<instance>`.

The `<instance>` uniquely identifies a service and its corresponding
configuration file that is located under
`/etc/cento/cento-<instance>.conf`.

For example, to start two cento services, on interface `eno1` and
another on zero-copy interface `eth1` respectively, one can create the
following configuration files:

.. code-block:: text

   /etc/cento/cento-eno1.conf
   /etc/cento/cento-zc:eth1.conf

And then start the services with:

.. code-block:: console

   systemctl start cento@eno1
   systemctl start cento@zc:eth1

Optionally, one may want to start the services on boot with:

.. code-block:: console

   systemctl enable cento@eno1
   systemctl enable cento@zc:eth1

The status of the services above can be controlled with:

.. code-block:: console

   systemctl status cento@eno1
   systemctl status cento@zc:eth1

