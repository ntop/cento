Definitions
===========

- Aggregated Egress Queue

A queue that is output by nProbe™ Cento which carries traffic that has been aggregated from multiple input interfaces.

- Balanced Egress Queue

A queue that is output by nProbe™ Cento which carries a subset of traffic received from an input interface. The subset is build to make sure packets belonging to the same flow are always forwarded to the same balanced egress queue.

- Collector

Shorthand for flow collector.

- Egress Queue

A queue that is output by nProbe™ Cento and is consumed by some other software such and an IDS/IPS or a traffic recorder.

- Exporter

Shorthand for flow exporter.

- Flow

Network packets can be aggregated into logical pipes termed “flows”. A flow is uniquely identified by: source and destination IP addresses, source and destination ports, and layer 4 protocol.

- Flow exporter

A piece of hardware/software that outputs flows to a medium (e.g., over the network, to file, to other other software).

- Flow collector

A piece of hardware/software that collects flows from a medium (e.g., from the network, from file, from other software).

- IDS

An Intrusion Detection System that detects known threats, policy violations and malicious behaviors.

- IPFIX

The Internet Protocol Flow Information Export (IPFIX) is a protocol that defines how to transfer flow data from an exporter to a collector.

- IPS

An Intrusion Prevention System that protects the network against possible known threats, policy violators and malicious hosts.

- Kafka

A multi-producer, multi-consumer, publish-subscribe distributed messaging system.

- n2disk

The ntop high-performance packet-to-disk software that records network packets to disk and indexes packet metadata in near realtime to enable fast searches.

- NetFlow v5/v9

Standards that define and describe how to aggregate packets into flows, and how to transfer flow data from an exporter to a collector. 

- ntopng

The ntop network traffic visualization software.

- Packet-to-Disk

The act of writing full network packets (i.e., headers and payloads at any level) to persistent storage. See also traffic recorder.

- Probe

Shorthand for Flow exporter.

- Shunting

The act of filtering network packets that limits the number of per-flow packets to a given fixed value k. Any flow packet that arrives after the k-th is dropped.

- Snort

An open source network IDS for Unix and Windows.

- Suricata

An IDS/IPS to match on known threats, policy violations and malicious behavior.

- Slice-l3

The act of filtering network packets that truncates packets right after the IP headers.

- Slice-l4

The act of filtering network packets that truncates packets right after the TCP/UDP headers.

- Syslog

A standard for message logging.

- Traffic Recorder

A piece of hardware/software that writes network packets to persistent storage (e.g., HDD, SSD, nVME) for archiving purposes or further processing.

- TAP

A network TAP (Test Access Point) is a hardware device inserted at a specific point in the network to monitor full-duplex data.

