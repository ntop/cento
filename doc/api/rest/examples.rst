Examples
========

Identifying the base REST Endpoint
----------------------------------

The base REST endpoint to contact when configuring egress queues has to be built according to the following format

.. code-block:: text

   https://<host>:<port>/egress/{aggregated,balanced,bridge}/

The user will specify *aggregated* or *balanced* or *bridge* after the /egress/ part of the url depending on what kind of egress is being configured. 

Configuring Queue-Level Rules
-----------------------------

Once the base endpoint has been built, one has to add the keyword default to configure the default policy, followed by a ?-separated query string with the *action* that can be any of: *forward*, *discard*, *shunt*, *slice-l4* and *slice-l3*. 

For example, a default slice-l4 policy can be set to policy the queue-level of the aggregated egress queues as follow

.. code-block:: text

   https://<host>:<port>/egress/aggregated/default?action=slice-l4

Similarly, a default shunt policy for balanced egress queues can be set as:

.. code-block:: text

   https://<host>:<port>/egress/balanced/default?action=shunt

Configuring Subnet-Level Rules
------------------------------

Subnet-level rules must specify the keyword *ip*, followed by a *subnet* parameter (value has the CIDR format <address/mask>) needed to indicate the subnet that is being policed. An *action* parameter is required to indicate the actual policy that has to be enforced. Action can be any of: *forward*, *discard*, *shunt*, *slice-l4* and *slice-l3*.

For example, the following request can be issued to enforce, on balanced egress queues, a discard policy for subnet 192.168.2.0/24

.. code-block:: text

   https://<host>:<port>/egress/balanced/ip?subnet=192.168.2.0/24&action=discard

Alternatively, one may set a *forward* policy for subnet 192.168.3.0/24 on aggregated egress queues as:

.. code-block:: text

   https://<host>:<port>/egress/aggregated/ip?subnet=192.168.3.0/24&action=forward

Configuring Protocol-Level Rules
--------------------------------

Protocol-level rules must specify the keyword *protocol*, followed by the actual protocol string that has to be policed. An *action* is required to indicate the actual policy that has to be adopted. Action can be any of: *forward*, *discard*, *shunt*, *slice-l4* and *slice-l3*.

For example, to enforce, on balanced egress queues, a discard policy for protocol SSH, one may perform the following request 

.. code-block:: text

   https://<host>:<port>/egress/balanced/protocol/SSH?action=discard

Alternatively, one may set a *forward* policy for protocol HTTP on aggregated egress queue as

.. code-block:: text

   https://<host>:<port>/egress/aggregated/protocol/HTTP?action=forward


