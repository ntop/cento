nProbe Cento Documentation
==========================

Measuring network traffic is a fundamental task in any modern packet-switched network. Accurate measurements offer an effective support in the timely diagnosis of network issues. Misbehaving hosts, faulty adapters, intruders, undesired traffic, are just a few examples of issues that are likely to occur in any real-world deployment. Other popular use cases that demand for accurate traffic monitoring include, but are not limited to, billing and reporting systems used by service providers and network operators.

The steady increase in network and adapter speeds is determining an alway increasing demand for novel measurement systems that are able to keep up with ultra-high-speed environments. Nowadays, 40 and 100 Gbps adapters are rapidly becoming popular and will slowly replace the mainstream 10 Gbps adapters. Being able to process traffic at these speeds in software requires to deeply reconsider the well-established design choices in order to leverage on modern multi-CPU, multi-core elaboration systems.

Another challenge that is currently being faced is the ability to actively measure network traffic. Passive measurements no longer suffice as they do provide answers but, at the same time, fail to provide timely solutions. Ex post analyses are still useful but can’t prevent or alleviate the effects of an undergoing network issue. For these reasons, there is an increasing interest in systems that are able to proactively react and decide on the basis of monitored data such as, for example, systems that shape or even isolate an attacker in a network.

nProbe™ Cento is a high-speed NetFlow probe that has been designed to address the issues above. Cento is able to keep up with 1/10/40/100 Gbps adapters. However, it is not just a fast NetFlow probe, it has been designed as the first component of a modular monitoring system: besides capturing ingress packets and computing flow data, it can be used to classify the traffic via DPI (Deep Packet Inspection) and to optionally perform actions based on selected packets/flows when it is used as a traffic forwarder in combination with other applications such as IPS/IDS and traffic recorders.

Main Features
-------------

nProbe™ Cento features include:

NetFlow v5/v9/IPFIX export support.

- Export also in JSON, Text, Kafka, Syslog.
- IPv4 and IPv6 support.
- Native PF_RING and PF_RING ZC support for high-speed packet processing.
- Support of FPGA-based network adapters.
- A scalable, multithreaded design: ingress traffic load can be balanced across multiple streams on multicore architectures.
- Full Layer-7 application visibility using nDPI (Deep Packet Inspection).
- Lightweight Layer-7 application visibility using µ-nDPI (a lightweight DPI library supporting the most important protocols such as HTTP/HTTPS/DNS).
- Flow-based Load Balancing to Intrusion Detection and Intrusion Prevention Systems IDS/IPS including Snort, Bro, and Suricata.
- Traffic filtering based on Layer-7 protocols, CIDR, and interfaces to reduce the load on the IDS/IPS.
- Feedback channel for traffic filtering/shunting from an IPS: “forward this flow”, “drop this flow”, “divert this flow through the IPS”.
- Traffic filtering and slicing for saving storage space removing meaningless data when forwarding traffic to a packet-to-disk application such as n2disk.


.. toctree::
   :maxdepth: 2
   :caption: User's Guide

   installation
   versions
   definitions
   usecases
   clickhouse
   offload
   egress
   bridge
   options
   systemd
   api/index
   hugepages
   geolocation
   references

.. toctree::
   :caption: Other Products

   ntopng <https://www.ntop.org/guides/ntopng/>
   nProbe <https://www.ntop.org/guides/nprobe/>
   n2disk <https://www.ntop.org/guides/n2disk/>
   nDPI <https://www.ntop.org/guides/nDPI/>
   PF_RING <https://www.ntop.org/guides/pf_ring/>
   nEdge <https://www.ntop.org/guides/nedge/>
   nScrub <https://www.ntop.org/guides/nscrub/>
   nBox <https://www.ntop.org/guides/nbox/>
   nTap <https://www.ntop.org/guides/ntap/>
   
.. Indices and tables
.. ==================
.. 
.. * :ref:`genindex`
.. * :ref:`modindex`
.. * :ref:`search`

