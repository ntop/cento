ClickHouse Export
=================

nProbe™ Cento can export flow records directly to a `ClickHouse <https://clickhouse.com>`_ database. The export format is compatible with the ntopng ClickHouse flow dump, allowing ntopng to visualize and analyze flow records produced by nProbe™ Cento.

This integration is designed for high-speed monitoring environments where nProbe™ Cento handles packet capture and flow generation at line rate, while ntopng provides flow visualization, historical analysis, and reporting by reading flows directly from the shared ClickHouse database.

Overview
--------

The typical deployment involves:

1. **nProbe™ Cento** captures packets and exports flow records to ClickHouse.
2. **ClickHouse** stores the flow records in a format compatible with ntopng.
3. **ntopng** connects to the same ClickHouse database in read-only mode to visualize and analyze the flows.

.. code-block:: text

   Network Traffic -> nProbe Cento (write) -> ClickHouse -> ntopng (read-only)

Connection
~~~~~~~~~~

The ``--clickhouse`` option specifies the ClickHouse server address and port. The port refers to the ClickHouse native TCP protocol port (default: 9000). To enable SSL encryption, append ``s`` to the port number.

Examples:

.. code-block:: console

   # Connect to localhost on default port 9000
   cento -i zc:eth1 --clickhouse 127.0.0.1

   # Connect to a remote host on default port 9000
   cento -i zc:eth1 --clickhouse 192.168.1.1

   # Connect with SSL encryption
   cento -i zc:eth1 --clickhouse 192.168.1.1:9000s

Authentication
~~~~~~~~~~~~~~

The ``--clickhouse-auth`` option specifies the ClickHouse credentials in ``user:password`` format. By default, it uses the ClickHouse ``default`` user.

.. code-block:: console

   cento -i zc:eth1 --clickhouse 192.168.1.1 --clickhouse-auth myuser:mypassword

Database Name
~~~~~~~~~~~~~

The ``--clickhouse-dbname`` option specifies the database name. The default is ``ntopng``, which matches the database name used by ntopng when dumping flows to ClickHouse.

.. code-block:: console

   cento -i zc:eth1 --clickhouse 192.168.1.1 --clickhouse-dbname ntopng

Interface ID
~~~~~~~~~~~~

The ``--clickhouse-interface-id`` option sets the ntopng interface ID associated with exported flows. This ID is used by ntopng to identify the source interface when reading flows. The default value is ``0``.

.. code-block:: console

   cento -i zc:eth1 --clickhouse 192.168.1.1 --clickhouse-interface-id 3

Cluster Support
~~~~~~~~~~~~~~~

The ``--clickhouse-cluster`` option enables export to a ClickHouse cluster. When specified, nProbe™ Cento creates the database and dumps flows to the cluster, enabling data replication across cluster nodes.

.. code-block:: console

   cento -i zc:eth1 --clickhouse 192.168.1.1 --clickhouse-cluster mycluster

This option is intended for use with a properly configured ClickHouse cluster. When connecting to a single-node ClickHouse instance, omit this option.

Integration with ntopng
-----------------------

Flow export to ClickHouse is designed to be compatible with the ntopng flow dump to ClickHouse. This allows ntopng to read historical flow records produced by nProbe™ Cento directly, without having to dump flows itself.

ntopng supports a read-only mode specifically designed for this integration when started with the ``--readonly-flows-dump`` option. With this option ntopng connects to ClickHouse to read flows produced by nProbe™ Cento without writing any flow data itself.

Step 1: Start nProbe Cento
~~~~~~~~~~~~~~~~~~~~~~~~~~

Start nProbe™ Cento with ClickHouse dump enabled, in addition to ZMQ export:

.. code-block:: console

   cento -i zc:eth1 --zmq tcp://127.0.0.1:5556 --clickhouse 192.168.1.1 --dpi-level 2

Step 2: Start ntopng in Read-Only Mode
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Start ntopng with ClickHouse support in read-only mode using the ``-F clickhouse`` and ``--readonly-flows-dump`` options:

.. code-block:: console

   ntopng -i tcp://*:5556c -F clickhouse --readonly-flows-dump

In this configuration, ntopng reads the historical flow data produced by nProbe™ Cento from ClickHouse, and provides historical flow visualization and analysis without performing any flow dump itself.
At the same time ntopng processes live traffic by collecting flows from ZMQ.

Please refer to the `ntopng documentation <https://www.ntop.org/guides/ntopng/flow_dump/>`_ for additional information on configuring ntopng with ClickHouse.

