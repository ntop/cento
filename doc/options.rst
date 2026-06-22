Command Line Options
====================

nProbe™ Cento command line options are briefly summarized below. The same summary shown below can obtained by running nProbe™ Cento with argument --help. Non-trivial options are then logically re-arranged and discussed in greater detail after the brief summary.

Interfaces
----------

*[--interface|-i] <ifname|pcap file>*

nProbe™ Cento handles network adapters both with vanilla or PF_RING drivers. An interface can be specified as “-i eth0” (in-kernel processing with vanilla or PF_RING-aware drivers) or as “-i zc:eth0” (kernel bypass). The prefix zc: tells PF_RING to use the Zero Copy (ZC) module.

nProbe™ Cento can also be fed with multiple Receive Side Scaling (RSS) queues available from one or more interfaces. Queues are specified using the @ sign between the interface name and the queue number. For example, the following command is to run nProbe™ Cento on 4 ZC queues of interface eth1

.. code-block:: console

   cento -i zc:eth1@0 -i zc:eth1@1 -i zc:eth1@2 -i zc:eth1@3

The same 4 queues can be processed in-kernel by omitting the zc: prefix as follows

.. code-block:: console

   cento -i eth1@0 -i eth1@1 -i eth1@2 -i eth1@3

nProbe™ Cento, for every distinct interface specified using -i, starts two parallel threads, namely a packet processor thread and a flow exporter thread. Additional threads may be spawned as discussed below.

The number of interfaces, as well as the number of queues, that have to be processed with nProbe™ Cento can grow significantly. For this reason a compact, square-brackets notation is allowed to briefly indicate ranges of interfaces (of queues). For example, using the compact notation, nProbe™ Cento can be started on the 4 ZC eth1 discussed above queues as

.. code-block:: console

   cento -i zc:eth1@[0-3]

Similarly, one can start the software on four separate ZC interfaces as

.. code-block:: console

   cento -i zc:eth[0-3]

Egress queues
-------------

*[--balanced-egress-queues|-o] <num|devices>*

Using this option nProbe™ Cento creates, for each interface submitted with -i, a number <num> of egress queues. Alternatively, a comma-separated list of egress devies (physical interfaces) can be specified. Egress traffic is balanced across the egress queues created using the 5-tuple. Therefore it is guaranteed that all the packets belonging to a given flow will be sent to the same egress queue. A detailed description of balanced egress queues is given in section “Egress Queues”. Following is just a quick example.

Assuming nProbe™ Cento has to be configured to monitor two eth1 ZC queues such that the traffic coming from each one will be forwarded to two distinct egress queues, it is possible to use the following command

.. code-block:: console

   cento -i zc:eth1@0 -i zc:eth1@1 --balanced-egress-queues 2

nProbe™ Cento output confirms the traffic will be properly balanced and forwarded to the egress queues:

.. code-block:: text

   [...]
   Initialized global ZC cluster 10
   Created interface zc:eth1@0 bound to 0
   Reading packets from interface zc:eth1@0...
   Created interface zc:eth1@1 bound to 2
   Reading packets from interface zc:eth1@1...
   Forwarding interface zc:eth1@0 balanced traffic to zc:10@0
   Forwarding interface zc:eth1@0 balanced traffic to zc:10@1
   [ZC] Egress queue zc:10@0 (0/2) bound to interface zc:eth1@0
   [ZC] Egress queue zc:10@1 (1/2) bound to interface zc:eth1@0
   Forwarding interface zc:eth1@1 balanced traffic to zc:10@2
   Forwarding interface zc:eth1@1 balanced traffic to zc:10@3
   [ZC] Egress queue zc:10@2 (0/2) bound to interface zc:eth1@1
   [ZC] Egress queue zc:10@3 (1/2) bound to interface zc:eth1@1
   [...]

Egress queues are identified with a string zc:<cluster id>:<queue id>, e.g., zc:10@0. Cluster  and queue ids assigned are output by nProbe™ Cento. Egress queues can be fed to other applications such as Intrusion Detection Systems (IDS) and Intrusion Prevention Systems (IPS). A simple packet count can be performed using the PF_RING pfcount utility. Counting packets received on ZC queue 1 on cluster 0 is as simple as

.. code-block:: console

   pfcount -i zc:10@0

Instead, to forward traffic to physical interfaces, a comma-separated list of interface names can be specified:

.. code-block:: console

   cento -i zc:eth1 --balanced-egress-queues zc:eth2,zc:eth3


For a detailed description of the PF_RING architecture we refer the reader to the PF_RING User’s Guide available at https://www.ntop.org/guides/pf_ring


*[--aggregated-egress-queue|-F [<dev>]]*

nProbe™ Cento can aggregate the traffic coming from every input interface and forward it to a single global aggregated egress queue. In other words, nProbe™ Cento run with this option acts as an N-to-1 traffic aggregator. This is basically the “dual” outcome of what is achieved via --balanced-egress-queues. In the latter case, nProbe™ Cento  acts as a 1-to-N traffic balancer. A detailed description of aggregated egress queues is given in section “Egress Queues”. Following is just a brief example.

Assuming nProbe™ Cento has to be configured to aggregate together 4 eth1 ZC queues, such that the traffic coming from each one will be aggregated in a single egress queue, it is sufficient to execute the following command

.. code-block:: console

   cento -izc:eth1@0 -izc:eth1@1 -izc:eth1@2 -izc:eth1@3 --aggregated-egress-queue

nProbe™ Cento output confirms the traffic will be properly aggregated into a single queue identified by zc:10@0.

.. code-block:: text

   [...]
   Initialized global ZC cluster 10
   Created interface zc:eth1@0 bound to 0
   Reading packets from interface zc:eth1@0...
   Created interface zc:eth1@1 bound to 2
   Reading packets from interface zc:eth1@1...
   Created interface zc:eth1@2 bound to 4
   Reading packets from interface zc:eth1@2...
   Created interface zc:eth1@3 bound to 6
   Reading packets from interface zc:eth1@3...
   Fwd.ing aggregated traffic to zc:10@0, you can now start your recorder/IDS
   [...]

In general, as already highlighted above, any egress queue is identified with a string zc:<cluster id>:<queue id>, e.g., zc:10@0.

To count the aggregated traffic it is possible to use the PF_RING pfcount utility as

.. code-block:: console

   pfcount -i zc:10@0

The same queue can be input to traffic recorders such as n2disk as well as to IDS/IPS.

nProbe™ Cento can also aggregate and send incoming traffic to a physical network device. In this case the device name <dev> has to be specified right after the option --aggregated-egress-queue.


*[--egress-conf|-e] <file>*

Both aggregated and balanced egress queues allow a fine-grained traffic control through a set of hierarchical rules. Rules are specified in a plain text <file> that follows the INI standard. A thorough description of the file format is give in the section “Egress Queues” of the present manual.

*[--balanced-egress-buffer] <buffer size>*

When the *--balanced-egress-queues* option is used, it is also possible to define specific destination queues/interfaces for selected traffic in the rules file provided with *--egress-conf*, see the Egress Queues - Policy Rules section for more details.
In this case, if the traffic is traffic is selected by application protocol, since the DPI engine may take a few packet to detect the application (depending on the actual protocol), the initial packets may not be delivered to the expected destination.
In order to overcome this, a *--balanced-egress-buffer* has been introduced to store packets in memory until the application has been detected, at that point all packets are forwarded to destination. This option required a fairly high amount of memory,
depending on the traffic rate, which is allocated from hugepages. The <buffer size> should be specified, which is the maximum number of packets that can be buffered, used by the application to reserve hugepages memory at startup.

Flows Generation
----------------

*[--lifetime-timeout|-t] <sec>*

nProbe™ Cento considers expired and emits any flow that has been active for more than <sec> seconds. If the flow is still active after <sec> seconds, further packets belonging to it will be accounted for on a new flow.

Default lifetime timeout is set to 300 seconds.


*[--lifetime-perturbation] <sec>*

Adds a random perturbation in the range 0 - <sec> to newly created flows, in order to avoid them to expire at the same time (in case of long lasting flows). By default, perturbation is disabled.


*[--idle-timeout|-d] <sec>*

nProbe™ Cento considers over and emits any flow that has not been transmitting packets for <sec> seconds. If the flow is still active and will transmit again after <sec> seconds, its further packets will be accounted for on a new flow.

This also means that, when applicable (e.g. SNMP walk) UDP flows will not be accounted on a 1 packet/1 flow basis, but on a single global flow that accounts all the traffic. This has the benefit in that it reduces the total number of generated flows and improves the overall collector performance.

Default idle timeout is set to 180 seconds.


*[--skip-fragments]*

Tells nProbe™ Cento to discard fragmented IP traffic. This is particularly useful in ultra-high speed environments where keeping track of fragments could be too costly.


*[--tunnel]*

Decode tunneled packets.

*[--cache-flow-direction|-U]*

For TCP flows, this option enables caching of flow directions so they are preserved in case of long flows

CPU Affinity
------------

*[--processing-cores|-g <cores>]*

nProbe™ Cento spawns a parallel processor thread for every input interface. Any processor thread is in duty to process all the packets associated to its interface and to build the corresponding flows. This design allows to leverage on modern multi-core multi-CPU architectures as threads can work independently and in parallel to process traffic flowing from multiple interfaces.

The affinity enables the user to bind any processor thread (and thus any interface) to a given core. Using this option, nProbe™ Cento will instruct the operating system scheduler to execute processor threads on the specified cores. Cores are identified using their core ids. Core ids can be found directly under /sys/devices.

For example, core ids for a 4-core hyper-threaded CPU Intel Xeon E3-1230 v3 can be found with

.. code-block:: console

   ~$ cat /sys/devices/system/cpu/cpu*/topology/thread_siblings_list
   0,4
   1,5
   2,6
   3,7
   0,4
   1,5
   2,6
   3,7

Every line has two numbers: the first is the core id and the second is the associated hyper-threaded core. Lines are repeated two times as the same information is contained both for the core and for its associated hyper-threaded core. Using affinity and core ids it is possible to greatly optimize cores usage and make sure multiple applications don’t interfere each other in high-performance setups.

To start nProbe™ Cento on two ZC interfaces and to bind processor threads to cores with id 0 and 1 respectively it is possible to run the following command

.. code-block:: console

   cento -i zc:eth0 -i zc:eth1 -g 0,1


*[--exporting-cores|-G <cores>]*

nProbe™ Cento spawns an exporter thread for every input interface. Any exporter thread deals with the export of flows according to the user submitted options. Examples of exporter threads are Netflow V5 / V5 exporters. The same description given for the --processing-cores option applies here to set the affinity of exporter cores.


*[--monitor-aggregator-core|-M <core>]*

This option regulates the affinity of nProbe™ Cento thread exports flows to a monitoring virtual interface for traffic visibility (see option --monitor).


*[--timer-core|-N <core>]*

This option regulates the affinity of nProbe™ Cento thread that keeps track of the time. Keeping this thread in a lightly loaded core allows to obtain very precise timestamps, especially when no hardware timestamping is available.

Flows Export Settings
---------------------

These settings instruct the interface exporter threads on the output format.


*[--dump-path|-P] [<d1>:<d2>:<ml>:]<dir>*

nProbe™ Cento exports flows in a plain, pipe-separated text format when --dump-path is specified.  Every interface is exported in a <dir> sub-directory that has the same name of the interface. Every subdirectory contains a tree of directories arranged in a nested, “timeline” fashion as: year, month, day, hour of the day, minute of the hour. The subdirectories that are the “leaves” of this tree contain the actual text files.

The subdirectory tree is populated according to the parameters <d1>, <d2> and <ml>. Specifically, nProbe™ Cento creates a new subdirectory every <d1> seconds. A text file contains at most <ml> flows, one per line. If more that <ml> flows are going to be written in less that <d2> seconds, then one or more files are created. If less that <ml> flows have been written in <d2> seconds, the flows file is closed and a new one is created.

For example, to capture from two eth1 ZC queues and write gathered flows to /tmp it is possible to use the following command

.. code-block:: console

   cento -izc:eth1@0 -izc:eth1@1 -P 300:50:20000:/tmp/

The command instructs nProbe™ Cento to dump flows into /tmp with the extra conditions that:
Every flows file shouldn’t contain more that 20000 flows and shouldn’t contain flows worth more than 50 seconds of traffic.
Every subdirectory shouldn’t contain more that 300 seconds of traffic.

nProbe™ Cento output confirms it is going to use two subdirectories in /tmp one for each interface

.. code-block:: text

   [...]
   Dumping flows onto /tmp/zc_eth1@0
   Dumping flows onto /tmp/zc_eth1@1
   [...]

An example of the resulting directory structure is as follow

.. code-block:: console

   ~/ find  /tmp/zc_eth1\@0/*
   /tmp/zc_eth1@0/2016
   /tmp/zc_eth1@0/2016/05
   /tmp/zc_eth1@0/2016/05/04
   /tmp/zc_eth1@0/2016/05/04/18
   /tmp/zc_eth1@0/2016/05/04/18/1462379942_0.txt
   /tmp/zc_eth1@0/2016/05/04/18/1462380145_0.txt
   /tmp/zc_eth1@0/2016/05/04/18/1462379214_0.txt
   /tmp/zc_eth1@0/2016/05/04/18/1462379603_0.txt
   /tmp/zc_eth1@0/2016/05/04/18/1462379881_0.txt

The default value for <d1> is 300 seconds.
The default value for <d2> is 60 seconds.
The default value for <ml> is 10000 seconds.


*[--dump-compression <mode>]*

nProbe™ Cento can export flows to compressed text files when used with option --dump-path. Dump compression format is controlled via this option by selecting a <mode>.

The <mode> can have the following values:
- n: No compression (default)
- g: gzip compression
- b: bzip compression


*[--v5|-5 <host:port>]*

Exports NetFlow to the destination <host:port> using version 5. A comma-separated list can be used for load-balancing across multiple destinations.


*[--v9|-9 <host:port>]*

Exports NetFlow to the destination <host:port> using version 9. A comma-separated list can be used for load-balancing across multiple destinations.


*[--ipfix|-I <host:port>]*

Exports IPFIX to the destination <host:port>. A comma-separated list can be used for load-balancing across multiple destinations.


*[--zmq <endpoint>]*

Flows can be exported by nProbe™ Cento via ZMQ to a given <endpoint>. Typically this <endpoint> is an instance of ntopng that continuously collect received flows for further analysis and visualization. A comma-separated list of endpoints is also supported, to load-balance flow export to multiple endpoints. Note: probe mode is used, configure ntopng as collector. See section “Integration with ntopng” for a detailed description of the use case.


*[--zmq-encrypt-pwd <pwd>]*

Flows that are exported via ZMQ by nProbe™ Cento can be optionally encrypted using a <pwd>. The same <pwd> must be used on the <endpoint> as well to properly decrypt received flows.


*[--zmq-direct-mapping]*

When multiple *--zmq* endpoints and multiple interfaces are specified, this option maps each interface directly to a corresponding ZMQ endpoint (first interface to first endpoint, second interface to second endpoint, etc.). Without this option, flows from all interfaces are load-balanced across all ZMQ endpoints.

**Example**

.. code-block:: console

  cento -i zc:eth1@[0-1] --zmq tcp://127.0.0.1:5556 --zmq tcp://127.0.0.1:5557 --zmq-direct-mapping


*[--tcp <address>:<port>]*

Flows can be exported by nProbe™ Cento in JSON format via TCP to a given server specifying <addres> and <port>. It is possible to use labels instead of numeric keys by adding the option *--json-labels*.


*[--kafka <brokers>;<topic>[;<ack>;<compr>]*

nProbe™ Cento can export flows to one or more Kafka <brokers> that are responsible for a given <topic> in a cluster. While the topic is a plain text string, Kafka brokers must be specified as a comma-separated list according to the following format

.. code-block:: text

   <host1>[:<port1>],<host2>[:<port2>]

Initially, nProbe™ Cento tries to contact one or more user-specified brokers to retrieve Kafka cluster metadata. Metadata include, among other things, the list of brokers available in the cluster that are responsible for a given user-specified topic. Once the metadata-retrieved list of brokers is available, nProbe™ Cento starts exporting flows to them.

The user can also decide to compress data indicating in <compr> one of none, gizp, and snappy. Additionally, the <ack> policy can be specified as follows:
- <ack> =  0: Don’t wait for ACKs 
- <ack> =  1: Receive an ACK only from the Kafka leader
- <ack> = -1: Receive an ACK from every replica 

**Example**

Assuming there is a Zookeeper on localhost port 2181 that manages the Kafka cluster, it is possible to ask for the creation of a new Kafka topic. Let’s say the topic is called “topicFlows” and it has to be split across three partitions with replication factor two. The command that has to be issued is the following

.. code-block:: console

   bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic topicFlows \ 	--partitions 3 --replication-factor 2

Now that the Kafka topic has been created, it is possible to execute nProbe™ Cento and tell the instance to export to Kafka topic “topicFlows”. We also tell the instance a list of three brokers all running on localhost (on ports 9092, 9093 and 9094 respectively) that will be queried at the beginning to obtain kafka cluster/topic metadata.

.. code-block:: console

   cento --kafka “127.0.0.1:9092,127.0.0.1:9093,127.0.0.1:9094;topicFlows"

At this point the nProbe™ Cento instance is actively exporting flows to the Kafka cluster. To verify that everything is running properly, it is possible to take messages out of the Kafka topic with the following command

.. code-block:: console

   bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic topicFlows\ 	--from-beginning 


*[--avro]*

When exporting flows to Kafka, nProbe™ Cento uses JSON encoding by default. It is possible to optinally enable Avro encoding when exporting data to Kafka by using the *--avro* option.


*[--template]*

Contrary to nProbe™ Pro/Enterprise, nProbe™ Cento uses a fixed template when exporting Netflow v5/v9/IPFIX data, however  when exporting to ZMQ, Kafka (--kafka) or TCP (--tcp) using the JSON or Avro encoding, nProbe™ Cento provides some flexibility, with the ability to select the export template, similar to nProbe™ Pro/Enterprise (but with a limited set of Information Elements). If the *--template* option is not specified, a default template compatible with ntopng is used.

Example: 

.. code-block:: console

   cento -i eno1 --kafka “127.0.0.1:9092,127.0.0.1:9093,127.0.0.1:9094;topicFlows" --json-labels --template "%SRC_VLAN %SRC_MAC %DST_MAC %IP_PROTOCOL_VERSION %IPV4_SRC_ADDR %IPV4_DST_ADDR %IPV6_SRC_ADDR %IPV6_DST_ADDR %EXPORTER_IPV4_ADDRESS %DIRECTION %INPUT_SNMP %OUTPUT_SNMP %SRC_TO_DST_PKTS %SRC_TO_DST_BYTES %DST_TO_SRC_PKTS %DST_TO_SRC_BYTES %FIRST_SWITCHED %LAST_SWITCHED %L4_SRC_PORT %L4_DST_PORT %PROTOCOL %L7_PROTO %L7_PROTO_NAME %L7_PROTO_CATEGORY"

List of supported Information Elements:

.. code-block:: text

   %BIFLOW_DIRECTION
   %CLIENT_NW_LATENCY_MS
   %CLIENT_TCP_FLAGS
   %DIRECTION
   %DNS_NUM_ANSWERS
   %DNS_QUERY
   %DNS_QUERY_ID
   %DNS_QUERY_TYPE
   %DNS_RESPONSE
   %DNS_RET_CODE
   %DNS_TTL_ANSWER
   %DOWNSTREAM_TUNNEL_ID
   %DST_AS
   %DST_AS_MAP
   %DST_IP_CITY
   %DST_IP_COUNTRY
   %DST_MAC
   %DST_TOS
   %DST_TO_SRC_BYTES
   %DST_TO_SRC_PKTS
   %DST_VLAN
   %EXPORTER_IPV4_ADDRESS
   %FIRST_SWITCHED
   %FLOW_DOMAIN_NAME
   %FLOW_DURATION_MILLISECONDS
   %FLOW_END_MILLISECONDS
   %FLOW_SERVER_NAME
   %FLOW_START_MILLISECONDS
   %FLOW_USER_NAME
   %FLOW_UUID
   %HASSH_CLIENT
   %HASSH_SERVER
   %HTTP_HOST
   %HTTP_RET_CODE
   %HTTP_URL
   %HTTP_USER_AGENT
   %INPUT_SNMP
   %IN_BYTES
   %IN_PKTS
   %IPV4_DST_ADDR
   %IPV4_SRC_ADDR
   %IPV6_DST_ADDR
   %IPV6_SRC_ADDR
   %IP_DST_ADDR
   %IP_PROTOCOL_VERSION
   %IP_SRC_ADDR
   %JA4C_HASH
   %L4_DST_PORT
   %L4_SRC_PORT
   %L7_APP_PROTOCOL
   %L7_APP_PROTOCOL_NAME
   %L7_PROTO
   %L7_PROTO_NAME
   %L7_PROTO_CATEGORY
   %L7_PROTO_RISK
   %L7_RISK_SCORE
   %L7_SERVICE
   %L7_SERVICE_NAME
   %LAST_SWITCHED
   %OOORDER_IN_PKTS
   %OOORDER_OUT_PKTS
   %OUTPUT_SNMP
   %OUT_BYTES
   %OUT_PKTS
   %PROTOCOL
   %QOE_DST_TO_SRC
   %QOE_SRC_TO_DST
   %RETRANSMITTED_IN_PKTS
   %RETRANSMITTED_OUT_PKTS
   %RTP_IN_PAYLOAD_TYPE
   %RTP_OUT_PAYLOAD_TYPE
   %SERVER_NW_LATENCY_MS
   %SERVER_TCP_FLAGS
   %SRC_AS
   %SRC_AS_MAP
   %SRC_IP_CITY
   %SRC_IP_COUNTRY
   %SRC_MAC
   %SRC_TOS
   %SRC_TO_DST_BYTES
   %SRC_TO_DST_PKTS
   %SRC_VLAN
   %TCP_FLAGS
   %TLS_ALPN
   %TLS_CERT_AFTER
   %TLS_CERT_ISSUER_DN
   %TLS_CERT_NOT_BEFORE
   %TLS_CERT_SHA1
   %TLS_CERT_SUBJECT_DN
   %TLS_CIPHER
   %TLS_REQUESTED_SNI
   %TLS_SERVER_NAME
   %TLS_SERVER_NAMES
   %TLS_UNSAFE_CIPHER
   %TLS_VERSION
   %UPSTREAM_TUNNEL_ID


In this configuration, nProbe™ Cento also provides the ability to:

- define aliases, that is renaming an Information Element (e.g. L7_PROTO to APPLICATION_PROTOCOL)
- define custom Information Elements, that means adding new fields with hardcoded values (e.g. DOMAIN, with type "string" and value "example.local").

Aliases can be defined by using the below syntax when listing Information Elements in the template:

.. code-block:: text

   %ORIGINAL_NAME=>NEW_NAME

Example (check the APPLICATION_PROTOCOL alias definition):

.. code-block:: console

   cento -i eno1 --kafka “127.0.0.1:9092,127.0.0.1:9093,127.0.0.1:9094;topicFlows" --json-labels --template "%SRC_VLAN %SRC_MAC %DST_MAC %IP_PROTOCOL_VERSION %IPV4_SRC_ADDR %IPV4_DST_ADDR %IPV6_SRC_ADDR %IPV6_DST_ADDR %EXPORTER_IPV4_ADDRESS %DIRECTION %INPUT_SNMP %OUTPUT_SNMP %SRC_TO_DST_PKTS %SRC_TO_DST_BYTES %DST_TO_SRC_PKTS %DST_TO_SRC_BYTES %FIRST_SWITCHED %LAST_SWITCHED %L4_SRC_PORT %L4_DST_PORT %PROTOCOL %L7_PROTO=>APPLICATION_PROTOCOL %L7_PROTO_NAME=>APPLICATION_PROTOCOL_NAME"

Example with Avro format (check the APPLICATION_PROTOCOL alias definition):

.. code-block:: console

   cento -i eno1 --kafka “127.0.0.1:9092,127.0.0.1:9093,127.0.0.1:9094;topicFlows" --avro --template "%SRC_VLAN %SRC_MAC %DST_MAC %IP_PROTOCOL_VERSION %IPV4_SRC_ADDR %IPV4_DST_ADDR %IPV6_SRC_ADDR %IPV6_DST_ADDR %EXPORTER_IPV4_ADDRESS %DIRECTION %INPUT_SNMP %OUTPUT_SNMP %SRC_TO_DST_PKTS %SRC_TO_DST_BYTES %DST_TO_SRC_PKTS %DST_TO_SRC_BYTES %FIRST_SWITCHED %LAST_SWITCHED %L4_SRC_PORT %L4_DST_PORT %PROTOCOL %L7_PROTO=>APPLICATION_PROTOCOL %L7_PROTO_NAME=>APPLICATION_PROTOCOL_NAME"

Custom Information Elements and types can be defined by using the below syntax when listing Information Elements in the template:

.. code-block:: text

   %VAR:STRING=<name>:<value>
   %VAR:INT=<name>:<value>
   %VAR:LONG=<name>:<value>

Example:

.. code-block:: console

   cento -i eno1 --kafka “127.0.0.1:9092,127.0.0.1:9093,127.0.0.1:9094;topicFlows" --json-labels --template "%SRC_VLAN %SRC_MAC %DST_MAC %IP_PROTOCOL_VERSION %IPV4_SRC_ADDR %IPV4_DST_ADDR %IPV6_SRC_ADDR %IPV6_DST_ADDR %EXPORTER_IPV4_ADDRESS %DIRECTION %INPUT_SNMP %OUTPUT_SNMP %SRC_TO_DST_PKTS %SRC_TO_DST_BYTES %DST_TO_SRC_PKTS %DST_TO_SRC_BYTES %FIRST_SWITCHED %LAST_SWITCHED %L4_SRC_PORT %L4_DST_PORT %PROTOCOL %L7_PROTO %L7_PROTO_NAME %VAR:STRING=DOMAIN:example.local"

*[--skip-empty]*

(supported with --template only)

Do not export empty Information Elements (e.g. skip DNS fields when the flow application protocol is not DNS).

*[--monitor|-A]*

Export flows to a monitoring virtual interface for traffic visibility


ClickHouse Export Settings
--------------------------

*[--clickhouse] <host[:port[s]]>*

Export flows to a ClickHouse database at the specified host and port. The port refers to the ClickHouse native TCP protocol port (default: 9000). Append ``s`` to the port number to enable SSL encryption for the connection.

Flow data exported to ClickHouse is compatible with the ntopng flow dump format, allowing ntopng to read flow records produced by nProbe™ Cento directly. See the `ClickHouse Export <clickhouse.html>`_ section for a detailed description of this integration.

Examples:

.. code-block:: console

   cento -i zc:eth1 --clickhouse 127.0.0.1
   cento -i zc:eth1 --clickhouse 192.168.1.1:9000s


*[--clickhouse-dbname] <name>*

ClickHouse database name. Default: ``ntopng``.


*[--clickhouse-auth] <user:pwd>*

ClickHouse authentication credentials in the format ``user:password``. Default: ``default`` (ClickHouse default user with no password).


*[--clickhouse-interface-id] <id>*

ntopng interface ID for exported flows. This ID is used by ntopng to identify the interface that produced the flows. Default: ``0``.


Miscellaneous Settings
----------------------

*[--blacklists] <file>*

In the specified <file> contains a list of files (separated by carriage return) thet contain blacklisted IP. Example: if <file> contains the following lines

.. code-block:: console

	/home/cento/blacklist-file1.txt
	/home/cento/blacklist-file2.txt
	/home/cento/blacklist-file3.txt

cento will open all the files and load the enclosed IP (both IPv4 and IPv6 are supported) addresses. The blacklist file format contains IP addresses with an optional CIDR. Example: both 1.2.3.4 and 192.168.1.0/24 are valid IP addresses. In case a flow (source or destination IP) will match at least one IP of the list, the flow is considered blacklisted and it will not be forwarded (in case cento has been started in inline mode).

*[--dpi-level|-D] <level>*

This option enables Deep Packet Inspection (DPI) for flows. DPI attempts to detect Layer-7 Application Protocols by looking at packet contents rather than just using Layer-4 protocols and ports. As DPI is a costly operation in high-speed environments, nProbe™ Cento can optionally operate in µ-DPI mode that increases performances by focusing only on the most common Layer-7 protocols.

The <level> is used to specify the DPI version that has to be activated:

- 0: DPI disabled
- 1: enable uDPI (obsolete, use -D 2)
- 2: enable nDPI (full fledged nDPI engine)


*[--active-poll|-a]*

Enables active packet polling. Using this option nProbe™ Cento processes will actively spin to check for new packets rather than using polling mechanisms or sleeps.


*[--dont-drop-privileges|-z]*

By default, nProbe™ Cento drops current user privileges to make sure the process is run with the least possible set of permissions. This option prevents privileges from being dropped.


*[--pid-file|-p <path>]*

Write the process Id in the specified file


*[--hash-size|-w] <size>*

Sets the per-interface flow cache hash size. The size is exposed using a value optionally followed by the suffix “MB”. If the suffix is not specified, then the value is interpreted as the number of buckets. If the suffix is specified, then the value is interpreted as the in-memory size of the flow cache.

Default: 512000 [estimated hash memory used 144.5 MB]


*[--max-hash-size|-W] <size>*

Sets the max number of active hash buckets (it defaults to 2 million). Usually -W should be at least twice the value of -w. Note that in addition to the active flow buckets, there are flows being exported so you can have in total more that -W active flows. This option is good to limit the memory being used by cento even in the worst case of (almost) infinite number of flows (e.g. in case of DoS attack) .


*[--redis|-R] <host[:port][@db-id]>*

Redis DB host[:port][@database id]. Redis needs to be used when the REST API is used.


*[--daemon|-b]*

Starts the software in daemon mode.


*[--version]*

Print the version and system identifier.


*[--check-license]*

Checks if the license is present and valid.


*[--check-maintenance]*

Checks the maintenance status for the specified license.


*[--local-nets|-L] <local nets>*

Local nets list (default: no address defined). Example: -L “192.168.0.0/24,172.16.0.0/16”.


*[--banned-hosts|-B] <path>*

nProbe™ Cento features a host ban functionality that can be enabled using this option. The list of banned hosts must be specified, one per line, in a text file located at <path>. µ-nDPI engine is required.

For example, to ban facebook.com and google.com from interface eth0, one should create a text file, let’s say banned.txt, with the following two lines

.. code-block:: text

   google.com
   facebook.com

An then start nProbe™ Cento as

.. code-block:: console

   cento -ieth0 -D 1 -B /tmp/banned.txt

PF_RING / PF_RING Zero Copy
---------------------------

*[--zc|-Z]*

Forces the use of zero copy on linux.


*[--cluster-id|-C] <id>*

Forces nProbe™ Cento to use the ZC cluster id <id>. If no cluster id is specified, and arbitrary id will be assigned by nProbe™ Cento.

REST
----

*[--rest-address|-Y <address>]*

Enable a REST server binding it to the specified address


*[--rest-http-port|-T <port>]*

REST server HTTP port


*[--rest-https-port|-S] <port>*

REST server HTTPs port


*[--rest-docs-root|-J] <dir>*

REST server docs root directory

