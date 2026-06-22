Flow Offload
============

cento has been designed to process high-speed traffic with moderate CPU load.
Traffic processing can be further optimized by offloading flows to the adapter, on Napatech adapters
featuring an hardware Flow Manager. This dramatically reduces the load on the CPU.
Cento offloads flows to the adapter as soon as the application protocol has been detected (this
requires a few packets for each flow, depending on the actual protocol).

Adapter Configuration
~~~~~~~~~~~~~~~~~~~~~

.. note:: Flow Manager enabled firmware required
   A Flow Manager enabled firmware is required in order to use flow offload on the adapter.

When enabling flow offload with the *--flow-offload* option in cento, the software receives updates 
from the adapter both periodically (flow still programmed in the adapter) and on termination (flow removed from the hardware table).

Max flow duration is managed by the adapter, which is sending periodic updates to the software (--lifetime-timeout|-t is ignored by cento). 
Every time a periodic update is received, the software sends out a clone of the flow with partial stats since last update. For this reason 
max duration checks are disabled in software when flow offload is enabled, otherwise that would lead to flow table synchronization issues. 
FlowPeriodicStatisticsProfiles (nsec) needs to be configured in the adapter to deliver periodic flow updates.

Max flow duration is set to 10 seconds in the example below (snippet from /opt/napatech3/config/ntservice.ini):

.. code-block:: text

   FlowPeriodicStatisticsProfiles = [0,0,10000000000],[0,0,0],[0,0,0],[0,0,0],[0,0,0],[0,0,0],[0,0,0]


Idle flows are managed by the adapter as well, however the software still checks for idle flows for safety (according to the --idle-timeout|-d
parameter), to avoid keeping flows forever in case the software misses some flow termination message from the adapter. For this reason the flow 
idle timeout in cento should be configured using a value higher than the hardware timeout. FlowTimeOut (msec) needs to be configured in the adapter.

Flow idle timeout is set to 120 seconds in the example below (snippet from /opt/napatech3/config/ntservice.ini):

.. code-block:: text

   FlowTimeOut = 120000 (ms)

Flow offload needs to be configured in the adapter by means of a NTPL script before running cento.
Please find below a sample "flow-offload.ntpl" script that can be used to configure one (or more) stream.

.. code-block:: text

   Delete = All
   
   // Pair of uplink and downlink ports
   Define UL_PORT = Macro("0")
   Define DL_PORT = Macro("1")
   
   // Streams / threads
   Define STREAMS = Macro("(0..0)")
   Define NUMA    = Macro("0")
   
   Define TX_FORWARD = Macro("0x00")
   Define TX_DISCARD = Macro("0x08")
   
   Define FLM_HIT       = Macro("0x00")
   Define FLM_MISS      = Macro("0x10")
   Define FLM_UNHANDLED = Macro("0x20")
   
   Define FLM_DISCARD = Macro("3")
   Define FLM_FORWARD = Macro("4")
   
   Define isIPv4    = Macro("Layer3Protocol==IPv4")
   Define isTcpUdp  = Macro("Layer4Protocol==TCP,UDP")
   
   KeyType[Name=KT_4Tuple] = {32, 32, 16, 16}
   
   Define KeyDefProtoSpec = Macro("(Layer3Header[12]/32, Layer3Header[16]/32, Layer4Header[0]/16, Layer4Header[2]/16)")
   KeyDef[Name=KD_4Tuple; KeyType=KT_4Tuple; IpProtocolField=Outer; KeySort=Sorted] = KeyDefProtoSpec
   
   HashMode = Hash5TupleSorted
   
   Setup[NUMANode=NUMA] = StreamId == STREAMS
   
   // Tx port number is the first 3 bits of the NtDyn1Descr_s->color at offset 138 (0b000XXX)
   // Tx forward/discard verdict is bit 4 of the NtDyn1Descr_s->color at offset 141 (0b00X000)
   Setup[TxDescriptor=DYN; TxPorts=UL_PORT,DL_PORT; TxPortPos=138; TxIgnorePos=141] = StreamId==STREAMS
   
   // Set TX port value, which is the first 3 bits of the NtDyn1Descr_s->color
   Assign[ColorMask=DL_PORT; Descriptor=DYN1] = Port==UL_PORT
   Assign[ColorMask=UL_PORT; Descriptor=DYN1] = Port==DL_PORT
   
   // Set TX forward/discard, which is bit 4 of the NtDyn1Descr_s->color
   Assign[ColorMask=TX_DISCARD; Descriptor=DYN1] = Port==UL_PORT,DL_PORT
   
   // New and unhandled flows are sent to the host
   Assign[StreamId=STREAMS; ColorMask=FLM_UNHANDLED; Descriptor=DYN1] = Port==UL_PORT,DL_PORT AND isIPv4 AND isTcpUdp AND Key(KD_4Tuple, KeyID=1) == UNHANDLED
   Assign[StreamId=STREAMS; ColorMask=FLM_MISS;      Descriptor=DYN1] = Port==UL_PORT,DL_PORT AND isIPv4 AND isTcpUdp AND Key(KD_4Tuple, KeyID=1) == MISS
   
   // Drop discarded flows
   Assign[StreamId=DROP] = Port==UL_PORT,DL_PORT AND isIPv4 AND isTcpUdp AND Key(KD_4Tuple, KeyID=1) == FLM_DISCARD
   
   // Forward allowed flows between ports
   Assign[StreamId=Drop; DestinationPort=DL_PORT] = Port==UL_PORT AND isIPv4 AND isTcpUdp AND Key(KD_4Tuple, KeyID=1) == FLM_FORWARD
   Assign[StreamId=Drop; DestinationPort=UL_PORT] = Port==DL_PORT AND isIPv4 AND isTcpUdp AND Key(KD_4Tuple, KeyID=1) == FLM_FORWARD


Note: the above NTPL is similar to the inline/bridge configuration, expection made for the *ColorMask=TX_DISCARD* assignment.

Command to load the NTPL script to the adapter:

.. code-block:: bash

   sudo /opt/napatech3/bin/ntpl -f flow-offload.ntpl

Running Cento
~~~~~~~~~~~~~

In order to enable flow offload in nProbe Cento, a :code:`--flow-offload` option is available. Example:

.. code-block:: bash

   cento -i nt:stream0 --flow-offload --dpi-level 2

Configure Multiple Streams
~~~~~~~~~~~~~~~~~~~~~~~~~~

In order to run cento on multiple streams and scale performance, change the STREAMS definition in the NTPL script
as below (4 streams in the example):

.. code-block:: text

  Define STREAMS = Macro("(0..3)")

Reload the NTPL script to the adapter:

.. code-block:: bash

   sudo /opt/napatech3/bin/ntpl -f flow-offload.ntpl

Run cento on top of all configured streams:

.. code-block:: bash

   cento -i nt:stream[0-3] --flow-offload --dpi-level 2

Please note that when running cento at high speed, it is recommended to also tune the flow cache size
and the CPU cores affinity. This is an example:

.. code-block:: bash

   cento -i nt:stream[0-3] --flow-offload --dpi-level 2 --processing-cores 0,1,2,3 --exporting-cores 4,5,6,7 --max-hash-size 10000000

