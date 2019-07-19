############
Introduction
############

``FDBus`` is a middleware development framework targeting the following objectives:

* Inter-Process Communication (IPC) within single host and cross the network
* System abstraction (Windows, Linux, QNX)
* Components based on which middleware is built (job, worker, timer, watch...)

It is something like ``DBus`` or ``SOME/IP``, but with its own characteristic:

* :command:`Distributed`: unlike DBus, it has no central hub
* :command:`High performance`: endpoints talk to each other directly
* :command:`Addressing by name`: service is addressable through logic name
* :command:`Address allocation`: service address is allocated dynamically
* :command:`Networking`: communication inside host and cross hosts
* :command:`IDL and code generation`: using protocol buffer
* :command:`Total slution`: it is more than an IPC machanism. it is a middleware development framework
