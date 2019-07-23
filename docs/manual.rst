FDBus Manual
============

Abstract
--------

This manual describes a new type of ``IPC`` mechanism: Fast Distributed Bus (``FDBus``). 
From the perspective of ``IPC`` (Inter-Process Communication), ``FDBus`` has similarities 
with widely used ``D-Bus`` (Desktop Bus), but ``FDBus`` has its own advantages, more complete 
functions, higher performance and convenient use, in addition to supporting ``IPC`` in the host. 
It can also be networked between multiple hosts, and can customize security policies to support 
different security levels. ``FDBus`` is built on sockets (``Unix`` domain and ``TCP``) and 
serialized and deserialized using Google protobuf. ``FDBus`` supports the name of a string as 
the server address. The ``name server`` automatically assigns a ``Unix`` domain address and a 
``TCP`` port number to the server, so that the service name is used between the client and the server.

``FDBus`` aims to provide a connection-oriented, scalable, secure and reliable ``IPC`` mechanism 
between client-servers, and then develop into a middleware development framework for cross-platform 
(``Windows``, ``QNX``, ``Linux``), multi-threaded/multi-process middleware layers which working together. 
The ``FDBus`` development framework is suitable for developing interactive and complex distributed 
projects on custom systems, including:

- Linux-based vehicle ``ECU``, including instrumentation, entertainment host, TBox, 
  domain controller connected via Ethernet
- Communication between multiple Guest OSs on ``Hypervisors``
- Provide cross-host ``IPC`` mechanism for Android system (currently does not support ``Java API``)
- Small communication devices based on ``Linux``, such as home routers
- Other Linux-based industrial equipment, smart equipment
- Automated test equipment based on ``Windows`` development

You may get the open source code of FDBus on Github:
  https://github.com/jeremyczhen/fdbus.git


Background
----------

Unlike other cores, ``Linux`` has not had its own unique and easy-to-use ``IPC`` mechanism. 
``Windows``, ``Mac OS``, and ``QNX`` all have such a mechanism. Even Linux-based ``Android`` 
has developed a binder for ``IPC``. The ``Linux kernel`` only provides some of the most basic 
components - socket, pipe, message queue, shared memory, and so on. This is also in line with 
the ``Linux`` philosophy: each tool only does one thing and does it well. But the reality is 
often very complicated, and only one thing can't solve the problems encountered in reality, 
let alone product development and large commercial projects. For example, subscription-broadcasting 
is a basic communication requirement, but no basic component can satisfy it.


Actually ``Linux`` has a powerful ``IPC``: ``D-Bus``. It has sophisticated method invocation mechanisms 
and event broadcast mechanisms; it also includes advanced features such as security policies and 
on-demand startup of services. But the biggest controversy about it is performance: its performance is 
very low, due to the daemon relay, a request-reply needs to replicate ten times, four message verification, 
and four context switches. Therefore, it can only be used to handle control commands and message delivery 
with lower real-time requirements and smaller data volume, otherwise it will have to resort to the basic 
``IPC`` framework. For this reason, someone wrote ``D-Bus`` into the kernel and generated ``KDBus``. 
Although the performance is improved, the disadvantages are obvious. It can only be run on a single machine 
and does not support cross-host. In this case, Android's Binder is also sufficient, and Binder has been 
accepted by the kernel. ``KDBus`` has not yet `"turned positive"`. In addition, whether it is DBus or 
``KDBus``, the provision of the basic API, there is still a big gap from the "middleware development framework." 
However, there is an increasing demand for various industries, including the automotive industry, so that 
various DBus packages are produced: Qt DBus, gDBus, commonAPI, ``DBus-C++``... But these packages are either 
subordinate to the big frame. Or lack of maintenance, in short, it is not friendly to use.


In the automotive field where ``Linux`` and ``Ethernet`` are used more and more widely, the lack of suitable 
``IPC`` has gradually become a prominent problem: the company's original ``IPC`` mechanism is backward due to 
backward technology and obvious customization, and it has been unable to meet the requirements of distributed, 
high performance and security. However, it is unable to find a suitable ``IPC`` mechanism for the new platform, 
let alone a middleware development framework derived from the IPC mechanism. ``Ethernet`` in-vehicle applications 
have spawned ``SOME/IP`` (Scalable service-Oriented MiddlewarE over IP). SOME/IP is also a relatively complete 
``IPC`` specification, even developed specifically for the automotive industry. But as the name implies, it is 
based on the IP layer and does not perform well on a single machine. And ``SOME / IP`` open source implementation 
is also very few, GENIVI organization contributed vsomeip, but the activity is very low, ``GENIVI`` itself is a 
loose organization, more participants, fewer developers. Unlike ``DBus``, ``SOME/IP`` is built for the car and has 
a narrow range of applications. It is impossible to expect an active community to gather a group of professional 
programmers to maintain open source (this is probably why ``GENIVI`` is not a climate). Finally, it is very likely 
that you have to pay for closed source software.


``FDBus`` was developed to solve the above problems and has the following characteristics:

- Distributed: Based on TCP sockets and Unix Domain sockets (UDS), it can be used for both local 
  ``IPC`` and ``IPC`` between network hosts.
- Cross-platform: Currently verified on ``Windows``, ``Linux`` and ``QNX``
- High performance: point-to-point direct communication, not forwarded through a central hub or broker
- Security: Ability to configure different levels of access for server method calls and event broadcasts. 
  Only clients with high enough permissions can characterize methods and receive specific events.
- Service name resolution: The server address is identified by name, the service is registered by 
  the name server, and the name is resolved, so that the server can be deployed anywhere on the network.
- Support cross-platform middleware development framework, including the following components:
 * Thread model
 * Event Loop
 * Job-to-thread communication based on Job-Worker
 * Event Loop based Timer
 * Event Loop based watch
 * Mutex
 * Semaphore
 * Socket
 * Notification
- IPC adopts Client-Server mode and supports the following communication modes:
 * Sync request with timeout - reply
 * Asynchronous request with timeout - reply
 * Unanswered command request
 * Registration-release mode for multicast
- IPC message serialization and deserialization using Protocol buffer, support IDL code generation, 
  efficient and simple; also supports raw data format, convenient for large data transmission
- Reliable heartbeat and reconnection mechanisms ensure that all parties remain connected regardless 
  of network conditions, regardless of which service is back online or restarted
- C++ implementation, easy to develop and maintain

Mainstream IPC framework comparison
-----------------------------------

+--------+-------------------+---------+--------------+-----------+--------------------+----------+----------+------------+
| Bottom |    performance    |  Sync   | Asynchronous |  Request  |     Cross-host     | Message  |  Cross-  |  security  |
| layer  |                   | request |   request    | timed out |                    |  push    | platform |  strategy  |
+========+===================+=========+==============+===========+====================+=====================+============+
| Socket | Point-to-point,   |   YES   |      YES     |    YES    | YES                | YES      | Window   |  YES       |
|        | high performance, |         |              |           | with timeout       | with     | Linux    | Developing |
|        | second only to    |         |              |           | and heartbeat      | simple   | QNX      |            |
|        | Binder            |         |              |           | to ensure reliable | string   |          |            |
|        |                   |         |              |           | connection         | matching |          |            |
+--------+-------------------+---------+--------------+-----------+--------------------+----------+----------+------------+


