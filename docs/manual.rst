FDBus Manual
============

Abstract
--------

This manual describes a new type of ``IPC`` mechanism: Fast Distributed Bus (``FDBus``). 
From the perspective of ``IPC`` (Inter-Process Communication), ``FDBus`` has similarities 
with widely used ``D-Bus`` (Desktop Bus), but ``FDBus`` has its own advantages, more complete 
functions, higher performance and convenient use, in addition to supporting IPC in the host. 
It can also be networked between multiple hosts, and can customize security policies to support 
different security levels. ``FDBus`` is built on sockets (``Unix`` domain and ``TCP``) and 
serialized and deserialized using Google protobuf. ``FDBus`` supports the name of a string as 
the server address. The ``name server`` automatically assigns a ``Unix`` domain address and a 
``TCP`` port number to the server, so that the service name is used between the client and the server.

``FDBus`` aims to provide a connection-oriented, scalable, secure and reliable ``IPC`` mechanism 
between client-servers, and then develop into a middleware development framework for cross-platform
 (``Windows``, ``QNX``, ``Linux``), multi-threaded/multi-process A middleware layer that works together.
 The ``FDBus`` development framework is suitable for developing interactive and complex distributed 
 projects on custom systems, including:

- Linux-based vehicle ECU, including instrumentation, entertainment host, TBox, 
  domain controller connected via Ethernet
- Communication between multiple Guest OSs on Hypervisors
- Provide cross-host IPC mechanism for Android system (currently does not support Java API)
- Small communication devices based on Linux, such as home routers
- Other Linux-based industrial equipment, smart equipment
- Automated test equipment based on Windows development

.. image:: ./images/1.png
  :width: 600px
